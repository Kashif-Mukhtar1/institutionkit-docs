```markdown
# Teacher & Staff Management Overview

The Teacher Management module handles all staff-related operations — from hiring and contract management to attendance tracking and parent-teacher meetings.

---

## Accessing Teacher Management

Navigate to **InstitutionKit → Teacher Management** in the admin menu.

---

## Dashboard Cards

The Teacher Management landing page presents 7 action cards:

| Card | Destination | Description |
|------|-------------|-------------|
| 👥 **All Staff** | Staff list page | View and manage all staff members |
| ➕ **Add New Staff** | Staff creation form | Add a new staff member |
| 📋 **Staff Attendance** | Attendance marking | Mark and track daily staff attendance |
| 🔄 **Transfer Staff** | Campus transfer | Transfer staff between campuses |
| 📅 **Meeting Slots** | Meeting management | Manage parent-teacher meeting availability |
| 📚 **Homework** | Homework management | View and manage homework assignments |
| 📅 **All Grading Periods** | Period management | View and manage grading periods |

---

## Staff vs Teacher Terminology

InstitutionKit uses "Staff" as the umbrella term for all employees:

| Role | Category | Description |
|------|----------|-------------|
| `teacher_permanent` | Teaching | Full-time permanent teacher |
| `teacher_visiting` | Teaching | Part-time or visiting teacher |
| `campus_head` | Administrative | Campus principal or head |
| `office_staff` | Administrative | Administrative and clerical staff |
| `admin_roving` | Administrative | Staff managing multiple campuses |
| `maintenance` | Support | Maintenance and facilities |
| `security` | Support | Security personnel |
| `other` | Support | Any other role |

Teaching staff (`teacher_permanent`, `teacher_visiting`) have access to gradebook, homework, and attendance marking. Non-teaching staff do not.

---

## Staff Data Structure

All staff are stored in the `institutionkit_staff` custom table — not as WordPress posts.

### Key Fields

| Field | Type | Description |
|-------|------|-------------|
| `staff_id` | BIGINT AUTO_INCREMENT | Primary key |
| `user_id` | BIGINT | Linked WordPress user (if account created) |
| `employee_code` | VARCHAR(50) UNIQUE | Auto-generated: `EMP{YY}{XXXX}` |
| `full_name` | VARCHAR(255) | Full legal name |
| `email` | VARCHAR(255) | Email address |
| `phone` | VARCHAR(50) | Contact number |
| `role` | VARCHAR(30) | One of 8 predefined roles |
| `primary_campus_id` | BIGINT | Primary campus assignment |
| `contract_type` | VARCHAR(20) | `monthly_fixed`, `hourly`, `per_lecture` |
| `base_salary` | DECIMAL(12,2) | For monthly fixed contracts |
| `hourly_rate` | DECIMAL(10,2) | For hourly contracts |
| `lecture_rate` | DECIMAL(10,2) | For per-lecture contracts |
| `employment_status` | VARCHAR(20) | `active`, `on_leave`, `terminated`, `retired` |
| `join_date` | DATE | Employment start date |

---

## Contract Types

InstitutionKit supports three contract types, each affecting payroll calculation differently:

### Monthly Fixed Salary

| Attribute | Value |
|-----------|-------|
| Payment Basis | Fixed monthly amount |
| Attendance Impact | Deductions for absences beyond 1 day |
| Payroll Formula | `base_salary - (absent_days - 1) × (base_salary ÷ 30)` |
| Perfect Attendance | +1 day bonus salary |

### Hourly Rate

| Attribute | Value |
|-----------|-------|
| Payment Basis | Hours worked × hourly rate |
| Attendance Impact | Direct — unpaid for hours not worked |
| Payroll Formula | `hours_worked × hourly_rate` |
| Standard Hours | Configurable per staff (default: 160/month) |

### Per Lecture Rate

| Attribute | Value |
|-----------|-------|
| Payment Basis | Lectures delivered × rate per lecture |
| Attendance Impact | Direct — unpaid for lectures not delivered |
| Payroll Formula | `lectures_count × lecture_rate` |
| Standard Lectures | Configurable per staff |

---

## WordPress User Integration

When a staff member is added with a role of `teacher_permanent`, `teacher_visiting`, `campus_head`, or `admin_roving`, InstitutionKit automatically:

1. **Creates a WordPress user account**
2. **Assigns the appropriate WordPress role:**
    - Teaching roles → `teacher`
    - Administrative roles → `campus_admin`
3. **Generates a random password**
4. **Sends a welcome email** with login credentials
5. **Links the WordPress user ID** to the staff record

Non-teaching roles (`office_staff`, `maintenance`, `security`, `other`) do not get WordPress accounts by default.

---

## Staff Lifecycle

```
Recruitment → Add Staff → Active Employment
                │
                ├── Attendance Tracking (daily)
                ├── Salary Management (monthly)
                ├── Loan Management (as needed)
                ├── Performance Monitoring
                │
                ├── On Leave (temporary)
                ├── Campus Transfer (permanent)
                ├── Termination (with reason and date)
                └── Retirement
```

---

## Quick Reference

### Get Staff by ID

```php
global $wpdb;
$staff = $wpdb->get_row($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_staff WHERE staff_id = %d",
    $staff_id
));
```

### Get All Active Teachers

```php
global $wpdb;
$teachers = $wpdb->get_results(
    "SELECT * FROM {$wpdb->prefix}institutionkit_staff 
     WHERE role IN ('teacher_permanent', 'teacher_visiting') 
     AND employment_status = 'active'
     ORDER BY full_name ASC"
);
```

### Get Staff by Campus

```php
global $wpdb;
$campus_staff = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_staff 
     WHERE primary_campus_id = %d 
     AND employment_status = 'active'
     ORDER BY full_name ASC",
    $campus_id
));
```

### Get Staff Attendance

```php
global $wpdb;
$attendance = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_staff_attendance 
     WHERE staff_id = %d 
     AND attendance_date BETWEEN %s AND %s
     ORDER BY attendance_date ASC",
    $staff_id, $start_date, $end_date
));
```
```
