```markdown
# Report Cards

The Report Cards module generates comprehensive student report cards with marks, grades, GPA, class rank, attendance, and teacher remarks — available as HTML view and PDF download.

---

## Accessing Report Cards

Navigate to **InstitutionKit → Exams → Report Cards**.

---

## Report Card List View

| Column | Description |
|--------|-------------|
| Student | Student name |
| Exam Type | The examination |
| Academic Year | e.g., "2025-2026" |
| Total Marks | Sum of all subject max marks |
| Obtained Marks | Sum of all marks scored |
| Percentage | Overall percentage |
| GPA | Calculated GPA |
| Grade | Overall grade |
| Rank | Class ranking |
| Status | Published / Draft |
| Actions | View / Download PDF / Publish |

---

## Generating Report Cards

### Individual Generation

1. Select a student
2. Select an exam type
3. Click **Generate Report Card**
4. System aggregates all subject results
5. Calculates total, percentage, GPA, and rank
6. Creates report card record

### Bulk Generation

1. Select exam type and class
2. Click **Generate All**
3. System generates report cards for all students in the class
4. Progress is shown as cards are generated

---

## Report Card Content

Each report card includes:

### Header

- Institution name and logo
- Campus name
- Academic year
- Exam type and name

### Student Information

| Field | Source |
|-------|--------|
| Student Name | Post title |
| Roll Number | `_ik_roll_number` |
| Class | Class post title |
| Section | `_ik_section` |

### Subject-Wise Marks

| Column | Description |
|--------|-------------|
| Subject | Subject name |
| Max Marks | Maximum marks |
| Marks Obtained | Scored marks |
| Percentage | Subject percentage |
| Grade | Letter grade |

### Summary

| Stat | Calculation |
|------|-------------|
| Total Marks | Sum of all max marks |
| Obtained Marks | Sum of all scored marks |
| Overall Percentage | (Obtained ÷ Total) × 100 |
| GPA | Calculated from grade scale |
| Overall Grade | Letter grade from overall percentage |
| Rank in Class | Position based on percentage |
| Attendance % | From attendance records |

### Remarks

| Field | Source |
|-------|--------|
| Teacher Remarks | Entered during report card generation |
| Principal Remarks | Entered during report card generation |

---

## Report Card Database

```sql
CREATE TABLE wp_ik_report_cards (
    card_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    student_id BIGINT UNSIGNED NOT NULL,
    exam_type_id BIGINT UNSIGNED NOT NULL,
    academic_year VARCHAR(9) NOT NULL,
    total_marks DECIMAL(8,2) DEFAULT 0,
    obtained_marks DECIMAL(8,2) DEFAULT 0,
    percentage DECIMAL(5,2) DEFAULT 0,
    gpa DECIMAL(3,2),
    grade_letter VARCHAR(5),
    rank_in_class INT,
    attendance_percentage DECIMAL(5,2),
    teacher_remarks TEXT,
    principal_remarks TEXT,
    pdf_url VARCHAR(500),
    is_published TINYINT(1) DEFAULT 0,
    published_at DATETIME,
    campus_id BIGINT UNSIGNED NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_student_id (student_id),
    INDEX idx_exam_type_id (exam_type_id),
    INDEX idx_academic_year (academic_year)
);
```

---

## Publishing Report Cards

### Publish Individual

Click **Publish** on any report card to make it visible to students and parents.

### Publish All

After bulk generation, click **Publish All** to publish all generated cards at once.

### Publication Effects

Once published:

- Report cards appear in the Parent Portal
- Students can view their results
- PDF download becomes available
- Card is locked from further edits

---

## PDF Report Cards

### Generating PDFs

Click **Download PDF** to generate a formatted PDF version of the report card.

### PDF Content

The PDF includes:

- Institution letterhead
- All marks and grades in a formatted table
- Watermark ("ORIGINAL" or "DUPLICATE")
- Generation date and time
- Computer-generated disclaimer

### PDF Generation Technology

PDFs are generated using Dompdf with the DejaVu Sans font for multi-language support.

---

## Class Ranking

### How Ranking Works

Students are ranked within their class based on overall percentage:

```php
// Get all report cards for the class and exam
$cards = $wpdb->get_results($wpdb->prepare(
    "SELECT card_id, percentage 
     FROM {$wpdb->prefix}ik_report_cards 
     WHERE exam_type_id = %d 
       AND student_id IN (SELECT ID FROM {$wpdb->posts} WHERE ...)
     ORDER BY percentage DESC",
    $exam_type_id
));

// Assign ranks
$rank = 1;
foreach ($cards as $card) {
    $wpdb->update(
        "{$wpdb->prefix}ik_report_cards",
        ['rank_in_class' => $rank],
        ['card_id' => $card->card_id]
    );
    $rank++;
}
```

### Tie Handling

Students with the same percentage receive the same rank. The next rank skips accordingly (Olympic ranking).

---

## Attendance Integration

Report cards include attendance percentage from the exam period:

```php
// Get attendance during exam period
$attendance = $wpdb->get_results($wpdb->prepare(
    "SELECT status, COUNT(*) as count 
     FROM {$wpdb->prefix}institutionkit_attendance 
     WHERE student_id = %d 
       AND attendance_date BETWEEN %s AND %s
     GROUP BY status",
    $student_id, $period_start, $period_end
));

// Calculate percentage
$total = array_sum(array_column($attendance, 'count'));
$present = // sum of present + late counts
$percentage = $total > 0 ? round(($present / $total) * 100, 2) : 0;
```

---

## Published Cards Count

The exam dashboard tracks published report cards:

```php
public function get_published_cards_count($campus_id) {
    global $wpdb;
    return $wpdb->get_var($wpdb->prepare(
        "SELECT COUNT(*) FROM {$wpdb->prefix}ik_report_cards 
         WHERE campus_id = %d AND is_published = 1",
        $campus_id
    ));
}
```

---

## Programmatic Usage

### Generate a Report Card

```php
$exam_module = IK_Exam_Module::get_instance();

$card = $exam_module->ajax_generate_report_card([
    'student_id'    => $student_id,
    'exam_type_id'  => $exam_type_id,
    'academic_year' => '2025-2026',
]);
```

### Bulk Generate

```php
$exam_module->ajax_bulk_generate_report_cards([
    'class_id'      => $class_id,
    'exam_type_id'  => $exam_type_id,
    'academic_year' => '2025-2026',
]);
```

### Publish a Report Card

```php
$exam_module->ajax_publish_report_card([
    'card_id' => $card_id,
]);
```

### Download Report Card PDF

```php
$exam_module->ajax_download_report_card([
    'card_id' => $card_id,
]);
```

### Get Report Card List

```php
$cards = $exam_module->ajax_get_report_card_list([
    'campus_id'     => $campus_id,
    'exam_type_id'  => $exam_type_id,
    'academic_year' => '2025-2026',
]);
```
