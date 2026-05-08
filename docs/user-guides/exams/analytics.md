```markdown
# Exam Analytics

The Analytics module provides comprehensive performance analysis — class-wide trends, subject-level breakdowns, student comparisons, and visual charts for data-driven decision making.

---

## Accessing Analytics

Navigate to **InstitutionKit → Exams → Analytics**.

---

## Analytics Dashboard

The analytics page provides multiple views of exam data:

| Section | Description |
|---------|-------------|
| Class Performance | Average performance by class |
| Subject Analysis | Performance breakdown by subject |
| Student Performance | Individual student tracking |
| Comparative Analysis | Cross-campus or cross-exam comparison |

---

## Class Performance

### What It Shows

Aggregated performance statistics for each class:

| Stat | Calculation |
|------|-------------|
| Average % | Mean of all student percentages |
| Highest % | Top score in the class |
| Lowest % | Bottom score in the class |
| Pass % | Students above passing threshold |
| Students | Total students with results |

### Visualization

A bar chart comparing class averages:

```
Class Performance
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Grade 5    ████████████████████ 78.5%
Grade 4    ██████████████████░░ 72.3%
Grade 3    ███████████████████░ 75.1%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Subject Analysis

### What It Shows

Performance broken down by subject:

| Metric | Description |
|--------|-------------|
| Subject Average | Mean percentage for this subject |
| Above Average | Students scoring above class average |
| Below Average | Students scoring below class average |
| Top Performers | Students with highest marks |
| Needs Improvement | Students below passing threshold |

### Subject Comparison

```
Subject Analysis - Grade 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Mathematics  ████████████████████ 82.1%
English      ██████████████████░░ 76.4%
Science      ███████████████████░ 79.8%
History      ██████████████░░░░░░ 65.2%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Student Performance

### Individual Student View

Select a student to see:

- Performance across all exams
- Subject-wise breakdown
- Progress over time (line chart)
- Comparison to class average
- Strengths and weaknesses identification

### Progress Tracking

```
Student: Ahmed Khan — Progress Over Time
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Monthly 1   ██████████████░░░░░░ 68%
Monthly 2   ██████████████████░░ 76%
Midterm     ███████████████████░ 82%
Final       ████████████████████ 88%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Comparative Analysis

### Cross-Exam Comparison

Compare performance across different exam types:

| Exam | Average % | Pass % | Top Score |
|------|-----------|--------|-----------|
| Monthly Test 1 | 72% | 85% | 98% |
| Monthly Test 2 | 75% | 88% | 96% |
| Midterm | 78% | 90% | 99% |
| Final | 82% | 92% | 100% |

### Cross-Campus Comparison (Super Admin)

Compare exam performance across campuses:

```
Campus Comparison — Final Exam 2026
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Main Campus      ████████████████████ 85.2%
Downtown         ██████████████████░░ 78.7%
Northwest        ███████████████████░ 82.1%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Analytics Filters

All analytics views support these filters:

| Filter | Description |
|--------|-------------|
| Exam Type | Filter by specific examination |
| Class | Filter by class |
| Subject | Filter by subject |
| Academic Year | Filter by academic year |
| Date Range | Filter by custom date range |
| Campus | Filter by campus |

---

## Data Sources

Analytics pull from multiple tables:

| Data | Source Table |
|------|-------------|
| Results | `ik_exam_results` |
| Schedules | `ik_exam_schedules` |
| Exam Types | `ik_exam_types` |
| Report Cards | `ik_report_cards` |
| Grade Scales | `ik_grade_scales` |

---

## Key Metrics

### Pass Rate

```
Pass Rate = (Students above passing marks ÷ Total students) × 100
```

### Class Average

```
Class Average = Sum of all student percentages ÷ Number of students
```

### Subject Difficulty Index

```
Difficulty Index = (Max Marks - Class Average) ÷ Max Marks × 100
```

Higher difficulty index = more challenging subject.

### Performance Trend

```
Trend = Current Average - Previous Average
```

Positive = improving. Negative = declining.

---

## Export Options

Analytics data can be exported:

| Format | Use Case |
|--------|----------|
| CSV | Spreadsheet analysis |
| PDF | Formal reports |
| Chart Images | Presentations |

---

## Programmatic Usage

### Get Class Performance

```php
$exam_module = IK_Exam_Module::get_instance();

$class_data = $exam_module->ajax_get_class_performance([
    'exam_type_id'  => $exam_type_id,
    'class_id'      => $class_id,
    'campus_id'     => $campus_id,
]);
```

### Get Student Performance

```php
$student_data = $exam_module->ajax_get_student_performance([
    'student_id'    => $student_id,
    'academic_year' => '2025-2026',
]);
```

### Get Subject Analysis

```php
$subject_data = $exam_module->ajax_get_subject_analysis([
    'subject_id'    => $subject_id,
    'exam_type_id'  => $exam_type_id,
    'campus_id'     => $campus_id,
]);
```

### Get Comparative Analysis

```php
$comparison = $exam_module->ajax_get_comparative_analysis([
    'exam_type_ids' => [1, 2, 3],
    'campus_ids'    => [1, 2],
    'academic_year'  => '2025-2026',
]);
```

---

## Dashboard Integration

Exam analytics appear on the main dashboard:

- **Class Performance Chart** — Top 5 classes by average
- **Subject Performance Chart** — Top 5 subjects by average

Both are rendered as Chart.js bar charts with campus filtering applied.
```
