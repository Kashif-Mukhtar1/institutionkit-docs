```markdown
# Grade Entry

Grade Entry is the primary interface for teachers to record student marks. It supports both admin panel and frontend access, with real-time grade calculation, bulk operations, and keyboard navigation for efficient data entry.

---

## Accessing Grade Entry

| Interface | Path | Best For |
|-----------|------|----------|
| **Admin Panel** | InstitutionKit → Student Management → Gradebook | Admins, Campus Admins |
| **Frontend** | Page with `[institutionkit_gradebook]` shortcode | Teachers |

---

## Grade Entry Workflow

```
1. Select Period
   └── Choose grading period (weekly, monthly, quarterly, yearly, exam)

2. Select Class & Section
   └── Filter students by class and optional section

3. Select Subject
   └── Choose the subject to enter grades for

4. Enter Marks
   └── Fill in marks obtained and max marks for each student

5. Add Remarks (optional)
   └── Teacher comments for individual students

6. Save
   └── AJAX save to ik_grades_v2 table
```

---

## Filter Form

### Period Type Filter

| Option | Description |
|--------|-------------|
| All Types | Show all periods regardless of type |
| Weekly | Only weekly assessment periods |
| Monthly | Only monthly test periods |
| Quarterly | Only quarterly exam periods |
| Yearly | Only yearly final exam periods |
| Exam | Only specific exam periods |

### Period Filter

After selecting a period type, choose the specific period from the dropdown. Periods are listed chronologically.

### Class Filter

Select the class. Students are loaded based on this selection.

### Section Filter

Optional. If sections exist in the system, select one to filter students.

### Subject Filter

Select the subject. Only subjects assigned to the selected class appear.

---

## Grade Grid Interface

### Grid Columns

| Column | Description | Editable |
|--------|-------------|:---:|
| Student | Name with initial avatar | No |
| Max Marks | Maximum possible marks | Yes |
| Marks | Obtained marks | Yes |
| % | Auto-calculated percentage | No |
| Grade | Auto-calculated letter grade | No |
| Remarks | Teacher comments | Yes |

### Real-Time Calculation

As you enter marks, the percentage and grade update instantly:

```
Student: Ahmed Khan
Max Marks: 100
Marks: 85
%: 85.00%  ← Auto-calculated
Grade: A    ← Auto-calculated
```

### Grade Formula

```javascript
function getGradeLetter(percentage) {
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

## Bulk Operations

### Fill All Marks

Click **Fill All Marks** to set every student's marks to the same value:

1. Click the button
2. Enter the marks value in the prompt
3. All marks fields are populated
4. All percentages and grades update

### Set Max Marks

Click **Set Max Marks** to set every student's max marks to the same value:

1. Click the button
2. Enter the max marks value (default: 100)
3. All max marks fields are updated
4. The global max marks field also updates

### Apply Global Max

1. Set the desired max marks in the **Default Max Marks** field at the top
2. Click **Apply to All**
3. All student max marks update to match

---

## Keyboard Navigation

Press **Enter** in any marks field to move to the next student:

```
Student 1: [85] ← Enter
Student 2: [72] ← Enter
Student 3: [90] ← Enter
...
Last Student: [88] ← Enter → Form submits
```

On the last student, pressing Enter submits the form.

---

## Saving Grades

### AJAX Save

Grades are saved via AJAX without page reload:

```javascript
$('#ik-gradebook-form').on('submit', function(e) {
    e.preventDefault();
    var formData = $(this).serialize();
    formData += '&action=ik_frontend_save_grades';
    
    $.ajax({
        url: ajaxurl,
        type: 'POST',
        data: formData,
        success: function(response) {
            if (response.success) {
                // Redirect with success parameter
                window.location.href = currentUrl + '&saved=1';
            }
        }
    });
});
```

### Save Logic (Upsert)

For each student, the system checks if a grade already exists:

```php
foreach ($grades as $student_id => $data) {
    $existing = $wpdb->get_var($wpdb->prepare(
        "SELECT grade_id FROM {$grades_table} 
         WHERE student_id = %d AND subject_id = %d AND period_id = %d",
        $student_id, $subject_id, $period_id
    ));
    
    if ($existing) {
        // Update existing grade
        $wpdb->update($grades_table, $grade_data, ['grade_id' => $existing]);
    } elseif ($marks !== null) {
        // Insert new grade
        $wpdb->insert($grades_table, $grade_data);
    }
}
```

### Success Confirmation

After saving, a success banner appears:

```
✅ Grades saved successfully!
```

---

## Campus Restriction

In the frontend gradebook, teachers can only see students from their assigned campus:

```php
$campus_id = $this->get_teacher_campus_id();

if (!$is_admin && $campus_id > 0) {
    $args['meta_query'][] = [
        'relation' => 'OR',
        ['key' => '_ik_campus_id', 'value' => $campus_id, 'compare' => '='],
        ['key' => '_ik_campus_id', 'compare' => 'NOT EXISTS']
    ];
}
```

Administrators can view all campuses.

---

## Grade Data Structure

Each grade entry stored:

```php
$grade_data = [
    'student_id'     => $student_id,
    'subject_id'     => $subject_id,
    'class_id'       => $class_id,
    'period_id'      => $period_id,
    'period_type'    => $period_type,
    'marks_obtained' => $marks,
    'max_marks'      => $max,
    'grade_letter'   => $letter,
    'remarks'        => $remarks,
    'teacher_id'     => get_current_user_id(),
];
```

### Unique Constraint

One grade per student per subject per period:

```sql
UNIQUE KEY unique_grade (student_id, subject_id, period_id)
```

This prevents duplicate entries and enables the upsert pattern.

---

## Existing Grades Display

When loading a class/subject/period combination with existing grades:

- Marks fields are pre-filled with saved values
- Percentages and grades are calculated from saved data
- Teachers can modify and re-save

---

## Teacher Identification

The `teacher_id` column records which staff member entered the grade:

```php
$grade_data['teacher_id'] = get_current_user_id();
```

This references `staff_id` from the `institutionkit_staff` table.

---

## Programmatic Usage

### Save Grades Programmatically

```php
global $wpdb;
$grades_table = $wpdb->prefix . 'ik_grades_v2';

foreach ($students as $student) {
    $existing = $wpdb->get_var($wpdb->prepare(
        "SELECT grade_id FROM {$grades_table} 
         WHERE student_id = %d AND subject_id = %d AND period_id = %d",
        $student->ID, $subject_id, $period_id
    ));
    
    $data = [
        'student_id'     => $student->ID,
        'subject_id'     => $subject_id,
        'class_id'       => $class_id,
        'period_id'      => $period_id,
        'period_type'    => 'monthly',
        'marks_obtained' => 85.00,
        'max_marks'      => 100.00,
        'grade_letter'   => 'A',
        'teacher_id'     => $staff_id,
    ];
    
    if ($existing) {
        $wpdb->update($grades_table, $data, ['grade_id' => $existing]);
    } else {
        $wpdb->insert($grades_table, $data);
    }
}
```

### Get Grades for a Period

```php
global $wpdb;
$grades = $wpdb->get_results($wpdb->prepare(
    "SELECT 
        g.*,
        p.post_title as student_name
     FROM {$wpdb->prefix}ik_grades_v2 g
     JOIN {$wpdb->posts} p ON g.student_id = p.ID
     WHERE g.class_id = %d 
       AND g.period_id = %d 
       AND g.subject_id = %d
     ORDER BY p.post_title ASC",
    $class_id, $period_id, $subject_id
));
```
