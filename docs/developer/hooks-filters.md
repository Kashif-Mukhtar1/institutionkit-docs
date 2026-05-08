```markdown
# Hooks & Filters Reference

InstitutionKit provides action hooks and filters throughout the codebase for developers to extend functionality without modifying core plugin files.

---

## Action Hooks

### Exam Module

| Hook | Fires When | Parameters |
|------|-----------|------------|
| `ik_exam_type_deleted` | An exam type is permanently deleted | `$exam_type_id` (int) |

### Usage

```php
add_action('ik_exam_type_deleted', function($exam_type_id) {
    // Clean up related data
    // Log the deletion
    // Notify administrators
    error_log("Exam type #{$exam_type_id} was deleted.");
});
```

---

## Filter Hooks

### Dashboard Widgets

| Filter | Purpose | Default |
|--------|---------|---------|
| `ik_show_parent_stats` | Show/hide parent portal stats on dashboard | `true` |

### Usage

```php
// Hide parent stats for non-administrators
add_filter('ik_show_parent_stats', function($show) {
    if (!current_user_can('manage_options')) {
        return false;
    }
    return $show;
});
```

---

## Custom Hooks You Can Add

InstitutionKit's modular architecture supports adding your own hooks. Here are recommended locations:

### Student Lifecycle

```php
// After student creation
do_action('ik_student_created', $student_id, $student_data);

// After student promotion
do_action('ik_student_promoted', $student_id, $from_class_id, $to_class_id);

// After campus transfer
do_action('ik_student_transferred', $student_id, $from_campus_id, $to_campus_id);
```

### Fee & Invoice

```php
// After invoice generation
do_action('ik_invoice_generated', $invoice_id, $student_id);

// After payment recorded
do_action('ik_payment_recorded', $transaction_id, $invoice_id, $amount);

// After fee reminder sent
do_action('ik_fee_reminder_sent', $student_id, $invoice_id);
```

### Attendance

```php
// After attendance saved
do_action('ik_attendance_saved', $class_id, $attendance_date, $marked_by);

// After staff attendance saved
do_action('ik_staff_attendance_saved', $campus_id, $attendance_date);
```

### Payroll

```php
// After payroll generated
do_action('ik_payroll_generated', $campus_id, $payroll_month, $record_count);

// After payslip downloaded
do_action('ik_payslip_downloaded', $staff_id, $month);
```

---

## Extending Without Hooks

### Using Class Inheritance

Most InstitutionKit classes follow a singleton pattern. You can extend them:

```php
class My_Custom_Exam_Module extends IK_Exam_Module {
    public function render_exam_types() {
        // Add custom content before
        echo '<div class="my-custom-header">Custom Header</div>';
        
        // Call parent method
        parent::render_exam_types();
        
        // Add custom content after
        echo '<div class="my-custom-footer">Custom Footer</div>';
    }
}
```

### Using WordPress Core Hooks

InstitutionKit uses many WordPress core hooks you can leverage:

```php
// Modify student query
add_action('pre_get_posts', function($query) {
    if ($query->get('post_type') === 'ik_student') {
        // Add custom filtering
    }
});

// Modify admin menu
add_action('admin_menu', function() {
    // Add custom submenu items
}, 20); // After InstitutionKit (default priority 10)

// Enqueue custom assets on IK pages
add_action('admin_enqueue_scripts', function($hook) {
    if (strpos($hook, 'ik-') !== false) {
        wp_enqueue_style('my-custom-css', '...');
    }
});
```

---

## Database Query Modification

### Filter SQL WHERE Clauses

Many methods return SQL fragments you can modify:

```php
// Modify campus filtering
add_filter('ik_campus_where_clause', function($where, $table_alias) {
    // Add additional conditions
    return $where;
}, 10, 2);
```

### Extend Database Methods

The `IK_Database` class can be extended or replaced:

```php
class My_Custom_Database extends IK_Database {
    public function get_students($filters = []) {
        // Add custom filtering logic
        $filters['my_custom_filter'] = true;
        return parent::get_students($filters);
    }
}
```

---

## AJAX Endpoint Extension

### Adding Custom AJAX Handlers

```php
add_action('wp_ajax_my_custom_action', function() {
    check_ajax_referer('ik_payroll_nonce', 'nonce');
    
    // Your custom logic using InstitutionKit classes
    $db = new IK_Database();
    $result = $db->get_expenses(['campus_id' => 3]);
    
    wp_send_json_success($result);
});
```

### Modifying AJAX Responses

```php
add_action('wp_ajax_ik_get_dashboard_data', function() {
    // Intercept and modify dashboard data
}, 1); // Priority 1 — runs before InstitutionKit's handler
```

---

## Best Practices for Extending

1. **Use hooks when available** — Hooks are stable across updates
2. **Avoid modifying core files** — Use WordPress hooks and filters instead
3. **Check for class existence** — Use `class_exists()` before extending
4. **Respect capability checks** — Use `current_user_can()` for access control
5. **Use proper nonces** — Always verify nonces in custom AJAX handlers
6. **Sanitize all input** — Follow InstitutionKit's sanitization patterns
7. **Escape all output** — Use `esc_html()`, `esc_attr()`, `esc_url()`
8. **Campus-filter your queries** — Use `IK_Campus_Manager::get_campus_where_clause()`

---

## Example: Adding a Custom Dashboard Widget

```php
add_action('admin_head', function() {
    $page = $_GET['page'] ?? '';
    if ($page !== 'ik-dashboard') return;
    ?>
    <script>
    jQuery(document).ready(function($) {
        $.ajax({
            url: ajaxurl,
            type: 'POST',
            data: {
                action: 'my_custom_dashboard_data',
                nonce: '<?php echo wp_create_nonce("my_nonce"); ?>'
            },
            success: function(response) {
                var html = '<div class="ik-stat-card">' +
                    '<h3>My Widget</h3>' +
                    '<div class="ik-stat-number">' + response.data + '</div>' +
                    '</div>';
                $('.ik-stats-grid').append(html);
            }
        });
    });
    </script>
    <?php
});

add_action('wp_ajax_my_custom_dashboard_data', function() {
    check_ajax_referer('my_nonce', 'nonce');
    
    global $wpdb;
    $count = $wpdb->get_var("SELECT COUNT(*) FROM {$wpdb->posts} WHERE post_type = 'ik_student'");
    
    wp_send_json_success($count);
});
```
