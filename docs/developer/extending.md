```markdown
# Extending InstitutionKit

This guide covers best practices for extending InstitutionKit — adding custom functionality, integrating with external systems, and modifying behavior without touching core plugin files.

---

## Extension Philosophy

InstitutionKit is built with WordPress extensibility in mind:

1. **WordPress-native patterns** — Uses WordPress hooks, filters, and APIs
2. **Modular architecture** — Traits and classes can be extended or replaced
3. **Singleton pattern** — Most classes use `::instance()` or `::get_instance()`
4. **Clear data access** — Database class provides CRUD methods for all tables
5. **Capability system** — 62 custom capabilities for granular access control

---

## Safe Extension Methods

### Method 1: WordPress Hooks

**Best for:** Adding custom behavior at specific points

```php
// Run code after a student is promoted
add_action('ik_student_promoted', function($student_id, $from_class, $to_class) {
    // Send notification email
    // Update external system
    // Log to custom table
}, 10, 3);
```

### Method 2: Filter Hooks

**Best for:** Modifying data or behavior

```php
// Modify the grade scale
add_filter('ik_grade_scale', function($scales) {
    // Add custom grading logic
    return $scales;
});
```

### Method 3: Class Extension

**Best for:** Adding new features to existing modules

```php
class My_Extended_Payroll extends InstitutionKit_Payroll_Expenses {
    public function __construct() {
        parent::__construct();
        // Add custom hooks
        add_action('ik_payroll_generated', [$this, 'export_to_accounting']);
    }
    
    public function export_to_accounting($campus_id, $month) {
        // Export payroll data to external accounting system
    }
}
```

### Method 4: Database Extension

**Best for:** Adding custom data storage

```php
class My_Custom_Database extends IK_Database {
    public function create_tables() {
        parent::create_tables();
        $this->create_my_custom_tables();
    }
    
    private function create_my_custom_tables() {
        global $wpdb;
        $sql = "CREATE TABLE {$wpdb->prefix}my_custom_table (
            id BIGINT AUTO_INCREMENT PRIMARY KEY,
            name VARCHAR(255) NOT NULL
        ) {$wpdb->get_charset_collate()};";
        
        require_once ABSPATH . 'wp-admin/includes/upgrade.php';
        dbDelta($sql);
    }
}
```

---

## Common Integration Patterns

### Integrating with External Systems

```php
// Hook into invoice generation for accounting export
add_action('ik_invoice_generated', function($invoice_id, $student_id) {
    global $wpdb;
    
    $invoice = $wpdb->get_row($wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}institutionkit_invoices WHERE invoice_id = %d",
        $invoice_id
    ));
    
    // Send to external accounting API
    $response = wp_remote_post('https://accounting.example.com/api/invoices', [
        'headers' => ['Authorization' => 'Bearer ' . API_KEY],
        'body'    => json_encode($invoice),
    ]);
    
    // Log the integration
    update_post_meta($invoice_id, '_exported_to_accounting', current_time('mysql'));
}, 10, 2);
```

### Adding Custom Report

```php
add_action('admin_menu', function() {
    add_submenu_page(
        'ik-dashboard',
        'Custom Report',
        '📊 Custom Report',
        'manage_options',
        'ik-custom-report',
        'render_custom_report'
    );
}, 20);

function render_custom_report() {
    // Use InstitutionKit database methods
    $db = new IK_Database();
    $campuses = $db->get_campuses();
    
    ?>
    <div class="wrap">
        <h1>Custom Report</h1>
        <table class="wp-list-table widefat striped">
            <thead>
                <tr>
                    <th>Campus</th>
                    <th>Students</th>
                    <th>Revenue</th>
                </tr>
            </thead>
            <tbody>
                <?php foreach ($campuses as $campus): 
                    $summary = $db->get_campus_financial_summary(
                        $campus['campus_id'],
                        date('Y-m-01'),
                        date('Y-m-t')
                    );
                ?>
                <tr>
                    <td><?php echo esc_html($campus['campus_name']); ?></td>
                    <td><?php echo $summary['metrics']['student_count']; ?></td>
                    <td><?php echo number_format($summary['revenue']['total_collections']); ?></td>
                </tr>
                <?php endforeach; ?>
            </tbody>
        </table>
    </div>
    <?php
}
```

### Adding Custom Student Field

```php
// Add meta box
add_action('add_meta_boxes', function() {
    add_meta_box(
        'custom_medical_info',
        'Medical Information',
        'render_medical_meta_box',
        'ik_student',
        'normal',
        'default'
    );
});

