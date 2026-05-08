```markdown
# Shortcodes Reference

InstitutionKit provides shortcodes for embedding frontend modules, student grade displays, and certificate verification on any WordPress page.

---

## Available Shortcodes

| Shortcode | Module | Authentication Required |
|-----------|--------|:---:|
| `[institutionkit_admission_form]` | Frontend Admissions | No |
| `[institutionkit_attendance]` | Frontend Attendance | Yes (Teacher/Campus Admin) |
| `[institutionkit_gradebook]` | Frontend Gradebook | Yes (Teacher/Campus Admin) |
| `[ik_invoices]` | Frontend Invoices | Yes (Staff with campus) |
| `[ik_certificate_verify]` | Certificate Verification | No |
| `[ik_student_grades]` | Student Grade Display | Yes (Student/Parent) |

---

## Shortcode Details

### `[institutionkit_admission_form]`

Renders the public online admission form.

**Usage:**

```
[institutionkit_admission_form]
```

**Requirements:**
- No login required
- Page must be publicly accessible

**What it does:**
- Displays a complete admission form
- Validates all fields server-side
- Creates `ik_student` post on submission
- Redirects with success message

---

### `[institutionkit_attendance]`

Renders the teacher attendance marking interface.

**Usage:**

```
[institutionkit_attendance]
```

**Requirements:**
- User must be logged in
- User must be a teacher, campus admin, or administrator
- Teacher must be assigned to a campus

**What it does:**
- Shows campus/class/section/date filters
- Displays student list with status radio buttons
- Saves attendance to `institutionkit_attendance`
- Read-only mode for previously saved dates

---

### `[institutionkit_gradebook]`

Renders the teacher gradebook interface.

**Usage:**

```
[institutionkit_gradebook]
```

**Requirements:**
- User must be logged in
- User must be a teacher, campus admin, or administrator
- Teacher must be assigned to a campus

**What it does:**
- Period/class/section/subject filter form
- Grade grid with real-time calculation
- Bulk fill and max marks operations
- Keyboard navigation (Enter to next student)
- AJAX save to `ik_grades_v2`

---

### `[ik_invoices]`

Renders the staff invoice management interface.

**Usage:**

```
[ik_invoices]
```

**Requirements:**
- User must be logged in
- User must have campus access or be an administrator

**What it does:**
- Invoice list with status tabs and filters
- Single invoice detail view
- Payment recording via AJAX
- PDF invoice download
- Print-optimized invoice view

---

### `[ik_certificate_verify]`

Renders a public certificate verification form.

**Usage:**

```
[ik_certificate_verify]
```

**Requirements:**
- No login required

**What it does:**
- Shows certificate number input field
- On submit, queries `institutionkit_certificates`
- Displays certificate details if found
- Shows "Certificate not found" if invalid

---

### `[ik_student_grades]`

Displays a student's grades (for student/parent viewing).

**Usage:**

```
[ik_student_grades]
```

**Requirements:**
- User must be logged in
- User must be a student or parent
- Linked to appropriate student records

**What it does:**
- Shows grades for the current student
- Organized by grading period
- Displays subject, marks, grade, and remarks

---

## Using Shortcodes in Templates

### In PHP Templates

```php
// Display the admission form
echo do_shortcode('[institutionkit_admission_form]');

// Display with specific parameters (if supported)
echo do_shortcode('[institutionkit_gradebook class_id="5"]');
```

### In Page Builders

Most page builders support shortcodes:

- **Elementor**: Use the "Shortcode" widget
- **Gutenberg**: Use the "Shortcode" block
- **Classic Editor**: Paste directly into content
- **Thrive Architect**: Use the "WordPress Content" element

---

## Styling Shortcode Output

All shortcode output uses `.ik-` prefixed CSS classes for scoping:

```css
/* Customize the admission form */
.ik-admission-form {
    max-width: 800px;
    margin: 0 auto;
}

.ik-form-group {
    margin-bottom: 20px;
}

.ik-form-error {
    color: #dc3545;
    font-size: 14px;
}

.ik-form-success {
    background: #d4edda;
    color: #155724;
    padding: 15px;
    border-radius: 8px;
}
```

---

## Creating Custom Shortcodes

### Extend with Your Own

```php
add_shortcode('my_custom_ik_feature', function($atts) {
    // Parse attributes
    $atts = shortcode_atts([
        'campus_id' => 0,
        'class_id'  => 0,
    ], $atts);
    
    // Check authentication
    if (!is_user_logged_in()) {
        return '<p>Please log in to view this content.</p>';
    }
    
    // Use InstitutionKit classes
    if (!class_exists('IK_Database')) {
        return '<p>InstitutionKit is not active.</p>';
    }
    
    $db = new IK_Database();
    
    // Your custom logic
    $students = get_posts([
        'post_type'      => 'ik_student',
        'posts_per_page' => -1,
        'meta_key'       => '_ik_student_class_id',
        'meta_value'     => $atts['class_id'],
    ]);
    
    // Render output
    ob_start();
    ?>
    <div class="my-custom-ik-widget">
        <h3>Students in Class <?php echo get_the_title($atts['class_id']); ?></h3>
        <ul>
            <?php foreach ($students as $student): ?>
                <li><?php echo esc_html($student->post_title); ?></li>
            <?php endforeach; ?>
        </ul>
    </div>
    <?php
    return ob_get_clean();
});
```

### Register with AJAX

```php
add_action('wp_ajax_my_custom_ik_action', function() {
    // Verify nonce
    check_ajax_referer('my_custom_nonce', 'nonce');
    
    // Check capabilities
    if (!current_user_can('manage_options')) {
        wp_send_json_error(['message' => 'Permission denied.']);
    }
    
    // Use InstitutionKit database
    global $wpdb;
    $data = $wpdb->get_results("SELECT * FROM {$wpdb->prefix}institutionkit_invoices LIMIT 10");
    
    wp_send_json_success($data);
});

// Localize script with nonce
wp_localize_script('my-script', 'myAjax', [
    'ajax_url' => admin_url('admin-ajax.php'),
    'nonce'    => wp_create_nonce('my_custom_nonce'),
]);
```

---

## Shortcode Placement Best Practices

1. **Dedicated pages** — Create separate pages for each shortcode rather than combining them
2. **Full-width templates** — Use full-width page templates to avoid sidebar conflicts
3. **Restrict access** — Use WordPress visibility settings or membership plugins to restrict shortcode pages
4. **Test theme compatibility** — Some themes may override form styles; test on your theme
5. **Cache exclusion** — Exclude shortcode pages from caching plugins to ensure fresh data
```
