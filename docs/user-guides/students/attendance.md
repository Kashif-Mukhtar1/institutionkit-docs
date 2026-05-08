File name: `docs/user-guides/students/attendance.md`

```markdown
# Student Attendance

The attendance module tracks daily student presence with four status options: Present, Absent, Late, and On Leave. Attendance can be marked from both the admin panel and the frontend teacher portal.

---

## Accessing Attendance

| Interface | Path | Who Uses It |
|-----------|------|-------------|
| **Admin Panel** | InstitutionKit → Student Management → Student Attendance | Admins, Campus Admins |
| **Frontend** | Page with `[institutionkit_attendance]` shortcode | Teachers |

---

## Admin Attendance Page

### Filter Form

Before marking attendance, select:

| Filter | Description |
|--------|-------------|
| **Class** | Required — select the class to take attendance for |
| **Section** | Optional — filter to a specific section |
| **Date** | Defaults to today. Can be changed for backdating. |

Click **Show Student List** to load students.

### Student List

Each student row displays:

| Column | Description |
|--------|-------------|
| Student Name | Full name from post title |
| Roll Number | `_ik_roll_number` |
| Status | Radio buttons: Present / Absent / Late / On Leave |
| Parent Contacts | Father's and mother's phone numbers |

### Status Options

| Status | Color | Remarks Field |
|--------|-------|---------------|
| **Present** | Green | Not shown |
| **Absent** | Red | Not shown |
| **Late** | Yellow | **Shown** — "Reason for late..." |
| **On Leave** | Blue | **Shown** — "Reason for leave..." |

The remarks field appears dynamically when "Late" or "On Leave" is selected.

### Save Behavior

- **First save**: Records are inserted into `institutionkit_attendance`
- **Subsequent saves**: Records are updated (upsert based on `student_id + attendance_date + class_id`)
- **After save**: The page becomes **read-only** — statuses cannot be changed from the frontend. Administrators can still modify from the admin panel.

---

## Frontend Teacher Attendance

### Shortcode

```
[institutionkit_attendance]
```

### Teacher Campus Restriction

Teachers can only mark attendance for students in their assigned campus:

```php
// Get teacher's campus from staff table
$staff_id = $wpdb->get_var($wpdb->prepare(
    "SELECT staff_id FROM {$wpdb->prefix}institutionkit_staff WHERE user_id = %d",
    $user->ID
));
$campus_id = $wpdb->get_var($wpdb->prepare(
    "SELECT primary_campus_id FROM {$wpdb->prefix}institutionkit_staff WHERE staff_id = %d",
    $staff_id
));
```

### Section Requirement

If **any sections exist** in the system, the teacher **must** select a section before students appear. This ensures teachers only see their assigned students.

### Read-Only Mode

When attendance has already been saved for a date:

```
⚠️ Attendance for this date has been saved and is now read-only from the front end. 
It can be modified by an administrator from the back end.
```

The page displays who submitted the attendance and at what time:

```
Attendance submitted by Sarah Ahmed at 08:45 AM
```

---

## Attendance Data Structure

### Database Table

```sql
CREATE TABLE wp_institutionkit_attendance (
    attendance_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    student_id BIGINT(20) UNSIGNED NOT NULL,
    class_id BIGINT(20) UNSIGNED NOT NULL,
    attendance_date DATE NOT NULL,
    status VARCHAR(20) DEFAULT 'present',
    remarks TEXT,
    marked_by BIGINT(20) UNSIGNED NOT NULL,
    campus_id INT DEFAULT 1,
    PRIMARY KEY (attendance_id),
    KEY campus_id (campus_id)
);
```

### Status Values

| Value | Meaning |
|-------|---------|
| `present` | Student was present |
| `absent` | Student was absent |
| `late` | Student arrived late (remarks recommended) |
| `leave` | Student is on approved leave (remarks recommended) |

---

## Attendance Reports

### Accessing Reports

Navigate to **Student Management → Attendance Report**.

### Report Features

| Feature | Description |
|---------|-------------|
| Date Range | Filter by start and end date |
| Class Filter | Filter by specific class |
| Student Filter | Filter by individual student |
| Status Summary | Count of present/absent/late/leave |
| Percentage | Attendance rate calculation |
| Export | Download as CSV |

### Monthly Attendance Statistics

```php
// Get attendance stats for a month
global $wpdb;
$month = '2025-06';
$stats = $wpdb->get_results($wpdb->prepare(
    "SELECT 
        status,
        COUNT(*) as count
     FROM {$wpdb->prefix}institutionkit_attendance 
     WHERE student_id = %d 
       AND attendance_date BETWEEN %s AND %s
     GROUP BY status",
    $student_id,
    $month . '-01',
    $month . '-31'
));
```

---

## Dashboard Attendance Widget

The main dashboard displays today's attendance summary:

| Stat | Source |
|------|--------|
| Students Present | COUNT WHERE status = 'present' AND attendance_date = today |
| Students Absent | COUNT WHERE status = 'absent' AND attendance_date = today |
| Students Late | COUNT WHERE status = 'late' AND attendance_date = today |
| Students On Leave | COUNT WHERE status = 'leave' AND attendance_date = today |
| Attendance Rate | (Present + Late) ÷ Total Marked × 100 |

Clicking on any status count opens a detailed popup showing the specific students.

---

## Bulk Actions

### Mark All Present

Click **Mark All Present** to set every student in the list to "Present" status. This is useful when most students are present and only a few need status changes.

### Mark All Absent

Click **Mark All Absent** to set every student to "Absent" — useful for holidays or school closures.

### Quick Status Toggle

Click any status radio button to change that student's status. The remarks field automatically shows/hides based on the selected status.

---

## Attendance Calculation

### Attendance Percentage

```
Attendance % = (Present Days + Late Days) ÷ Total School Days × 100
```

Leave days are excluded from the total school days calculation.

### Example

```
Present: 18 days
Absent: 2 days
Late: 3 days
Leave: 2 days
Total School Days: 23 (Present + Absent + Late) = 23
Attendance: (18 + 3) ÷ 23 × 100 = 91.3%
```

---

## Programmatic Attendance

### Record Attendance via Code

```php
global $wpdb;
$table = $wpdb->prefix . 'institutionkit_attendance';
$current_time = current_time('mysql');

