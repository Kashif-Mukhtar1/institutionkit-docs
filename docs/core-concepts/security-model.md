File name: `docs/core-concepts/security-model.md`

```markdown
# Security Model

InstitutionKit implements a multi-layered security architecture protecting student data, financial records, and system configuration. This document covers all security mechanisms — from capability enforcement to SQL injection prevention.

---

## Security Layers

```
┌─────────────────────────────────────────┐
│         WordPress Authentication         │ ← Is the user logged in?
├─────────────────────────────────────────┤
│         Role Verification                │ ← Does the role have access?
├─────────────────────────────────────────┤
│         Capability Check                 │ ← Does the user have this specific cap?
├─────────────────────────────────────────┤
│         Campus Boundary                  │ ← Is the data within the user's campus?
├─────────────────────────────────────────┤
│         Nonce Verification               │ ← Is the request intentional?
├─────────────────────────────────────────┤
│         Data Sanitization                │ ← Is the input safe?
├─────────────────────────────────────────┤
│         SQL Escaping                     │ ← Is the query safe?
└─────────────────────────────────────────┘
```

---

## Layer 1: WordPress Authentication

Every admin page and AJAX endpoint requires a logged-in WordPress user. This is enforced by WordPress core — InstitutionKit inherits this protection without additional code.

For frontend modules, explicit login checks are implemented:

```php
// Parent Portal
if (!is_user_logged_in()) {
    // Render login prompt, then exit
    exit;
}

// Frontend Gradebook
if (!is_user_logged_in()) {
    return $this->render_login_prompt();
}
```

---

## Layer 2: Role Verification

After authentication, role checks determine broad access:

```php
// Check if user has a specific role
if (!in_array('parent', $current_user->roles)) {
    wp_die('Only parents can access the parent portal.');
}

// Check combined role + capability
if (!in_array('teacher', $user->roles) && 
    !in_array('campus_admin', $user->roles) && 
    !current_user_can('manage_options')) {
    return 'Permission denied.';
}
```

---

## Layer 3: Capability Enforcement

### Page-Level Checks

Every admin page verifies capabilities before rendering. The `IK_Admin_Menu` class provides a unified verification system:

```php
// Method used by all page renderers
private function verify_access($caps) {
    // Administrators always pass
    if (current_user_can('manage_options')) {
        return;
    }
    
    // Check single capability
    if (is_string($caps) && current_user_can($caps)) {
        return;
    }
    
    // Check array of capabilities (any one is sufficient)
    if (is_array($caps)) {
        foreach ($caps as $cap) {
            if (current_user_can($cap)) {
                return;
            }
        }
    }
    
    wp_die('You do not have sufficient permissions to access this page.');
}
```

**Usage examples:**

```php
// Strict: single capability required
$this->verify_access('ik_manage_payroll');

// Flexible: any of these capabilities
$this->verify_access(['ik_manage_expenses', 'ik_view_expense_reports']);

// Admin override: always passes for manage_options users
$this->verify_access('ik_manage_settings');
```

### Menu-Level Visibility

Menu items are registered with capability requirements — users without the required capability simply don't see the menu:

```php
add_submenu_page(
    'ik-dashboard',
    'Payroll & Expenses',
    '💵 Payroll & Expenses',
    ['ik_manage_payroll', 'ik_manage_expenses', 'ik_view_payroll_reports'],
    'ik-payroll-expenses',
    [$this, 'render_payroll_expenses_dashboard']
);
```

### AJAX Endpoint Security

All AJAX handlers verify nonces (which implicitly verifies the user has access to the page that generated the nonce):

```php
public function ik_generate_payroll() {
    check_ajax_referer('ik_payroll_nonce', 'nonce');
    // Nonce is only available to users who can access the payroll page
    // No additional capability check needed
}
```

---

## Layer 4: Campus Boundary Enforcement

### For Campus Admins

Campus Admins are **SQL-level restricted** — they cannot query data outside their campus:

```php
// This runs on every admin page load
IK_Campus_Manager::determine_current_campus();

