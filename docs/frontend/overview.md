```markdown
# Frontend Modules Overview

InstitutionKit includes four frontend modules that extend functionality to public-facing pages — enabling online admissions, teacher attendance marking, teacher gradebook access, and staff invoice management.

---

## Module Architecture

All frontend modules follow a consistent pattern:

```
WordPress Page
    │
    ├── Shortcode or dedicated URL
    │
    ├── Singleton Manager Class
    │   ├── Authentication check
    │   ├── Campus restriction
    │   ├── Form rendering
    │   ├── Form handling (POST)
    │   └── AJAX handlers
    │
    └── Standalone Template (Parent Portal only)
```

---

## Available Modules

| Module | Access Method | Users |
|--------|--------------|-------|
| **Frontend Admissions** | `[institutionkit_admission_form]` shortcode | Public (no login) |
| **Frontend Attendance** | `[institutionkit_attendance]` shortcode | Teachers, Campus Admins |
| **Frontend Gradebook** | `[institutionkit_gradebook]` shortcode | Teachers, Campus Admins |
| **Frontend Invoices** | `[ik_invoices]` shortcode | Staff with campus access |
| **Parent Portal** | `/parent-portal.php` standalone | Parents only |

---

## Common Security Patterns

### Authentication

```php
if (!is_user_logged_in()) {
    return $this->render_login_prompt();
}
```

### Role Verification

```php
if (!in_array('teacher', $user->roles) && 
    !in_array('campus_admin', $user->roles) && 
    !current_user_can('manage_options')) {
    return 'You do not have permission.';
}
```

### Campus Restriction

All teacher/staff-facing modules restrict data to the user's assigned campus:

```php
private function get_teacher_campus_id() {
    $user = wp_get_current_user();
    if (current_user_can('manage_options')) return 0;
    
    global $wpdb;
    $staff_id = $wpdb->get_var($wpdb->prepare(
        "SELECT staff_id FROM {$wpdb->prefix}institutionkit_staff 
         WHERE user_id = %d LIMIT 1",
        $user->ID
    ));
    
    if ($staff_id) {
        return $wpdb->get_var($wpdb->prepare(
            "SELECT primary_campus_id FROM {$wpdb->prefix}institutionkit_staff 
             WHERE staff_id = %d",
            $staff_id
        ));
    }
    
    // Fallback to IK_Campus_Manager
    if (class_exists('IK_Campus_Manager')) {
        $campuses = IK_Campus_Manager::get_user_campuses($user->ID);
        return !empty($campuses) ? $campuses[0] : 0;
    }
    
    return 0;
}
```

### Nonce Verification

```php
if (!wp_verify_nonce($_POST['nonce'], 'ik_action_nonce')) {
    wp_send_json_error(['message' => 'Security check failed.']);
}
```

---

## Campus Data Access

| User Type | Data Scope |
|-----------|-----------|
| **Administrator** (`manage_options`) | All campuses, no filter |
| **Campus Admin** | Their assigned campus only |
| **Teacher** | Their primary campus only |
| **Staff (Accountant)** | Their primary campus only |
| **Parent** | Their linked children only |

---

## Form Sanitization

All frontend modules sanitize input before processing:

```php
$data = [
    'student_name'   => sanitize_text_field($_POST['student_name']),
    'email'          => sanitize_email($_POST['email']),
    'class_id'       => absint($_POST['class_id']),
    'address'        => sanitize_textarea_field($_POST['address']),
    'id_card_number' => sanitize_text_field($_POST['id_card_number']),
];
```

---

## Validation Rules

Common validation patterns:

| Field | Rule |
|-------|------|
| Required text | `!empty()` check with error message |
| Email | `is_email()` WordPress function |
| ID Card | `preg_match('/^\d{13}$/', $value)` for 13-digit numbers |
| Class selection | `absint()` > 0 |
| Date | Date format validation |
| Amount | `floatval()` > 0 |

---

## Error Display

Errors are collected in an array and displayed to the user:

```php
private $form_errors = [];

private function validate_form_data() {
    if (empty($this->form_data['student_name'])) {
        $this->form_errors[] = 'Student\'s name is required.';
    }
    if (!is_email($this->form_data['email'])) {
        $this->form_errors[] = 'A valid email address is required.';
    }
}

// In template:
if (!empty($form_errors)) {
    foreach ($form_errors as $error) {
        echo '<div class="ik-form-error">' . esc_html($error) . '</div>';
    }
}
```

---

## Success Handling

Successful submissions typically redirect with a success parameter:

```php
if (empty($this->form_errors)) {
    $this->process_submission();
    wp_safe_redirect(add_query_arg('success', 'true', get_permalink()));
    exit;
}
```

---

## Plugin Files

### Admissions

```
/wp-content/plugins/institutionkit-frontend-admissions/
├── class-ik-frontend-admissions-manager.php
└── templates/
    └── admission-form-template.php
```

### Attendance

```
/wp-content/plugins/institutionkit-frontend-attendance/
└── class-ik-frontend-attendance-manager.php
```

### Gradebook

```
/wp-content/plugins/institutionkit-frontend-gradebook/
└── institutionkit-frontend-gradebook.php
```

### Invoices

```
/wp-content/plugins/institutionkit-frontend-invoices/
└── class-ik-frontend-invoices-manager.php
```

---

## Theme Compatibility

Frontend modules output custom HTML/CSS that may conflict with some themes. For best results:

1. **Use a lightweight theme** (Hello Elementor, GeneratePress, Kadence, Astra)
2. **Set pages to full-width template** — Removes sidebar interference
3. **Parent Portal is standalone** — No theme interference at all
4. **CSS is scoped** — All styles use `.ik-` prefixed classes to avoid conflicts

---

## Shortcode Reference

| Shortcode | Module |
|-----------|--------|
| `[institutionkit_admission_form]` | Admissions form |
| `[institutionkit_attendance]` | Teacher attendance |
| `[institutionkit_gradebook]` | Teacher gradebook |
| `[ik_invoices]` | Staff invoice management |
| `[ik_certificate_verify]` | Public certificate verification |
| `[ik_student_grades]` | Student grade display |
```
