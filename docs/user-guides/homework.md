```markdown
# Homework Management

The Homework module enables teachers to create and assign homework, track submissions, and provide grades and feedback. Parents can view assigned homework through the Parent Portal.

---

## Accessing Homework

| Interface | Path | Who Uses It |
|-----------|------|-------------|
| **Admin Panel** | InstitutionKit → Student Management → Homework | Admins, Teachers |
| **Parent Portal** | Parent Portal → Homework Assignments | Parents |

---

## Homework Dashboard Cards

| Card | Destination | Description |
|------|-------------|-------------|
| 📚 **Homework** | Homework list | View and manage assignments |
| ✏️ **Assign Homework** | Homework creation | Create and assign new homework |

---

## Homework List View

| Column | Description |
|--------|-------------|
| Subject | Subject name with icon |
| Title | Homework title |
| Class | Target class |
| Section | Target section (if assigned) |
| Due Date | Submission deadline |
| Status | Active / Archived |
| Recurring | Recurring badge if applicable |
| Actions | Edit / Delete |

---

## Creating Homework

Click **Assign Homework** to open the creation form.

### Homework Form

| Field | Required | Description |
|-------|----------|-------------|
| Class | Yes | Target class |
| Section | No | Optional section filter |
| Subject | Yes | Subject name |
| Title | Yes | Homework title |
| Description | No | Full details and instructions |
| Attachment | No | File upload (image, PDF, document) |
| Due Date | Yes | Submission deadline |
| Recurring | No | Enable recurring homework |
| Recurring Pattern | If recurring | Daily / Weekly / Monthly |
| Recurring Value | If recurring | Interval (e.g., every 2 weeks) |

---

## Recurring Homework

### How Recurring Works

When a homework is marked as recurring:

1. The original homework is created with the recurring settings
2. A badge indicates the recurrence pattern
3. The homework appears repeatedly based on the schedule

### Recurring Badge

```
📖 Mathematics
   🔄 Recurring (Every 1 week)
```

### Recurring Patterns

| Pattern | Example | Value |
|---------|---------|-------|
| Daily | Every 1 day | 1 |
| Weekly | Every 2 weeks | 2 |
| Monthly | Every 1 month | 1 |

---

## Homework Status

| Status | Description |
|--------|-------------|
| **Active** | Currently assigned, accepting submissions |
| **Archived** | Past due date or manually archived |

Homework past its due date should be manually archived or will remain active until changed.

---

## Homework Database

```sql
CREATE TABLE wp_institutionkit_homework (
    id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    class_id BIGINT(20) UNSIGNED NOT NULL,
    section VARCHAR(50),
    subject VARCHAR(255) NOT NULL,
    title VARCHAR(255) NOT NULL,
    description LONGTEXT,
    attachment VARCHAR(500),
    due_date DATE NOT NULL,
    is_recurring TINYINT(1) DEFAULT 0,
    recurring_pattern VARCHAR(50),
    recurring_value INT(11),
    assigned_by BIGINT(20) UNSIGNED NOT NULL,
    assigned_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'active',
    campus_id INT DEFAULT 1,
    KEY class_id (class_id),
    KEY assigned_by (assigned_by),
    KEY due_date (due_date),
    KEY status (status),
    KEY campus_id (campus_id)
);
```

---

## Homework Submissions

### Submission Table

```sql
CREATE TABLE wp_institutionkit_homework_submissions (
    submission_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    homework_id BIGINT(20) UNSIGNED NOT NULL,
    student_id BIGINT(20) UNSIGNED NOT NULL,
    submission_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    submission_text TEXT,
    attachment VARCHAR(500),
    status VARCHAR(20) DEFAULT 'submitted',
    grade VARCHAR(10),
    teacher_feedback TEXT,
    graded_by BIGINT(20) UNSIGNED,
    graded_date DATETIME,
    campus_id INT DEFAULT 1,
    KEY homework_id (homework_id),
    KEY student_id (student_id),
    KEY status (status),
    KEY campus_id (campus_id)
);
```

### Submission Status

| Status | Description |
|--------|-------------|
| Submitted | Student has submitted |
| Graded | Teacher has graded |
| Returned | Feedback returned to student |

---

## Parent Portal View

Parents see homework in the Parent Portal:

```
📚 Homework Assignments

┌──────────────────────────────────────────┐
│ 📖 Mathematics                          │
│ Worksheet Chapter 5                     │
│ 📅 Due: Friday, June 20, 2026           │
│                                          │
│ Complete all exercises from pages       │
│ 42-45. Show your working for each       │
│ problem.                                │
│                                          │
│ 📎 Download Attachment                   │
└──────────────────────────────────────────┘
```

### What Parents See

- Subject and title
- Due date
- Full description
- Attachment download link
- Recurring badge (if applicable)

---

## Capabilities

| Capability | Who Has It | Purpose |
|-----------|------------|---------|
| `ik_assign_homework` | Admin, Campus Admin, Teacher | Create homework |
| `ik_edit_homework` | Admin, Teacher | Edit homework |
| `ik_delete_homework` | Admin | Delete homework |
| `ik_view_all_homework` | Admin, Campus Admin, Teacher | View all homework |
| `ik_grade_submissions` | Admin, Campus Admin, Teacher | Grade submissions |
| `ik_view_homework` | Parent | View child's homework |
| `ik_view_own_homework` | Parent | View own children's homework |

---

## Programmatic Usage

### Create Homework

```php
global $wpdb;
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_homework",
    [
        'class_id'     => $class_id,
        'section'      => 'A',
        'subject'      => 'Mathematics',
        'title'        => 'Worksheet Chapter 5',
        'description'  => 'Complete all exercises from pages 42-45.',
        'due_date'     => '2026-06-20',
        'assigned_by'  => $staff_id,
        'campus_id'    => $campus_id,
    ]
);
```

### Get Homework for a Class

```php
global $wpdb;
$homework = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_homework 
     WHERE class_id = %d 
       AND status = 'active' 
       AND due_date >= CURDATE()
     ORDER BY due_date ASC",
    $class_id
));
```

### Get Homework by Student

```php
global $wpdb;
$student_class_id = get_post_meta($student_id, '_ik_student_class_id', true);

$homework = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_homework 
     WHERE class_id = %d 
       AND status = 'active' 
       AND due_date >= CURDATE()
     ORDER BY due_date ASC",
    $student_class_id
));
```

### Record a Submission

```php
global $wpdb;
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_homework_submissions",
    [
        'homework_id'     => $homework_id,
        'student_id'      => $student_id,
        'submission_text' => 'My completed assignment...',
        'status'          => 'submitted',
        'campus_id'       => $campus_id,
    ]
);
```

### Grade a Submission

```php
global $wpdb;
$wpdb->update(
    "{$wpdb->prefix}institutionkit_homework_submissions",
    [
        'status'           => 'graded',
        'grade'            => 'A',
        'teacher_feedback' => 'Excellent work! Clear explanations.',
        'graded_by'        => $staff_id,
        'graded_date'      => current_time('mysql'),
    ],
    ['submission_id' => $submission_id]
);
```