// All queries filter by campus_id automatically
$where = IK_Campus_Manager::get_campus_where_clause('i');
// Campus Admin: " AND i.campus_id = 3"
// Super Admin (All Campuses): ""
```

### URL Parameter Protection

Campus Admins cannot override their campus via URL:

```php
if (isset($_GET['campus_id'])) {
    $requested_campus = absint($_GET['campus_id']);
    if (in_array($requested_campus, $user_campuses)) {
        self::$current_campus_id = $requested_campus;
    } else {
        // Ignore the request — use their assigned campus
        self::$current_campus_id = $user_campuses[0];
    }
}
```

### Empty Results for Unauthorized Access

When a Campus Admin attempts to access data outside their campus, the system returns **empty results** rather than permission errors:

```php
if (!IK_Campus_Manager::can_access_campus($user_id, $campus_id)) {
    return []; // Empty — reveals nothing about data existence
}
```

!!! info "Security Design Decision"
    A "Permission Denied" error confirms to a malicious actor that the resource exists. Empty results reveal nothing.

---

## Layer 5: Nonce Verification

Every form submission and destructive action is protected by WordPress nonces:

### Admin Forms

```php
// Nonce field in form
wp_nonce_field('ik_add_staff');

// Verification on submission
check_admin_referer('ik_add_staff');
```

### AJAX Requests

```php
// Nonce localized to JavaScript
wp_localize_script('ik-payroll-js', 'ik_ajax', [
    'nonce' => wp_create_nonce('ik_payroll_nonce'),
]);

// Verification in handler
check_ajax_referer('ik_payroll_nonce', 'nonce');
```

### Frontend Forms

```php
// Nonce field in parent portal
wp_nonce_field('ik_parent_meeting_nonce');

// Verification
if (!wp_verify_nonce($_POST['nonce'], 'ik_parent_meeting_nonce')) {
    wp_send_json_error(['message' => 'Security check failed.']);
}
```

### Nonce Lifetime

| Context | Nonce Lifetime |
|---------|---------------|
| Admin pages | 24 hours (WordPress default) |
| Frontend modules | 12 hours |
| AJAX actions | Session-based |

---

## Layer 6: Data Sanitization

### Input Sanitization

All user input is sanitized before processing:

```php
// Text fields
$name = sanitize_text_field($_POST['full_name']);

// Email
$email = sanitize_email($_POST['email']);

// Textarea
$description = sanitize_textarea_field($_POST['description']);

// Integers
$campus_id = absint($_POST['campus_id']);

// Floats
$amount = floatval($_POST['amount']);

// URLs
$url = esc_url_raw($_POST['redirect_url']);

// Keys (for arrays)
$key = sanitize_key($key);

// Recursive sanitization for nested arrays
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

### Output Escaping

All output is escaped for context:

```php
// HTML context
echo esc_html($student_name);

// Attribute context
echo esc_attr($roll_number);

// URL context
echo esc_url($redirect_url);

// JavaScript context
echo esc_js($dynamic_value);

// Textarea context
echo esc_textarea($notes);
```

---

## Layer 7: SQL Escaping

### Prepared Statements

All database queries use WordPress's `$wpdb->prepare()`:

```php
// Simple prepare
$result = $wpdb->get_var($wpdb->prepare(
    "SELECT COUNT(*) FROM {$table} WHERE campus_id = %d AND status = %s",
    $campus_id,
    $status
));

// Prepare with array expansion
$placeholders = implode(',', array_fill(0, count($ids), '%d'));
$sql = $wpdb->prepare(
    "SELECT * FROM {$table} WHERE student_id IN ({$placeholders})",
    ...$ids
);
```

### LIKE Queries

Search queries use `$wpdb->esc_like()` before preparing:

```php
$search_term = '%' . $wpdb->esc_like(sanitize_text_field($_GET['search'])) . '%';
$results = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$table} WHERE name LIKE %s",
    $search_term
));
```

