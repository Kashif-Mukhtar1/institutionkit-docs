File name: `docs/user-guides/students/promotion.md`

```markdown
# Student Promotion

The Student Promotion module handles end-of-year class advancement. Students can be promoted individually or in bulk, with full promotion history tracking.

---

## Accessing Promotions

Navigate to **InstitutionKit → Student Management → Student Promotion**.

---

## How Promotions Work

A promotion moves a student from one class to another and records the transition permanently:

```
Grade 5 (2025-2026) → Promotion → Grade 6 (2026-2027)
```

Each promotion record captures:

| Field | Description |
|-------|-------------|
| Student | The promoted student |
| From Class | Previous class |
| To Class | New class |
| Promotion Date | Date of promotion |
| Academic Year | e.g., "2025-2026" |
| Overall Percentage | Student's performance percentage |
| Status | `promoted` or `retained` |
| Promoted By | User who performed the promotion |
| Remarks | Optional notes |

---

## Promotion Criteria

### Automatic Promotion Calculation

The system calculates eligibility based on:

1. **Overall academic percentage** from the current academic year
2. **Attendance percentage** for the year
3. **Minimum promotion threshold** (configurable in Settings)

### Manual Override

Administrators can override automatic decisions and manually promote or retain any student.

---

## Promotion Page Interface

### Filter Section

| Filter | Description |
|--------|-------------|
| **From Class** | Select the current class to promote from |
| **To Class** | Select the destination class |
| **Academic Year** | The new academic year (auto-generated) |

### Student List

After applying filters, the page shows all students in the source class with:

| Column | Description |
|--------|-------------|
| Student Name | Full name |
| Roll Number | Current roll number |
| Current Class | Source class |
| Overall % | Calculated performance percentage |
| Attendance % | Year attendance rate |
| Status | `Eligible`, `Borderline`, `Not Eligible` |
| Action | Promote / Retain buttons |

---

## Bulk Promotion

### Promote All Eligible

Click **Promote All Eligible** to promote every student who meets the promotion criteria. The system:

1. Validates each student's eligibility
2. Creates a promotion record for each student
3. Updates the student's class meta (`_ik_student_class_id`)
4. Logs the promotion in `ik_student_promotions`

### Promote Selected

1. Check the students you want to promote
2. Select the destination class
3. Click **Promote Selected**

---

## Individual Promotion

Each student row has individual action buttons:

| Button | Action |
|--------|--------|
| **Promote** | Promote to the selected destination class |
| **Retain** | Mark as retained (repeat the same class) |
| **View History** | Show this student's complete promotion history |

---

## Promotion History

Every promotion is permanently recorded in the `ik_student_promotions` table:

```sql
CREATE TABLE wp_ik_student_promotions (
    promotion_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    student_id BIGINT UNSIGNED NOT NULL,
    from_class_id BIGINT UNSIGNED NOT NULL,
    to_class_id BIGINT UNSIGNED NOT NULL,
    promotion_date DATE NOT NULL,
    academic_year VARCHAR(9) NOT NULL,
    overall_percentage DECIMAL(5,2),
    status VARCHAR(20) DEFAULT 'promoted',
    promoted_by BIGINT UNSIGNED NOT NULL,
    remarks TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX (student_id),
    INDEX (from_class_id),
    INDEX (to_class_id),
    INDEX (academic_year)
);
```

### Viewing History

Click **View History** on any student row to see:

| Column | Description |
|--------|-------------|
| Academic Year | e.g., "2024-2025" |
| From Class | Previous class name |
| To Class | New class name |
| Percentage | Performance at time of promotion |
| Status | Promoted or Retained |
| Date | Promotion date |
| Promoted By | Staff member name |

---

## Capabilities

| Capability | Who Has It | Purpose |
|-----------|------------|---------|
| `ik_manage_student_promotions` | Administrator, Campus Admin | Access promotion page and perform promotions |
| `ik_view_promotion_history` | Administrator, Campus Admin, Teacher | View promotion records |
| `ik_bulk_promote_students` | Administrator, Campus Admin | Perform bulk promotions |

---

## After Promotion

Once students are promoted:

1. **Student meta is updated** — `_ik_student_class_id` changes to the new class
2. **Promotion record is created** — permanent history
3. **Historical data preserved** — previous attendance, grades, and invoices remain linked to the student
4. **New academic year begins** — the student appears in the new class for attendance and gradebook

### Roll Number Handling

Roll numbers may need reassignment after promotion. This is a manual process:

1. Go to **Student Management → All Students**
2. Filter by the new class
3. Edit each student to assign new roll numbers

---

## Retention

When a student does not meet promotion criteria:

| Action | Effect |
|--------|--------|
| **Retain** | Student repeats the same class |
| Promotion record created with status `retained` |
| Class meta unchanged |
| Historical grades preserved |

---

## Programmatic Promotion

### Promote a Student via Code

```php
global $wpdb;

$student_id = 123;
$from_class_id = 45;
$to_class_id = 46;
$academic_year = '2026-2027';

// Update student's class
update_post_meta($student_id, '_ik_student_class_id', $to_class_id);

// Record promotion
$wpdb->insert(
    "{$wpdb->prefix}ik_student_promotions",
    [
        'student_id'         => $student_id,
        'from_class_id'      => $from_class_id,
        'to_class_id'        => $to_class_id,
        'promotion_date'     => current_time('Y-m-d'),
        'academic_year'      => $academic_year,
        'overall_percentage' => 78.50,
        'status'             => 'promoted',
        'promoted_by'        => get_current_user_id(),
    ]
);
```

### Get Student Promotion History

```php
global $wpdb;
$history = $wpdb->get_results($wpdb->prepare(
    "SELECT 
        sp.*,
        fc.post_title as from_class_name,
        tc.post_title as to_class_name
     FROM {$wpdb->prefix}ik_student_promotions sp
     LEFT JOIN {$wpdb->posts} fc ON sp.from_class_id = fc.ID
     LEFT JOIN {$wpdb->posts} tc ON sp.to_class_id = tc.ID
     WHERE sp.student_id = %d
     ORDER BY sp.promotion_date DESC",
    $student_id
));
```

---

## Best Practices

1. **Run promotions at year-end** — After all final grades are entered and verified
2. **Review borderline cases manually** — The automatic calculation is a guideline, not a final decision
3. **Export promotion history** — Keep records for regulatory compliance
4. **Reassign roll numbers** — After bulk promotion, update roll numbers for the new class arrangement
5. **Verify class capacity** — Ensure destination classes have capacity before bulk promoting
```
