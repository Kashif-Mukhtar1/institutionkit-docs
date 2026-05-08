File name: `docs/user-guides/students/overview.md`

```markdown
# Student Management Overview

The Student Management module handles the complete student lifecycle — from admission to graduation. All student-related tools are accessible from a central dashboard with card-based navigation.

---

## Accessing Student Management

Navigate to **InstitutionKit → Student Management** in the admin menu.

---

## Dashboard Cards

The Student Management landing page presents 12 action cards:

| Card | Destination | Description |
|------|-------------|-------------|
| 👥 **All Students** | Student list page | View, search, and manage all enrolled students |
| ➕ **Add New Student** | WordPress post editor | Enroll a new student directly |
| 📝 **New Admission** | Guided admission form | Process new admissions with full data collection |
| 📚 **Homework** | Homework management | View and manage homework assignments |
| 📋 **Student Attendance** | Attendance marking | Mark and track daily student attendance |
| 📊 **Gradebook** | Gradebook entry | Enter and manage student grades |
| 🔄 **Campus Transfer** | Transfer interface | Transfer students between campuses |
| 📤 **Student Promotion** | Promotion tool | Promote students to the next class |
| ✏️ **Assign Homework** | Homework creation | Create and assign new homework |
| 📅 **All Grading Periods** | Period list | View and manage grading periods |
| 🎓 **Certificates** | Certificate generation | Generate Leaving, Character, and Achievement certificates |
| 📊 **Attendance Report** | Attendance analytics | View attendance reports and trends |

---

## Student Post Type

Students are stored as WordPress posts of type `ik_student` with extensive post meta.

### Core Fields

| Field | Meta Key | Type |
|-------|----------|------|
| Full Name | `post_title` | WordPress title |
| Photo | `_thumbnail_id` | WordPress featured image |
| Class | `_ik_student_class_id` | Post ID reference |
| Campus | `_ik_campus_id` | Integer |
| Roll Number | `_ik_roll_number` | Text |
| Date of Birth | `_ik_date_of_birth` | Date |
| Gender | `_ik_gender` | Text |
| Email | `_ik_email` | Email |
| Address | `_ik_address` | Textarea |
| Section | `_ik_section` | Text |

### Guardian Information

| Field | Meta Key |
|-------|----------|
| Guardian Name | `_ik_guardian_name` |
| Guardian Title | `_ik_guardian_title` |
| Father's Name | `_ik_father_name` |
| Father's Contact | `_ik_father_contact` |
| Mother's Contact | `_ik_mother_contact` |
| Father's Occupation | `_ik_father_occupation` |
| Mother's Occupation | `_ik_mother_occupation` |
| Father's Qualification | `_ik_father_qualification` |
| Mother's Qualification | `_ik_mother_qualification` |
| Emergency Contact | `_ik_emergency_contact` |
| ID Card Number | `_ik_cnic_number` |

---

## Student List View

The **All Students** page provides:

### Filters

| Filter | Description |
|--------|-------------|
| Class | Filter by enrolled class |
| Campus | Filter by campus (Super Admins only) |
| Section | Filter by section |
| Search | Search by name or roll number |

### Table Columns

| Column | Source |
|--------|--------|
| Photo | Featured image thumbnail |
| Name | Post title |
| Roll Number | `_ik_roll_number` |
| Class | Linked class name |
| Section | `_ik_section` |
| Guardian | `_ik_guardian_name` |
| Contact | `_ik_emergency_contact` or `_ik_father_contact` |
| Campus | Campus name from `_ik_campus_id` |

### Bulk Actions

| Action | Description |
|--------|-------------|
| Delete | Permanently remove selected students |
| Change Class | Move selected students to a different class |
| Export | Export student data as CSV |

---

## Adding a Student

Students can be added through two methods:

### Method 1: Quick Add (WordPress Editor)

Navigate to **Add New Student**. This opens the standard WordPress post editor with InstitutionKit meta boxes.

### Method 2: New Admission Form

Navigate to **New Admission**. This provides a guided form with:

1. **Student Information**: Name, DOB, gender, class selection
2. **Guardian Information**: Guardian details, both parents' information
3. **Contact Information**: Email, phone, address
4. **Document Upload**: Optional supporting documents

---

## Student Meta Boxes

The student edit screen includes custom meta boxes:

### Student Details Meta Box

| Field | Type |
|-------|------|
| Class | Dropdown of `ik_class` posts |
| Roll Number | Text input |
| Date of Birth | Date picker |
| Gender | Select (Male/Female/Other) |
| Email | Email input |
| Campus | Dropdown (Super Admins only) |

### Guardian Information Meta Box

| Field | Type |
|-------|------|
| Guardian Name | Text input |
| Relationship | Select (Father/Mother/Guardian) |
| Father's Name | Text input |
| Father's Contact | Phone input |
| Mother's Contact | Phone input |
| Father's Occupation | Text input |
| Mother's Occupation | Text input |
| ID Card Number | Text input (validated as 13 digits) |
| Address | Textarea |

---

## Campus Filtering

When a Campus Admin views students, the list is automatically filtered to their assigned campus. The campus column and filter are hidden.

Super Admins can:
- Filter by campus using the dropdown
- See the campus column in the student list
- Transfer students between campuses

---

## Student Data Relationships

```
ik_student (Post)
    │
    ├── Post Meta (_ik_*)
    │
    ├── institutionkit_attendance (N records)
    │   └── Daily attendance: present/absent/late/leave
    │
    ├── institutionkit_invoices (N records)
    │   └── Fee invoices with payment status
    │
    ├── institutionkit_gradebook / ik_grades_v2 (N records)
    │   └── Grades per subject per grading period
    │
    ├── ik_exam_results (N records)
    │   └── Exam performance per exam schedule
    │
    ├── ik_report_cards (N records)
    │   └── Generated report cards
    │
    ├── institutionkit_parent_child (N records)
    │   └── Links to parent WordPress users
    │
    ├── institutionkit_teacher_comments (N records)
    │   └── Teacher comments with parent responses
    │
    ├── ik_student_promotions (N records)
    │   └── Promotion history
    │
    ├── institutionkit_homework_submissions (N records)
    │   └── Homework submission records
    │
    └── institutionkit_meeting_bookings (N records)
        └── Parent-teacher meeting bookings
```

---

## Student Lifecycle

```
Admission → Active Enrollment → Class Promotion → Graduation/Leaving
                │
                ├── Campus Transfer
                ├── Certificate Issuance
                └── Fee Management (ongoing)
```

---

## Quick Reference

### Get Student by ID

```php
$student = get_post($student_id);
$name = $student->post_title;
$class_id = get_post_meta($student_id, '_ik_student_class_id', true);
$roll = get_post_meta($student_id, '_ik_roll_number', true);
```

### Get Students by Class

```php
$students = get_posts([
    'post_type'      => 'ik_student',
    'posts_per_page' => -1,
    'meta_key'       => '_ik_student_class_id',
    'meta_value'     => $class_id,
    'orderby'        => 'title',
    'order'          => 'ASC',
]);
```

### Get Student Attendance

```php
global $wpdb;
$attendance = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_attendance 
     WHERE student_id = %d 
     ORDER BY attendance_date DESC 
     LIMIT 30",
    $student_id
));
```

### Get Student Fees

```php
global $wpdb;
$invoices = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_invoices 
     WHERE student_id = %d 
     ORDER BY created_at DESC",
    $student_id
));
```
