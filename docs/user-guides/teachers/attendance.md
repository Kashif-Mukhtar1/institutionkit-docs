```markdown
# Staff Attendance

The Staff Attendance module tracks daily attendance for all employees with time-in, time-out, and lecture tracking. It supports multiple attendance statuses and integrates directly with payroll calculations.

---

## Accessing Staff Attendance

Navigate to **InstitutionKit → Teacher Management → Staff Attendance** or **Payroll & Expenses → Staff Attendance**.

---

## Attendance Page Interface

### Filter Form

| Filter | Description |
|--------|-------------|
| **Campus** | Required — select the campus |
| **Date** | Defaults to today. Can be changed for backdating. |

Click **Load Staff** to display all active staff for the selected campus.

---

## Attendance Table Columns

| Column | Description |
|--------|-------------|
| **Staff Name** | Full name + employee code |
| **Role** | Staff role label |
| **Status** | Dropdown: Present / Absent / Half Day / Leave |
| **Check In** | Time input — auto-filled with school open time |
| **Check Out** | Time input — auto-filled with school close time |
| **Lectures** | Number input — lectures delivered (for teaching staff) |
| **Emergency Contact** | Shown only when status is Absent or Leave |
| **Remarks** | Optional text notes |

---

## Status Options

| Status | Check In/Out | Lectures | Emergency Contact |
|--------|:---:|:---:|:---:|
| **Present** | Enabled — auto-filled with defaults | Enabled | Hidden |
| **Absent** | Disabled — cleared | Disabled | **Shown** |
| **Half Day** | Enabled | Enabled | Hidden |
| **Leave** | Disabled — cleared | Disabled | **Shown** |

When switching to "Present" from another status, check-in and check-out are auto-filled from school operating hours.

---

## School Operating Hours

Default check-in and check-out times are configured in InstitutionKit Settings:

| Setting | Default |
|---------|---------|
| School Open Time | 09:00 AM |
| School Close Time | 05:00 PM |

These are converted to 24-hour format (`09:00`, `17:00`) for the time inputs.

### Changing Operating Hours

Navigate to **InstitutionKit → Settings** and update:

1. School Open Time
2. School Close Time

These apply campus-wide for attendance defaults.

---

## Emergency Contact Display

When a staff member is marked **Absent** or **Leave**, their emergency contact information is displayed:

```
⚠️ Emergency Contact
John Doe
📞 +92 300 1234567
```

If no emergency contact is recorded:

```
⚠️ No emergency contact
```

---

## Bulk Actions

Three bulk action buttons are available:

| Button | Effect |
|--------|--------|
| **Mark All Present** | Sets every staff member to Present, auto-fills check-in/check-out times |
| **Mark All Absent** | Sets every staff member to Absent, clears time fields |
| **Mark All Leave** | Sets every staff member to Leave, clears time fields |

---

## Saving Attendance

Click **Save Attendance** to record all changes. The system:

1. Validates campus and date
2. Processes each staff row
3. Checks for existing records (upsert logic)
4. Inserts or updates `institutionkit_staff_attendance`
5. Displays success message with count of saved records

---

## Attendance Database

```sql
CREATE TABLE wp_institutionkit_staff_attendance (
    attendance_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    staff_id BIGINT(20) UNSIGNED NOT NULL,
    campus_id BIGINT(20) UNSIGNED NOT NULL,
    attendance_date DATE NOT NULL,
    check_in TIME,
    check_out TIME,
    hours_worked DECIMAL(5,2),
    lectures_count INT DEFAULT 0,
    status VARCHAR(20) DEFAULT 'present',
    leave_type VARCHAR(50),
    is_approved TINYINT(1) DEFAULT 1,
    approved_by BIGINT(20) UNSIGNED,
    remarks TEXT,
    UNIQUE KEY unique_daily_attendance (staff_id, attendance_date)
);
```

**Unique constraint**: One record per staff member per day.

---

## How Attendance Affects Payroll

Attendance data directly impacts payroll calculation based on contract type:

### Monthly Fixed Salary

```
Per Day Salary = Base Salary ÷ Working Days in Month
Absent Deduction = Absent Days × Per Day Salary
Half Day Deduction = Half Days × (Per Day Salary ÷ 2)
```

- First absent day: No deduction
- Additional absent days: Full per-day deduction
- Perfect attendance (0 absences): +1 day bonus

### Hourly Rate

```
Hours Worked = SUM(hours_worked) from attendance
Pay = Hours Worked × Hourly Rate
```

### Per Lecture Rate

```
Total Lectures = SUM(lectures_count) from attendance
Pay = Total Lectures × Lecture Rate
```

---

## Working Days Calculation

Working days exclude Sundays:

```php
private function get_working_days_in_month($month) {
    $start = new DateTime($month . '-01');
    $end = new DateTime($month . '-01');
    $end->modify('last day of this month');
    
    $working_days = 0;
    $period = new DatePeriod($start, new DateInterval('P1D'), $end->modify('+1 day'));
    
    foreach ($period as $date) {
        if ($date->format('N') != 7) { // Not Sunday
            $working_days++;
        }
    }
    return $working_days;
}
```

Example: March 2026 has 31 days. Excluding 4 Sundays = 27 working days.

---

## Attendance Summary on Payslips

When payroll is generated, attendance data is included on payslips:

| Stat | Source |
|------|--------|
| Present Days | COUNT WHERE status = 'present' |
| Absent Days | COUNT WHERE status = 'absent' |
| Half Days | COUNT WHERE status = 'half_day' |
| Leave Days | COUNT WHERE status = 'leave' |
| Working Days | Calculated from month |
| Total Lectures | SUM of lectures_count |

---

## Programmatic Attendance

### Record Attendance via Code

```php
global $wpdb;
$table = $wpdb->prefix . 'institutionkit_staff_attendance';

$data = [
    'staff_id'        => $staff_id,
    'campus_id'       => $campus_id,
    'attendance_date' => '2026-06-15',
    'status'          => 'present',
    'check_in'        => '09:00',
    'check_out'       => '17:00',
    'lectures_count'  => 5,
    'remarks'         => '',
];

// Check if record exists
$existing = $wpdb->get_var($wpdb->prepare(
    "SELECT attendance_id FROM {$table} 
     WHERE staff_id = %d AND attendance_date = %s",
    $staff_id, $data['attendance_date']
));

if ($existing) {
    $wpdb->update($table, $data, ['attendance_id' => $existing]);
} else {
    $wpdb->insert($table, $data);
}
```

### Get Monthly Attendance Summary

```php
global $wpdb;
$summary = $wpdb->get_results($wpdb->prepare(
    "SELECT 
        status,
        COUNT(*) as count,
        SUM(hours_worked) as total_hours,
        SUM(lectures_count) as total_lectures
     FROM {$wpdb->prefix}institutionkit_staff_attendance 
     WHERE staff_id = %d 
       AND attendance_date BETWEEN %s AND %s
     GROUP BY status",
    $staff_id, $month_start, $month_end
));
```
