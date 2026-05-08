```markdown
# Exam Schedule

The Exam Schedule module creates and manages examination date sheets — assigning subjects to specific dates, times, rooms, and invigilators.

---

## Accessing Exam Schedule

Navigate to **InstitutionKit → Exams → Exam Schedule**.

---

## Schedule List View

| Column | Description |
|--------|-------------|
| Exam Type | The exam category |
| Class | Target class |
| Subject | Subject being examined |
| Date | Exam date |
| Time | Start - End time |
| Room | Room number or location |
| Max Marks | Maximum marks for this exam |
| Status | Draft or Published |
| Actions | Edit / Delete / Publish |

---

## Creating an Exam Schedule

Click **Create Schedule** or **Add New**.

### Schedule Form

| Field | Required | Description |
|-------|----------|-------------|
| Exam Type | Yes | Select from active exam types |
| Class | Yes | Target class for this exam |
| Subject | Yes | Subject being examined |
| Exam Date | Yes | Date of the examination |
| Start Time | Yes | Exam start time |
| End Time | Yes | Exam end time |
| Room Number | No | Room or location identifier |
| Invigilator | No | Staff member supervising |
| Max Marks | Yes | Maximum marks for this exam |
| Passing Marks | No | Minimum passing threshold |
| Campus | Yes | Campus (auto-filled for Campus Admins) |

---

## Publishing Schedules

Schedules start in **Draft** status. Click **Publish** to make them visible to teachers for result entry.

| Status | Effect |
|--------|--------|
| **Draft** | Not visible for result entry. Can be edited freely. |
| **Published** | Visible for result entry. Editing restricted. |

---

## Schedule Database

```sql
CREATE TABLE wp_ik_exam_schedules (
    schedule_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    exam_type_id BIGINT UNSIGNED NOT NULL,
    class_id BIGINT UNSIGNED NOT NULL,
    subject_id BIGINT UNSIGNED NOT NULL,
    exam_date DATE NOT NULL,
    start_time TIME NOT NULL,
    end_time TIME NOT NULL,
    room_number VARCHAR(50),
    invigilator_id BIGINT UNSIGNED,
    max_marks DECIMAL(6,2) NOT NULL,
    passing_marks DECIMAL(6,2),
    is_published TINYINT(1) DEFAULT 0,
    campus_id BIGINT UNSIGNED NOT NULL,
    created_by BIGINT UNSIGNED NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_exam_type_id (exam_type_id),
    INDEX idx_class_id (class_id),
    INDEX idx_exam_date (exam_date)
);
```

---

## Schedule Actions

### Edit Schedule

Modify any field of a draft schedule. Published schedules have editing restrictions.

### Delete Schedule

Remove a schedule. Cannot delete if results have been entered.

### Publish Schedule

Make the schedule available for result entry. Once published:

- Teachers can enter marks
- Schedule appears in result entry dropdowns
- Students can view the date sheet (if frontend enabled)

---

## Upcoming Exams Display

The Exam Dashboard shows the next 5 upcoming exam schedules:

```
┌──────────────────────────────────────────┐
│ Upcoming Exams                           │
│                                          │
│ Subject        Date        Time    Room  │
│ Mathematics    Jun 20    9:00-11:00  101 │
│ English        Jun 21    9:00-11:00  102 │
│ Science        Jun 22    9:00-11:00  103 │
└──────────────────────────────────────────┘
```

---

## Recent Exams Display

Past exams appear in the Recent Exams section with result status:

| Status | Badge |
|--------|-------|
| No results entered | Draft |
| Results being entered | Draft |
| Results submitted | Submitted |
| Results published | Published |
| Results verified | Completed |

---

## Programmatic Usage

### Create a Schedule

```php
$exam_module = IK_Exam_Module::get_instance();

$schedule_id = $exam_module->ajax_save_schedule([
    'exam_type_id'  => $exam_type_id,
    'class_id'      => $class_id,
    'subject_id'    => $subject_id,
    'exam_date'     => '2026-06-20',
    'start_time'    => '09:00',
    'end_time'      => '11:00',
    'room_number'   => '101',
    'max_marks'     => 100,
    'passing_marks' => 40,
    'campus_id'     => $campus_id,
]);
```

### Get Active Schedules

```php
$schedules = $exam_module->get_upcoming_schedules($campus_id, 10);
// Returns next 10 upcoming exam schedules
```

### Get Recent Exams

```php
$recent = $exam_module->get_recent_exams($campus_id, 5);
// Returns last 5 exams with result status
```

### Get Active Schedules Count

```php
$count = $exam_module->get_active_schedules_count($campus_id);
// Count of schedules with exam_date >= today
```

### Publish a Schedule

```php
$exam_module->ajax_publish_schedule([
    'schedule_id' => $schedule_id,
]);
```

### Delete a Schedule

```php
$exam_module->ajax_delete_schedule([
    'schedule_id' => $schedule_id,
]);
```

---

## Calendar Integration

Exam schedules integrate with the dashboard calendar:

- Scheduled exams appear as events
- Color-coded by exam type
- Click to view schedule details
```