### No Direct Query Concatenation

The codebase follows a strict rule: **never concatenate user input into SQL strings**. All variables in SQL queries pass through `$wpdb->prepare()`.

---

## Shared Security Trait

InstitutionKit includes a shared security trait used across modules:

```php
trait IK_Security {
    /**
     * Verify nonce and capability in one call.
     */
    protected function verify_request($nonce_action, $capability = 'manage_options') {
        check_ajax_referer($nonce_action, 'nonce');
        if (!current_user_can($capability)) {
            wp_send_json_error(['message' => 'Permission denied.']);
        }
    }
    
    /**
     * Sanitize and validate required fields.
     */
    protected function validate_required($data, $required_fields) {
        $errors = [];
        foreach ($required_fields as $field => $label) {
            if (empty($data[$field])) {
                $errors[] = "$label is required.";
            }
        }
        return $errors;
    }
}
```

---

## Frontend Module Security

### Admissions Form

```php
// Nonce verification
if (!wp_verify_nonce($_POST['ik_admissions_nonce'], 'ik_submit_admission')) {
    return;
}

// Field validation
if (!preg_match('/^\d{13}$/', $form_data['id_card_number'])) {
    $this->form_errors[] = 'ID card number must be a 13-digit number.';
}
```

### Attendance Form

```php
// Role check
if (!current_user_can('mark_attendance')) {
    wp_die('Permission denied.');
}

// Campus boundary check
if (!$this->is_class_allowed_for_user($class_id, $campus_id, $is_admin, $teacher_campus_ids)) {
    wp_die('Permission denied.');
}
```

### Gradebook

```php
// Combined role + capability check
if (!in_array('teacher', $user->roles) && 
    !in_array('campus_admin', $user->roles) && 
    !current_user_can('manage_options')) {
    return 'You do not have permission to access the gradebook.';
}
```

### Invoices (Frontend)

```php
// Campus restriction
if (!current_user_can('manage_options')) {
    $student_campus = get_post_meta($invoice->student_id, '_ik_campus_id', true);
    if ($student_campus != $this->get_current_user_campus_id()) {
        wp_send_json_error(['message' => 'Access Denied.']);
    }
}
```

### Parent Portal

```php
// Role-specific access
if (!in_array('parent', $current_user->roles)) {
    wp_die('Only parents can access the parent portal.');
}

// Data access limited to linked children
$student_ids = $wpdb->get_col($wpdb->prepare(
    "SELECT student_id FROM {$parent_child_table} WHERE parent_id = %d",
    $current_user->ID
));
```

---

## License Validation

The license check uses a direct database read to bypass object cache:

```php
private function get_license_status() {
    global $wpdb;
    $status = $wpdb->get_var(
        $wpdb->prepare(
            "SELECT option_value FROM {$wpdb->options} WHERE option_name = %s",
            'ik_license_status'
        )
    );
    return trim((string) $status) === 'active';
}
```

!!! warning "Why Direct DB Read?"
    WordPress object cache can return stale values for license status. A direct database query ensures the check is always accurate — critical for license enforcement.

---

## Security Best Practices for Extending InstitutionKit

### Do's

- Always use `IK_Campus_Manager::get_campus_where_clause()` for campus-scoped queries
- Verify nonces before processing any form submission
- Use `$wpdb->prepare()` for all database queries
- Escape output with the appropriate `esc_*` function
- Use `current_user_can()` before granting access to features

### Don'ts

- Never concatenate `$_POST`, `$_GET`, or `$_REQUEST` values into SQL
- Never trust `campus_id` from URL parameters without validation
- Never bypass `verify_access()` in admin page renderers
- Never store plain-text passwords or sensitive data in post meta
- Never disable nonce verification for AJAX handlers

---

## Vulnerability Reporting

If you discover a security vulnerability in InstitutionKit, please report it privately:

- Email: support@institutionkit.com
- Response time: Within 48 hours
- Disclosure policy: Coordinated disclosure after patch release
```
