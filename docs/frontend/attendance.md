```markdown
# Frontend Attendance

The Frontend Attendance module enables teachers to mark daily student attendance from the frontend of your website, with campus restrictions, section filtering, and read-only mode for previously saved records.

---

## Setup

### 1. Install the Plugin

Upload and activate the `institutionkit-frontend-attendance` plugin.

### 2. Create an Attendance Page

1. Create a new WordPress page (e.g., "Attendance" or "Mark Attendance")
2. Add the shortcode:

```
[institutionkit_attendance]
```

3. Publish the page
4. Restrict access to logged-in teachers if needed

---

## Teacher Authentication

Only teachers, campus admins, and administrators can access the attendance form:

```php
$user = wp_get_current_user();
$is_admin = current_user_can('manage_options') || current_user_can('manage_campus');
```

Non-admin users are restricted to their assigned campus.

---

## Filter Form

Before marking attendance, teachers select:

| Filter | Description | Required |
|--------|-------------|:---:|
| Campus | Available only to administrators | Admin only |
| Class | Target class | Yes |
| Section | Filter by section | Yes (if sections exist) |
| Date | Attendance date | Yes (defaults to today) |

Click **Show Student List** to load students.

---

## Campus Restriction

### For Administrators

All campuses are visible. A campus dropdown appears for filtering.

### For Teachers

Teachers are restricted to their primary campus from the staff table:

```php
private function get_teacher_campus_ids() {
    $user = wp_get_current_user();
    if (!in_array('teacher', $user->roles) && !in_array('campus_admin', $user->roles)) {
        return [];
    }
    
    global $wpdb;
    $staff_id = $wpdb->get_var($wpdb->prepare(
        "SELECT staff_id FROM {$wpdb->prefix}institutionkit_staff WHERE user_id = %d LIMIT 1",
        $user->ID
    ));
    
    if ($staff_id) {
        $campus_id = $wpdb->get_var($wpdb->prepare(
            "SELECT primary_campus_id FROM {$wpdb->prefix}institutionkit_staff WHERE staff_id = %d",
            $staff_id
        ));
        return $campus_id ? [$campus_id] : [];
    }
    
    return [];
}
```

### Class Access Control

Teachers can only view classes within their campus:

```php
private function is_class_allowed_for_user($class_id, $selected_campus_id, $is_admin, $teacher_campus_ids) {
    $class_campus = get_post_meta($class_id, '_ik_campus_id', true);
    $class_campus = $class_campus ? (int)$class_campus : 0;
    
    // Global classes (no campus) are allowed
    if ($class_campus === 0) return true;
    
    if ($is_admin) {
        return $selected_campus_id == 0 || $class_campus == $selected_campus_id;
    }
    
    return in_array($class_campus, $teacher_campus_ids);
}
```

---

## Section Requirement

If the system has any sections defined, selecting a section is **mandatory** before students appear:

```php
$all_sections = get_terms(['taxonomy' => 'ik_section', 'hide_empty' => false, 'fields' => 'ids']);
$has_any_sections = !is_wp_error($all_sections) && !empty($all_sections);

if ($has_any_sections && empty($selected_section)) {
    echo '<div class="ik-card"><p>Please select a section to view students.</p></div>';
    return;
}
```

---

## Student List Grid

Each student row displays:

| Column | Content |
|--------|---------|
| Student Name | Full name in bold |
| Roll Number | `_ik_roll_number` |
| Status | Radio buttons: Present / Absent / Late / On Leave |
| Parent Contacts | Father's and mother's phone numbers |
| Remarks | Text input (shown for Late/Leave) |

### Status Options

| Status | Remarks Field | CSS Class |
|--------|:---:|-----------|
| Present | Hidden | `ik-status-present` |
| Absent | Hidden | `ik-status-absent` |
| Late | **Shown** | `ik-status-late` |
| On Leave | **Shown** | `ik-status-leave` |

---

## Dynamic Remarks Field

The remarks field appears or hides based on status selection:

```javascript
function toggleRemarksField(studentId, status) {
    var wrapper = document.getElementById('remarks-wrapper-' + studentId);
    if (wrapper) {
        wrapper.style.display = (status === 'late' || status === 'leave') ? 'block' : 'none';
    }
}
```

---

## Saving Attendance

### First Save

Records are inserted into `institutionkit_attendance`:

```php
$wpdb->insert($table_name, [
    'student_id'       => $student_id,
    'class_id'         => $class_id,
    'attendance_date'  => $attendance_date,
    'status'           => $status,
    'remarks'          => $remarks,
    'marked_by'        => get_current_user_id(),
    'marked_at'        => current_time('mysql'),
]);
```

### Subsequent Saves (Upsert)

Existing records are updated rather than duplicated:

```php
$existing = $wpdb->get_var($wpdb->prepare(
    "SELECT attendance_id FROM {$table} 
     WHERE student_id = %d AND attendance_date = %s AND class_id = %d",
    $student_id, $attendance_date, $class_id
));

if ($existing) {
    $wpdb->update($table, $data, ['attendance_id' => $existing]);
} else {
    $wpdb->insert($table, $data);
}
```

---

## Read-Only Mode

When attendance has already been saved for a date, the form becomes read-only:

```
⚠️ Attendance for this date has been saved and is now read-only 
from the front end. It can be modified by an administrator from 
the back end.
```

The submitting user and time are displayed:

```
Attendance submitted by Sarah Ahmed at 08:45 AM
```

Radio buttons and remarks fields are disabled. The save button is hidden.

---

## Remarks Logic

Remarks are cleared for Present and Absent status:

```php
if (in_array($status, ['present', 'absent'])) {
    $remarks = '';
}
```

Only Late and Leave statuses retain remarks.

---

## Parent Contact Display

Each student row shows available parent contacts:

```
Father's Contact: +92 300 1234567 | Mother's Contact: +92 300 7654321
```

If no contacts are found:

```
No parent contact number found.
```

---

## Redirect After Save

After saving, the form redirects with the current filters preserved:

```php
$redirect_args = [
    'class_id'          => $class_id,
    'attendance_date'   => $attendance_date,
    'attendance_saved'  => 'true',
    'filter_attendance' => 'Show',
];

if (isset($_POST['campus_id'])) {
    $redirect_args['campus_id'] = absint($_POST['campus_id']);
}
if (!empty($section)) {
    $redirect_args['section'] = sanitize_text_field($section);
}

wp_safe_redirect(add_query_arg($redirect_args, $current_page_url));
exit;
```

---

## Success Message

After a successful save:

```
✅ Student attendance saved successfully!
```

---

## Full-Width Layout

The attendance page uses a full-width wrapper to override theme constraints:

```html
<div class="ik-attendance-full-wrapper" 
     style="width: 100%; margin: 0; padding: 20px; 
            box-sizing: border-box; background: #FFF;">
    <div class="ik-attendance-wrap" 
         style="max-width: none !important; width: 95%; margin: 0 auto;">
        <!-- Content -->
    </div>
</div>
```

---

## Programmatic Usage

### Render the Attendance Form

```php
$manager = IK_Frontend_Attendance_Manager::instance();
$manager->render_view();
```

### Get Teacher Campus IDs

```php
$campus_ids = $manager->get_teacher_campus_ids();
```

### Check Class Access

```php
$allowed = $manager->is_class_allowed_for_user(
    $class_id, 
    $campus_id, 
    $is_admin, 
    $teacher_campus_ids
);
```
