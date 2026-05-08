```markdown
# Frontend Gradebook

The Frontend Gradebook module provides teachers with a complete grade entry interface on the frontend of your website — with period selection, class and subject filtering, real-time grade calculation, bulk operations, and keyboard navigation.

---

## Setup

### 1. Install the Plugin

Upload and activate the `institutionkit-frontend-gradebook` plugin.

### 2. Create a Gradebook Page

1. Create a new WordPress page (e.g., "Gradebook" or "Enter Grades")
2. Add the shortcode:

```
[institutionkit_gradebook]
```

3. Publish the page
4. Restrict access to logged-in teachers

---

## Authentication

### Login Requirement

If not logged in, a styled login prompt appears:

```
┌──────────────────────────────────┐
│           👤                      │
│      Gradebook Access             │
│                                    │
│  Please log in to access the      │
│  gradebook.                       │
│                                    │
│      [ Login to Continue ]        │
└──────────────────────────────────┘
```

### Role Requirement

Only teachers, campus admins, and administrators can access:

```php
$user = wp_get_current_user();
if (!in_array('teacher', $user->roles) && 
    !in_array('campus_admin', $user->roles) && 
    !current_user_can('manage_options')) {
    return 'You do not have permission to access the gradebook.';
}
```

---

## Campus Restriction

Teachers can only see students from their assigned campus:

```php
private function get_teacher_campus_id() {
    $user = wp_get_current_user();
    if (current_user_can('manage_options')) return 0;
    
    global $wpdb;
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

Students are filtered by campus before display. Administrators see all students.

---

## Filter Form

### Period Type Filter

| Option | Description |
|--------|-------------|
| All Types | Show all periods |
| Weekly | Weekly assessment periods |
| Monthly | Monthly test periods |
| Quarterly | Quarterly exam periods |
| Yearly | Yearly final exam periods |
| Exam | Specific exam periods |

### Period Filter

After selecting a period type, specific periods appear. Periods show title and type.

### Class Filter

Select the target class. Subjects load dynamically based on class selection.

### Section Filter

Optional. Filter students by section if sections exist.

### Subject Filter

Subjects assigned to the selected class appear. Required before the grade grid loads.

---

## Grade Grid Interface

### Header

```
📊 October 2025 Monthly Assessment
Grade 5 / Mathematics
                         25 Students
```

### Toolbar

```
Default Max Marks: [100] [Apply to All]
[Fill All Marks] [Set Max Marks]
```

### Grid Columns

| Column | Description | Editable |
|--------|-------------|:---:|
| Student | Name with colored avatar | No |
| Max Marks | Maximum possible marks | Yes |
| Marks | Obtained marks | Yes |
| % | Auto-calculated percentage | No |
| Grade | Auto-calculated letter grade | No |
| Remarks | Teacher comments | Yes |

---

## Real-Time Calculation

As marks are entered, percentage and grade update live:

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

### Grade Scale

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

### Apply Global Max Marks

1. Set a value in the **Default Max Marks** field
2. Click **Apply to All**
3. All student max marks fields update

### Fill All Marks

1. Click **Fill All Marks**
2. Enter marks value in prompt
3. All marks fields populate
4. Percentages and grades update automatically

### Set Max Marks

1. Click **Set Max Marks**
2. Enter max marks value (default: 100)
3. All max marks fields update
4. Global max marks field also updates

---

## Keyboard Navigation

Press **Enter** in a marks field to move to the next student. On the last student, pressing Enter submits the form:

```javascript
$('.ik-marks-input').on('keydown', function(e) {
    if (e.which === 13) {
        e.preventDefault();
        var $inputs = $('.ik-marks-input');
        var idx = $inputs.index(this);
        if (idx < $inputs.length - 1) {
            $inputs.eq(idx + 1).focus().select();
        } else {
            $('#ik-gradebook-form').submit();
        }
    }
});
```

---

## Saving Grades

### AJAX Submission

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
                window.location.href = currentUrl + '&saved=1';
            }
        }
    });
});
```

### Server-Side UPSERT

```php
foreach ($grades as $student_id => $data) {
    $existing = $wpdb->get_var($wpdb->prepare(
        "SELECT grade_id FROM {$grades_table} 
         WHERE student_id = %d AND subject_id = %d AND period_id = %d",
        $student_id, $subject_id, $period_id
    ));
    
    if ($existing) {
        $wpdb->update($grades_table, $grade_data, ['grade_id' => $existing]);
    } elseif ($marks !== null) {
        $wpdb->insert($grades_table, $grade_data);
    }
}
```

### Success State

After saving, the page reloads with a success banner:

```
✅ Grades saved successfully!
```

---

## Student Avatars

Each student row shows a colored avatar with the first letter of their name:

```
┌────┐
│ A  │ Ahmed Khan
└────┘
```

Generated via CSS gradient with the student's initial.

---

## Empty States

### No Students Found

```
ℹ️ No students found in this class/section.
```

### No Subjects

Subjects dropdown shows placeholder until a class is selected.

---

## Responsive Design

- Desktop: All columns visible
- Tablet: Horizontal scroll for the grade table
- Mobile: Stacked filter form, compact grade grid

---

## Programmatic Usage

### Render the Gradebook

```php
echo do_shortcode('[institutionkit_gradebook]');
```

### Get Subjects for a Class

```php
// AJAX endpoint
add_action('wp_ajax_ik_frontend_get_subjects', function() {
    $class_id = absint($_POST['class_id']);
    $subjects = wp_get_post_terms($class_id, 'ik_subject');
    wp_send_json_success($subjects);
});
```

### Save Grades Programmatically

```php
// AJAX endpoint
add_action('wp_ajax_ik_frontend_save_grades', function() {
    // Verify nonce
    // Verify role
    // Process grades (upsert)
    // Return success
});
```
