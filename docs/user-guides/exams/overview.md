```markdown
# Exams Management Overview

The Exams module provides a complete examination system — from defining exam types and creating schedules to entering results, generating report cards, and analyzing performance.

---

## Accessing Exams

Navigate to **InstitutionKit → Exams** in the admin menu.

---

## Dashboard Cards

The Exams landing page presents 5 action cards:

| Card | Destination | Description |
|------|-------------|-------------|
| 🏷️ **Exam Types** | Exam types list | Create and manage different exam categories |
| 📅 **Exam Schedule** | Schedule management | Create and manage exam date sheets |
| ✏️ **Result Entry** | Marks entry | Enter marks, verify, and publish results |
| 📄 **Report Cards** | Card generation | Generate and publish student report cards |
| 📊 **Analytics** | Performance analytics | View performance analytics and trends |

---

## Exam Workflow

```
1. Create Exam Types
   └── Define categories: Term, Midterm, Final, Quiz, Monthly, Assessment

2. Create Exam Schedule
   └── Set dates, times, subjects, rooms, invigilators, max marks

3. Enter Results
   └── Teachers enter marks → Marks submitted for verification

4. Verify Results
   └── Admin or verifier reviews → Marks verified

5. Publish Results
   └── Results published → Visible to students and parents

6. Generate Report Cards
   └── Comprehensive report cards → PDF download available

7. View Analytics
   └── Class performance, subject analysis, comparative reports
```

---

## Exam Categories

InstitutionKit supports 7 exam categories:

| Category | Description | Typical Use |
|----------|-------------|-------------|
| **Term** | Term examination | End-of-term assessment |
| **Midterm** | Mid-term examination | Mid-semester evaluation |
| **Final** | Final examination | Year-end comprehensive exam |
| **Monthly** | Monthly test | Regular progress assessment |
| **Board** | Board examination | External board exams |
| **Quiz** | Short quiz | Topic-specific quick assessment |
| **Assessment** | General assessment | Flexible evaluation type |

---

## Result Workflow States

Exam results follow a strict workflow:

```
Draft → Submitted → Verified → Published
  │        │           │           │
  │        │           │           └── Visible to students/parents
  │        │           │
  │        │           └── Admin/verifier confirms accuracy
  │        │
  │        └── Teacher completes entry, submits for review
  │
  └── Teacher entering marks, not yet complete
```

| State | Who Can Edit | Visible To |
|-------|-------------|------------|
| **Draft** | Entering teacher | Teacher only |
| **Submitted** | No one (must be unsubmitted) | Teacher, Verifier |
| **Verified** | Verifier only | Teacher, Verifier |
| **Published** | No one | Everyone (students, parents) |

---

## Grade Calculation

### Percentage Formula

```
Percentage = (Marks Obtained ÷ Max Marks) × 100
```

### Grade Assignment

Grades are assigned based on the configured grade scale:

| Percentage | Grade | GPA |
|------------|-------|-----|
| 98%+ | A++ | — |
| 95%+ | A+ | 4.00 |
| 90%+ | A | 4.00 |
| 80%+ | B | 3.00 |
| 65%+ | C | 2.00 |
| 55%+ | D | 1.00 |
| 40%+ | D | 1.00 |
| Below 40% | F | 0.00 |

!!! note "Configurable Scales"
    Grade scales are fully configurable in Settings. Each campus can have its own scale.

---

## Exam Module Architecture

The Exams module uses a singleton pattern with 5 specialized traits:

```php
class IK_Exam_Module {
    use IK_Exam_Types;        // Exam type CRUD
    use IK_Exam_Schedule;     // Schedule management
    use IK_Result_Entry;      // Marks entry and verification
    use IK_Report_Cards;      // Report card generation
    use IK_Exam_Analytics;    // Performance analytics
}
```

All sub-module pages are rendered through these traits.

---

## Centralized AJAX Router

All exam AJAX operations flow through a single handler:

```php
public function ajax_handler() {
    $action = $_POST['method'];  // e.g., 'save_exam_type', 'save_results'
    
    switch ($action) {
        case 'save_exam_type':       return $this->ajax_save_exam_type($data);
        case 'save_schedule':        return $this->ajax_save_schedule($data);
        case 'save_results':         return $this->ajax_save_results($data);
        case 'verify_results':       return $this->ajax_verify_results($data);
        case 'publish_results':      return $this->ajax_publish_results($data);
        case 'generate_report_card': return $this->ajax_generate_report_card($data);
        // ... 20+ actions
    }
}
```

---

## Exam Data Relationships

```
ik_exam_types (1)
    │
    └── ik_exam_schedules (N)
        │
        ├── Subjects + Classes + Dates
        │
        └── ik_exam_results (N)
            │
            └── Per student per schedule (UNIQUE)
                │
                └── ik_report_cards (N)
                    └── Aggregated per student per exam type
```

---

## Campus Integration

Exam types, schedules, and results are campus-scoped:

- Campus Admins see only their campus exams
- Super Admins can filter by campus or view all
- Report cards include campus name in the header

---

## Capabilities

| Capability | Who Has It | Purpose |
|-----------|------------|---------|
| `ik_manage_exam_types` | Admin | Create/edit/delete exam types |
| `ik_manage_exam_schedules` | Admin, Campus Admin, Teacher | Create exam schedules |
| `ik_enter_exam_results` | Admin, Campus Admin, Teacher | Enter student marks |
| `ik_verify_exam_results` | Admin | Verify submitted results |
| `ik_publish_exam_results` | Admin | Publish results to students |
| `ik_generate_report_cards` | Admin, Campus Admin, Teacher | Generate report cards |
| `ik_view_exam_analytics` | Admin, Campus Admin, Teacher | View performance analytics |

---

## Dashboard Statistics

The exam dashboard shows real-time statistics:

| Stat | Source |
|------|--------|
| Exam Types | Count of active exam types for the campus |
| Active Schedules | Count of schedules with exam_date >= today |
| Pending Results | Count of schedules past exam date with draft results |
| Published Cards | Count of published report cards |
| Recent Exams | Last 5 exams with result status |
| Upcoming Exams | Next 5 scheduled exams |

---

## Quick Reference

### Get Exam Types

```php
$exam_types = $exam_module->get_exam_types([
    'campus_id'   => $campus_id,
    'active_only' => true,
]);
```

### Get Student Results

```php
global $wpdb;
$results = $wpdb->get_results($wpdb->prepare(
    "SELECT 
        er.*,
        es.subject_id,
        es.exam_date
     FROM {$wpdb->prefix}ik_exam_results er
     JOIN {$wpdb->prefix}ik_exam_schedules es ON er.schedule_id = es.schedule_id
     WHERE er.student_id = %d
     ORDER BY es.exam_date DESC",
    $student_id
));
```

### Calculate Grade

```php
$percentage = ($marks_obtained / $max_marks) * 100;

foreach ($grade_scale as $grade) {
    if ($percentage >= $grade['min_percent'] && $percentage <= $grade['max_percent']) {
        return $grade['letter_grade'];
    }
}
return 'F';
```
