```markdown
# Teacher Comments

The Teacher Comments module enables teachers to leave feedback about students, and parents to respond — creating a two-way communication channel within the system.

---

## Accessing Comments

| Interface | Path | Who Uses It |
|-----------|------|-------------|
| **Admin Panel** | InstitutionKit → Teacher Management → Comments | Teachers, Admins |
| **Parent Portal** | Parent Portal → Teacher Comments | Parents |

---

## Teacher Comments (Admin View)

### Writing a Comment

Teachers can leave comments about any student:

| Field | Description |
|-------|-------------|
| **Student** | Select the student |
| **Class** | Auto-filled from student's class |
| **Comment Type** | General, Academic, Behavioral, Attendance |
| **Comment** | Full comment text |

### Comment Types

| Type | Use For |
|------|---------|
| **General** | General observations |
| **Academic** | Academic performance feedback |
| **Behavioral** | Classroom behavior and conduct |
| **Attendance** | Attendance-related concerns |

### Comment List View

| Column | Description |
|--------|-------------|
| Student | Student name |
| Teacher | Staff member who wrote the comment |
| Class | Student's class |
| Type | Comment category |
| Comment | Comment preview |
| Date | Date written |
| Response | Parent response status |
| Status | Read / Unread |

---

## Parent Response (Parent Portal)

### Viewing Comments

Parents see teacher comments in the Parent Portal under the **Teacher Comments** section.

### Responding to Comments

If a parent hasn't responded, a response form appears:

```
┌──────────────────────────────────────┐
│ 👩‍🏫 Sarah Ahmed                      │
│ June 15, 2026                        │
│                                      │
│ Your child has been doing well in    │
│ mathematics but needs to improve     │
│ in homework completion.              │
│                                      │
│ ┌──────────────────────────────────┐ │
│ │ Write your response...           │ │
│ └──────────────────────────────────┘ │
│ [Send Response]                      │
└──────────────────────────────────────┘
```

After responding, the parent's response appears below the comment:

```
┌──────────────────────────────────────┐
│ 👩‍🏫 Sarah Ahmed                      │
│ June 15, 2026                        │
│                                      │
│ Your child has been doing well...    │
│                                      │
│ ┌─ Your Response ──────────────────┐ │
│ │ Thank you for the feedback.      │ │
│ │ We will work on homework.        │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

---

## Comment Database

```sql
CREATE TABLE wp_institutionkit_teacher_comments (
    comment_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    student_id BIGINT(20) UNSIGNED NOT NULL,
    teacher_id BIGINT(20) UNSIGNED NOT NULL,
    class_id BIGINT(20) UNSIGNED NOT NULL,
    comment_type VARCHAR(50) DEFAULT 'general',
    comment TEXT NOT NULL,
    parent_response TEXT,
    response_date DATETIME,
    is_read TINYINT(1) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    campus_id INT DEFAULT 1,
    KEY student_id (student_id),
    KEY teacher_id (teacher_id),
    KEY created_at (created_at),
    KEY campus_id (campus_id)
);
```

### Teacher ID Reference

The `teacher_id` column now references `staff_id` from the `institutionkit_staff` table (not WordPress post IDs). This changed in version 1.2.0 when the Teacher CPT was migrated to the Staff table.

---

## Read Status Tracking

Comments track whether parents have read them:

| Status | Icon | Meaning |
|--------|------|---------|
| **Unread** | 🔵 | Parent hasn't viewed the comment |
| **Read** | ✅ | Parent has viewed the comment |

---

## AJAX Endpoint

### Parent Response

```php
// Action: ik_parent_respond_comment
// Nonce: ik_parent_comment_nonce

$_POST = [
    'comment_id' => 42,
    'response'   => 'Thank you for the feedback.',
    'nonce'      => '...',
];
```

Response:
```json
{
    "success": true,
    "message": "Response saved."
}
```

---

## Programmatic Usage

### Add a Comment

```php
global $wpdb;
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_teacher_comments",
    [
        'student_id'   => $student_id,
        'teacher_id'   => $staff_id,     // References institutionkit_staff
        'class_id'     => $class_id,
        'comment_type' => 'academic',
        'comment'      => 'Good progress in mathematics.',
        'created_at'   => current_time('mysql'),
        'campus_id'    => $campus_id,
    ]
);
```

### Get Comments for a Student

```php
global $wpdb;
$comments = $wpdb->get_results($wpdb->prepare(
    "SELECT 
        tc.*,
        s.full_name as teacher_name
     FROM {$wpdb->prefix}institutionkit_teacher_comments tc
     LEFT JOIN {$wpdb->prefix}institutionkit_staff s ON tc.teacher_id = s.staff_id
     WHERE tc.student_id = %d
     ORDER BY tc.created_at DESC
     LIMIT 10",
    $student_id
));
```

### Save Parent Response

```php
global $wpdb;
$wpdb->update(
    "{$wpdb->prefix}institutionkit_teacher_comments",
    [
        'parent_response' => sanitize_textarea_field($_POST['response']),
        'response_date'   => current_time('mysql'),
    ],
    ['comment_id' => $comment_id]
);
```

### Mark Comment as Read

```php
global $wpdb;
$wpdb->update(
    "{$wpdb->prefix}institutionkit_teacher_comments",
    ['is_read' => 1],
    ['comment_id' => $comment_id]
);
```

---

## Dashboard Integration

Teacher comments appear in two dashboard locations:

1. **Parent Portal** — Under "Teacher Comments" section
2. **Student Profile** — Comment history visible when viewing a student

---

## Best Practices

1. **Use comment types** — Categorize comments for easier filtering
2. **Keep comments constructive** — Focus on actionable feedback
3. **Respond promptly** — Parent responses acknowledge receipt
4. **Review unread comments** — The read status helps track parent engagement
5. **Use for documentation** — Comments create a dated record of communication
```
