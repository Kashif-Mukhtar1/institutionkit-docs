```markdown
# Result Entry

The Result Entry module enables teachers to enter student marks, submit them for verification, and track the approval workflow from draft to publication.

---

## Accessing Result Entry

Navigate to **InstitutionKit → Exams → Result Entry**.

---

## Result Entry Page

### Filter Form

Before entering results, select:

| Filter | Description |
|--------|-------------|
| Exam Type | The exam to enter results for |
| Class | The class to load students from |
| Subject | The specific subject (from exam schedule) |

The system loads the corresponding exam schedule and displays the student list.

---

## Student Result Grid

For each student, the grid displays:

| Column | Description |
|--------|-------------|
| Student Name | Full name |
| Roll Number | Student roll number |
| Marks Obtained | Numeric input for marks |
| Max Marks | Pre-filled from schedule (editable) |
| Absent | Checkbox — mark student as absent |
| Grade | Auto-calculated letter grade |
| Status | Draft / Submitted |

---

## Entering Marks

### Individual Entry

1. Type the marks obtained for each student
2. Max marks is pre-filled from the schedule
3. Grade is auto-calculated as you type

### Absent Students

Check the **Absent** checkbox — marks field is cleared and disabled.

### Grade Auto-Calculation

As marks are entered, grades are calculated in real-time:

```javascript
function calculateGrade(marks, max) {
    var percentage = (marks / max) * 100;
    // Uses the configured grade scale
    if (percentage >= 98) return 'A++';
    if (percentage >= 95) return 'A+';
    if (percentage >= 90) return 'A';
    if (percentage >= 80) return 'B';
    if (percentage >= 65) return 'C';
    if (percentage >= 55) return 'D';
    if (percentage >= 40) return 'D';
    return 'F';
}
```

---

## Saving Results

### Save as Draft

Click **Save** to store results with `draft` status. Draft results:

- Visible only to the entering teacher
- Can be edited freely
- Not visible to verifiers

### Submit for Verification

Click **Submit for Verification** to change status to `submitted`:

- Results become read-only for the teacher
- Verifier is notified
- Results enter the verification workflow

---

## Verification Workflow

### Verify Results

Administrators and designated verifiers can review submitted results:

1. Navigate to Result Entry
2. Select the exam, class, and subject
3. Review all marks
4. Click **Verify** to confirm accuracy
5. Status changes to `verified`

### Reject Results

If errors are found:

1. Click **Reject**
2. Enter rejection reason
3. Results return to `draft` status
4. Teacher can correct and resubmit

### Publish Results

After verification:

1. Click **Publish Results**
2. Results become visible to students and parents
3. Status changes to `published`
4. Results appear in report cards and parent portal

---

## Result Status Flow

```
Draft ──→ Submitted ──→ Verified ──→ Published
  ↑          │              │
  │          │              │
  └── Rejected ←───────────┘
```

| Status | Who Can See | Who Can Edit |
|--------|-------------|-------------|
| Draft | Teacher | Teacher |
| Submitted | Teacher, Verifier | No one |
| Verified | Teacher, Verifier | Verifier |
| Published | Everyone | No one |

---

## Result Database

```sql
CREATE TABLE wp_ik_exam_results (
    result_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    schedule_id BIGINT UNSIGNED NOT NULL,
    student_id BIGINT UNSIGNED NOT NULL,
    marks_obtained DECIMAL(6,2),
    marks_absent TINYINT(1) DEFAULT 0,
    grade_letter VARCHAR(5),
    remarks TEXT,
    entered_by BIGINT UNSIGNED NOT NULL,
    verified_by BIGINT UNSIGNED,
    verified_at DATETIME,
    is_verified TINYINT(1) DEFAULT 0,
    status VARCHAR(20) DEFAULT 'draft',
    campus_id BIGINT UNSIGNED NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY unique_result (schedule_id, student_id),
    INDEX idx_student_id (student_id),
    INDEX idx_status (status)
);
```

**Unique constraint**: One result per student per exam schedule.

---

## Pending Results Count

The exam dashboard tracks pending results:

```php
public function get_pending_results_count($campus_id) {
    global $wpdb;
    return $wpdb->get_var($wpdb->prepare(
        "SELECT COUNT(DISTINCT s.schedule_id)
         FROM {$schedules_table} s
         LEFT JOIN {$results_table} r ON s.schedule_id = r.schedule_id
         WHERE s.campus_id = %d
         AND s.exam_date <= CURDATE()
         AND (r.status IS NULL OR r.status = 'draft')",
        $campus_id
    ));
}
```

---

## Data Sanitization

All result data is recursively sanitized before processing:

```php
private function sanitize_recursive($data) {
    $sanitized = [];
    foreach ($data as $key => $value) {
        if (is_array($value)) {
            $sanitized[sanitize_key($key)] = $this->sanitize_recursive($value);
        } elseif (is_string($value)) {
            $sanitized[sanitize_key($key)] = sanitize_text_field($value);
        } else {
            $sanitized[sanitize_key($key)] = $value;
        }
    }
    return $sanitized;
}
```

---

## Programmatic Usage

### Save Results

```php
$exam_module = IK_Exam_Module::get_instance();

$exam_module->ajax_save_results([
    'schedule_id' => $schedule_id,
    'results'     => [
        ['student_id' => 101, 'marks' => 85, 'max_marks' => 100],
        ['student_id' => 102, 'marks' => 72, 'max_marks' => 100],
        ['student_id' => 103, 'marks' => 0, 'absent' => 1],
    ],
]);
```

### Verify Results

```php
$exam_module->ajax_verify_results([
    'schedule_id' => $schedule_id,
]);
```

### Publish Results

```php
$exam_module->ajax_publish_results([
    'schedule_id' => $schedule_id,
]);
```

### Get Results for a Schedule

```php
$results = $exam_module->ajax_get_results([
    'schedule_id' => $schedule_id,
]);
```

### Get Students for Result Entry

```php
$students = $exam_module->ajax_get_students_for_result([
    'class_id'   => $class_id,
    'subject_id' => $subject_id,
]);
```

---

## Grade Scale Lookup

```php
// Get grade scale (cached)
$grade_scale = $exam_module->get_grade_scale();

// Calculate grade letter
$grade = $exam_module->calculate_grade($marks_obtained, $max_marks);

// Calculate GPA
$gpa = $exam_module->calculate_gpa($marks_obtained, $max_marks);
```
