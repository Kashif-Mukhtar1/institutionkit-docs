```markdown
# Gradebook Overview

The Gradebook module provides a comprehensive multi-period grading system — allowing teachers to enter student grades by subject, grading period, and class, with automatic grade calculation and performance tracking.

---

## Accessing Gradebook

| Interface | Path | Who Uses It |
|-----------|------|-------------|
| **Admin Panel** | InstitutionKit → Student Management → Gradebook | Admins, Campus Admins |
| **Frontend** | Page with `[institutionkit_gradebook]` shortcode | Teachers |

---

## Gradebook Architecture

InstitutionKit uses a **multi-period gradebook system** (v2):

```
Grading Periods (ik_period CPT)
    │
    ├── Weekly Assessments
    ├── Monthly Tests
    ├── Quarterly Exams
    ├── Yearly Finals
    └── Custom Exams
        │
        ▼
Grade Entries (ik_grades_v2 table)
    │
    ├── Student + Subject + Period (unique)
    ├── Marks Obtained + Max Marks
    ├── Auto-calculated Grade Letter
    └── Teacher Remarks
```

---

## Grading Periods

Grading periods define the time frames for grade entry.

### Period Types

| Type | Typical Use | Frequency |
|------|-------------|-----------|
| **Weekly** | Weekly assessments | Every week |
| **Monthly** | Monthly tests | Every month |
| **Quarterly** | Term assessments | Every 3 months |
| **Yearly** | Final exams | Once per year |
| **Exam** | Specific examinations | As scheduled |

### Period Properties

| Property | Description |
|----------|-------------|
| Title | Descriptive name |
| Period Type | One of the five types |
| Start Date | Period start |
| End Date | Period end |
| Academic Year | e.g., "2025-2026" |
| Weight | For averaging (default 1.00) |
| Published | Visible to students/parents |

### Weight System

Weights determine how much a period contributes to final grades:

```
Weighted Average = SUM(Percentage × Weight) / SUM(Weights)
```

Example with three periods:

| Period | Weight | Percentage | Weighted |
|--------|--------|------------|----------|
| Weekly 1 | 1.0 | 78% | 78.0 |
| Monthly | 2.0 | 82% | 164.0 |
| Term Exam | 3.0 | 88% | 264.0 |
| **Weighted Average** | | | **84.3%** |

---

## Grade Scales

Grade scales map percentages to letter grades and GPA points.

### Default Scale

| Min % | Max % | Grade | GPA |
|-------|-------|-------|-----|
| 90.00 | 100.00 | A+ | 4.00 |
| 85.00 | 89.99 | A | 4.00 |
| 80.00 | 84.99 | A- | 3.70 |
| 77.00 | 79.99 | B+ | 3.30 |
| 73.00 | 76.99 | B | 3.00 |
| 70.00 | 72.99 | B- | 2.70 |
| 67.00 | 69.99 | C+ | 2.30 |
| 63.00 | 66.99 | C | 2.00 |
| 60.00 | 62.99 | C- | 1.70 |
| 50.00 | 59.99 | D | 1.00 |
| 0.00 | 49.99 | F | 0.00 |

### Campus-Specific Scales

Each campus can override the default scale with custom grade mappings.

---

## Grade Calculation

### Percentage

```
Percentage = (Marks Obtained / Max Marks) × 100
```

### Letter Grade

```php
public function calculate_grade($marks_obtained, $max_marks) {
    $percentage = ($marks_obtained / $max_marks) * 100;
    $grade_scale = $this->get_grade_scale();
    
    foreach ($grade_scale as $grade) {
        if ($percentage >= $grade['min_percent'] && $percentage <= $grade['max_percent']) {
            return $grade['letter_grade'];
        }
    }
    return 'F';
}
```

### GPA

```php
public function calculate_gpa($marks_obtained, $max_marks) {
    $percentage = ($marks_obtained / $max_marks) * 100;
    $grade_scale = $this->get_grade_scale();
    
    foreach ($grade_scale as $grade) {
        if ($percentage >= $grade['min_percent'] && $percentage <= $grade['max_percent']) {
            return (float) $grade['gpa_points'];
        }
    }
    return 0.00;
}
```

---

## Grade Database (v2)

```sql
CREATE TABLE wp_ik_grades_v2 (
    grade_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    student_id BIGINT(20) UNSIGNED NOT NULL,
    subject_id BIGINT(20) UNSIGNED NOT NULL,
    class_id BIGINT(20) UNSIGNED NOT NULL,
    period_id BIGINT(20) UNSIGNED NOT NULL,
    period_type VARCHAR(20) NOT NULL,
    marks_obtained DECIMAL(6,2),
    max_marks DECIMAL(6,2) DEFAULT 100.00,
    grade_letter VARCHAR(5),
    remarks TEXT,
    teacher_id BIGINT(20) UNSIGNED,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY unique_grade (student_id, subject_id, period_id)
);
```

**Unique constraint**: One grade per student per subject per period. Prevents duplicate entries.

---

## Frontend Gradebook

The frontend gradebook at `[institutionkit_gradebook]` provides:

### Filter Form

| Filter | Description |
|--------|-------------|
| Period Type | Weekly / Monthly / Quarterly / Yearly / Exam |
| Period | Specific grading period |
| Class | Target class |
| Section | Optional section filter |
| Subject | Subject to enter grades for |

### Grade Grid

| Column | Description |
|--------|-------------|
| Student | Name with avatar |
| Max Marks | Maximum marks (default 100, editable) |
| Marks | Numeric input for obtained marks |
| % | Auto-calculated percentage |
| Grade | Auto-calculated letter grade |
| Remarks | Optional teacher comments |

### Bulk Actions

| Button | Action |
|--------|--------|
| **Fill All Marks** | Set all students to the same marks value |
| **Set Max Marks** | Set all students to the same max marks |
| **Apply to All** | Apply the global max marks value |

### Keyboard Navigation

Press **Enter** in a marks field to move to the next student's field. On the last student, Enter submits the form.

### Real-Time Calculation

As marks are entered, percentages and grades update live:

```javascript
function calculatePercentage(studentId) {
    var marks = parseFloat($('input[name="grades[' + studentId + '][marks]"]').val()) || 0;
    var max = parseFloat($('input[name="grades[' + studentId + '][max_marks]"]').val()) || 100;
    var percentage = max > 0 ? (marks / max) * 100 : 0;
    var grade = getGradeLetter(percentage);
    $('#percentage-' + studentId).text(percentage.toFixed(2) + '%');
    $('#grade-' + studentId).text(grade);
}
```

---

## Teacher Campus Restriction

Frontend gradebook access is campus-restricted:

```php
private function get_teacher_campus_id() {
    $user = wp_get_current_user();
    if (current_user_can('manage_options')) return 0;
    
    $staff_id = $wpdb->get_var($wpdb->prepare(
        "SELECT staff_id FROM {$wpdb->prefix}institutionkit_staff WHERE user_id = %d",
        $user->ID
    ));
    
    return $wpdb->get_var($wpdb->prepare(
        "SELECT primary_campus_id FROM {$wpdb->prefix}institutionkit_staff WHERE staff_id = %d",
        $staff_id
    ));
}
```

Students are filtered by the teacher's campus before display.

---

## Grade Scale Caching

Grade scales are cached for performance:

```php
public function get_grade_scale() {
    $cache_key = 'ik_grade_scales';
    $scales = wp_cache_get($cache_key, 'institutionkit');
    
    if (false === $scales) {
        $scales = $wpdb->get_results($wpdb->prepare(
            "SELECT * FROM {$table} WHERE scale_type = %s OR campus_id = %d",
            'default', $this->get_current_campus_id()
        ), ARRAY_A);
        
        wp_cache_set($cache_key, $scales, 'institutionkit', HOUR_IN_SECONDS);
    }
    
    return $scales ?: [];
}
```

---

## Programmatic Usage

### Save Grades

```php
global $wpdb;
foreach ($grades as $student_id => $data) {
    $existing = $wpdb->get_var($wpdb->prepare(
        "SELECT grade_id FROM {$wpdb->prefix}ik_grades_v2 
         WHERE student_id = %d AND subject_id = %d AND period_id = %d",
        $student_id, $subject_id, $period_id
    ));
    
    if ($existing) {
        $wpdb->update("{$wpdb->prefix}ik_grades_v2", $grade_data, ['grade_id' => $existing]);
    } else {
        $wpdb->insert("{$wpdb->prefix}ik_grades_v2", $grade_data);
    }
}
```

### Get Student Grades

```php
global $wpdb;
$grades = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}ik_grades_v2 
     WHERE student_id = %d AND period_id = %d",
    $student_id, $period_id
));
```