function render_medical_meta_box($post) {
    $blood_group = get_post_meta($post->ID, '_ik_blood_group', true);
    $allergies = get_post_meta($post->ID, '_ik_allergies', true);
    ?>
    <p>
        <label>Blood Group:</label>
        <input type="text" name="ik_blood_group" value="<?php echo esc_attr($blood_group); ?>">
    </p>
    <p>
        <label>Allergies:</label>
        <textarea name="ik_allergies"><?php echo esc_textarea($allergies); ?></textarea>
    </p>
    <?php
}

// Save meta
add_action('save_post_ik_student', function($post_id) {
    if (isset($_POST['ik_blood_group'])) {
        update_post_meta($post_id, '_ik_blood_group', sanitize_text_field($_POST['ik_blood_group']));
    }
    if (isset($_POST['ik_allergies'])) {
        update_post_meta($post_id, '_ik_allergies', sanitize_textarea_field($_POST['ik_allergies']));
    }
});
```

---

## Custom Workflow Automation

```php
// Auto-generate certificates on promotion
add_action('ik_student_promoted', function($student_id, $from_class, $to_class) {
    global $wpdb;
    
    // Generate promotion certificate
    $cert_number = 'PROMO-' . date('Y') . '-' . str_pad($student_id, 5, '0', STR_PAD_LEFT);
    
    $wpdb->insert(
        "{$wpdb->prefix}institutionkit_certificates",
        [
            'recipient_id'       => $student_id,
            'recipient_type'     => 'student',
            'certificate_type'   => 'achievement',
            'student_name'       => get_the_title($student_id),
            'achievement_details' => 'Promoted from ' . get_the_title($from_class) . ' to ' . get_the_title($to_class),
            'issue_date'         => current_time('Y-m-d'),
            'certificate_number' => $cert_number,
            'issued_by'          => get_current_user_id(),
        ]
    );
}, 10, 3);
```

---

## Custom Email Notifications

```php
// Send email when attendance drops below threshold
add_action('ik_attendance_saved', function($class_id, $date, $marked_by) {
    global $wpdb;
    
    $students = $wpdb->get_results($wpdb->prepare(
        "SELECT student_id FROM {$wpdb->prefix}institutionkit_attendance 
         WHERE class_id = %d AND attendance_date = %s AND status = 'absent'",
        $class_id, $date
    ));
    
    foreach ($students as $student) {
        // Get parent email
        $parent_email = get_post_meta($student->student_id, '_ik_email', true);
        
        if ($parent_email) {
            wp_mail(
                $parent_email,
                'Attendance Alert - ' . get_bloginfo('name'),
                'Your child was marked absent on ' . $date . '. Please contact the school.'
            );
        }
    }
}, 10, 3);
```

---

## Best Practices

1. **Never modify core plugin files** — Use hooks, filters, and inheritance
2. **Check for class existence** — Use `class_exists()` before extending
3. **Respect campus boundaries** — Use `IK_Campus_Manager` for data filtering
4. **Use proper nonces** — Always verify nonces in custom handlers
5. **Sanitize ALL input** — Follow WordPress sanitization standards
6. **Escape ALL output** — Use `esc_html()`, `esc_attr()`, `esc_url()`
7. **Capability-check everything** — Use `current_user_can()` or `verify_access()`
8. **Cache responsibly** — Use WordPress transients for expensive queries
9. **Log errors** — Use `error_log()` for debugging custom code
10. **Test on staging first** — Never deploy custom extensions directly to production
```