// Check if record exists
$existing = $wpdb->get_var($wpdb->prepare(
    "SELECT attendance_id FROM {$table} 
     WHERE student_id = %d AND attendance_date = %s AND class_id = %d",
    $student_id, $date, $class_id
));

if ($existing) {
    // Update
    $wpdb->update($table, [
        'status'    => $status,
        'remarks'   => $remarks,
        'marked_by' => get_current_user_id(),
        'marked_at' => $current_time,
    ], ['attendance_id' => $existing]);
} else {
    // Insert
    $wpdb->insert($table, [
        'student_id'       => $student_id,
        'class_id'         => $class_id,
        'attendance_date'  => $date,
        'status'           => $status,
        'remarks'          => $remarks,
        'marked_by'        => get_current_user_id(),
        'marked_at'        => $current_time,
    ]);
}
```

### Get Student Attendance Count

```php
global $wpdb;
$total = $wpdb->get_var($wpdb->prepare(
    "SELECT COUNT(*) FROM {$wpdb->prefix}institutionkit_attendance 
     WHERE student_id = %d",
    $student_id
));

$present = $wpdb->get_var($wpdb->prepare(
    "SELECT COUNT(*) FROM {$wpdb->prefix}institutionkit_attendance 
     WHERE student_id = %d AND status IN ('present', 'late')",
    $student_id
));

$percentage = $total > 0 ? round(($present / $total) * 100) : 0;
```
