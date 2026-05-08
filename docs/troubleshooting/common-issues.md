```markdown
# Troubleshooting & Common Issues

Solutions for common issues encountered when installing, configuring, or using InstitutionKit.

---

## Installation Issues

### Tables Not Created on Activation

**Symptoms:** After activating InstitutionKit, some features show database errors or empty data.

**Causes:**
- Database user lacks `CREATE TABLE` privileges
- WordPress `dbDelta()` function failed silently
- Table prefix mismatch

**Solutions:**

1. **Check database privileges:**
    ```sql
    SHOW GRANTS FOR CURRENT_USER();
    ```
    Ensure `CREATE`, `ALTER`, and `INDEX` privileges are granted.

2. **Manual table creation:**
    - Go to **InstitutionKit → Settings → System Status**
    - Click **Recreate Tables**
    - If this fails, check the error log

3. **Verify table prefix:**
    ```php
    global $wpdb;
    echo $wpdb->prefix; // Should match your wp-config.php setting
    ```

---

### White Screen / Fatal Error After Activation

**Symptoms:** Blank white screen or "There has been a critical error" message.

**Solutions:**

1. **Enable debug mode** in `wp-config.php`:
    ```php
    define('WP_DEBUG', true);
    define('WP_DEBUG_LOG', true);
    define('WP_DEBUG_DISPLAY', false);
    ```

2. **Check the debug log** at `/wp-content/debug.log`

3. **Common causes:**
    - PHP version below 7.4 — Upgrade to PHP 8.0+
    - Missing PHP extensions — Ensure `mysqli`, `curl`, `dom`, `mbstring`, `gd` are enabled
    - Memory exhaustion — Increase `WP_MEMORY_LIMIT` to 256M

---

### Memory Exhausted Error

**Symptoms:** "Allowed memory size of X bytes exhausted" error.

**Solution:** Add to `wp-config.php`:
```php
define('WP_MEMORY_LIMIT', '256M');
define('WP_MAX_MEMORY_LIMIT', '512M');
```

---

## License Issues

### License Banner Won't Disappear

**Symptoms:** Yellow "License Required" banner persists after activation.

**Solutions:**

1. **Verify activation:**
    - Go to **InstitutionKit → Settings → License**
    - Confirm status shows "Active"

2. **Clear cache:**
    ```sql
    DELETE FROM wp_options WHERE option_name LIKE '%ik_license%';
    ```
    Then reactivate the license.

3. **Check connectivity:**
    - Your server must be able to make outbound HTTP requests to `institutionkit.com`
    - Firewall or security plugins may block license verification

---

### "Could Not Connect to License Server"

**Causes:**
- Server firewall blocking outbound connections
- cURL PHP extension not installed
- DNS resolution failure

**Solutions:**

1. Install/enable PHP cURL extension
2. Ask your host to allow outbound connections on port 443
3. Test connectivity:
    ```php
    $response = wp_remote_get('https://institutionkit.com');
    if (is_wp_error($response)) {
        echo $response->get_error_message();
    }
    ```

---

## Campus & Access Issues

### Campus Admin Sees No Data

**Symptoms:** Campus Admin logs in but sees empty student lists, no attendance, etc.

**Causes:**
- User assigned to wrong campus
- User has both `administrator` and `campus_admin` roles
- Data not assigned to the campus

**Solutions:**

1. **Verify role assignment:**
    - The user must have `campus_admin` role **without** `administrator` role
    - Having both roles makes them a Super Admin

2. **Check campus assignment:**
    ```sql
    SELECT * FROM wp_institutionkit_campus_users WHERE user_id = [USER_ID];
    ```

3. **Verify data has campus_id:**
    ```sql
    SELECT campus_id, COUNT(*) FROM wp_institutionkit_attendance GROUP BY campus_id;
    ```

---

### Campus Switcher Not Appearing

**Symptoms:** Super Admin doesn't see the campus switcher.

**Cause:** The user isn't recognized as a Super Admin.

**Check:**
```php
if (current_user_can('manage_options') && !IK_Campus_Manager::is_campus_admin($user_id)) {
    // Should see switcher
}
```

---

## Payroll Issues

### Payroll Generation Returns Zero Records

**Symptoms:** Clicking "Generate Payroll" shows "No payroll records generated."

**Causes:**
- No active staff in the selected campus
- Staff missing salary/contract information
- Payroll already generated for this month
- Wrong month format

**Solutions:**

1. **Verify active staff:**
    ```sql
    SELECT COUNT(*) FROM wp_institutionkit_staff 
    WHERE primary_campus_id = 3 AND employment_status = 'active';
    ```

2. **Check contract types:**
    ```sql
    SELECT full_name, contract_type, base_salary 
    FROM wp_institutionkit_staff 
    WHERE primary_campus_id = 3 AND employment_status = 'active';
    ```

3. **Check existing payroll:**
    ```sql
    SELECT COUNT(*) FROM wp_institutionkit_payroll 
    WHERE campus_id = 3 AND payroll_month = '2026-06-01';
    ```

---

### Debug Info Displayed on Payroll Page

When no records are found, the payroll page shows debug information:

```
Debug Info:
Active staff for campus: 12
Existing payroll records for this month: 0
Campus ID: 3
Month: 2026-06-01
```

This helps diagnose the issue. If records exist but don't display, the table name or month format may be wrong.

---

## Gradebook Issues

### Duplicate Grade Entries

**Symptoms:** Saving grades creates duplicate rows.

**Cause:** The unique constraint `(student_id, subject_id, period_id)` should prevent duplicates. If duplicates exist, the constraint may be missing.

**Solution:**
```sql
-- Check constraint exists
SHOW INDEX FROM wp_ik_grades_v2 WHERE Key_name = 'unique_grade';

-- If missing, clean duplicates and add constraint
ALTER TABLE wp_ik_grades_v2 
ADD UNIQUE KEY unique_grade (student_id, subject_id, period_id);
```

---

### Grade Scale Not Applying

**Symptoms:** Grades show incorrect letters or "F" for all students.

**Causes:**
- No grade scale defined
- Scale not matching the campus
- Scale cache not cleared

**Solutions:**

1. **Check scales exist:**
    ```sql
    SELECT * FROM wp_ik_grade_scales WHERE scale_type = 'default';
    ```

2. **Clear scale cache:**
    ```php
    wp_cache_delete('ik_grade_scales', 'institutionkit');
    ```

---

## Attendance Issues

### Cannot Mark Attendance from Frontend

**Symptoms:** Teacher sees "You are not assigned to any campus" or empty student list.

**Solutions:**

1. **Check staff-campus assignment:**
    ```sql
    SELECT full_name, primary_campus_id FROM wp_institutionkit_staff WHERE user_id = [USER_ID];
    ```

2. **Verify campus exists and is active:**
    ```sql
    SELECT * FROM wp_institutionkit_campuses WHERE campus_id = [CAMPUS_ID] AND is_active = 1;
    ```

3. **Check sections requirement:**
    - If sections exist in the system, a section must be selected before students appear

---

### Attendance Read-Only on Frontend

**Symptoms:** Teacher cannot mark attendance; form shows submission info.

This is **by design** — attendance can only be modified once from the frontend. Administrators can modify it from the backend admin panel.

---

## AJAX Issues

### AJAX Returns "0" or Blank

**Common causes:**

1. **Nonce mismatch** — Ensure the correct nonce is being sent
2. **User not logged in** — AJAX handlers require authentication
3. **Action not registered** — Check the action name matches exactly

**Debug:**
```javascript
$.ajax({
    url: ajaxurl,
    type: 'POST',
    data: { action: 'ik_get_dashboard_data', nonce: ik_ajax.nonce },
    success: function(response) {
        console.log(response); // Check browser console
    },
    error: function(xhr, status, error) {
        console.log(status, error); // Check for errors
    }
});
```

---

## PDF Generation Issues

### Payslip/Invoice PDF Not Generating

**Causes:**
- Dompdf library not found
- PHP `dom` extension missing
- Memory exhausted during generation

**Solutions:**

1. **Check Dompdf exists:**
    ```
    /wp-content/plugins/institutionkit/includes/lib/dompdf/autoload.inc.php
    ```

2. **Install PHP dom extension:**
    ```bash
    sudo apt-get install php-dom  # Ubuntu/Debian
    ```

3. **Increase memory:**
    ```php
    define('WP_MEMORY_LIMIT', '512M');
    ```

---

## Quick Diagnostic Queries

Run these in your database to diagnose common issues:

```sql
-- Check all InstitutionKit tables exist
SELECT TABLE_NAME FROM information_schema.TABLES 
WHERE TABLE_SCHEMA = DATABASE() 
  AND (TABLE_NAME LIKE 'wp_institutionkit%' OR TABLE_NAME LIKE 'wp_ik_%')
ORDER BY TABLE_NAME;

-- Check campus data distribution
SELECT campus_id, COUNT(*) as records 
FROM wp_institutionkit_attendance 
GROUP BY campus_id;

-- Check staff without campus
SELECT staff_id, full_name, primary_campus_id 
FROM wp_institutionkit_staff 
WHERE primary_campus_id IS NULL OR primary_campus_id = 0;

-- Check orphaned parent links
SELECT pc.* FROM wp_institutionkit_parent_child pc
LEFT JOIN wp_users u ON pc.parent_id = u.ID
WHERE u.ID IS NULL;
```

---

## Getting Help

If these solutions don't resolve your issue:

- Email: support@institutionkit.com
- Phone: +92 300 455 1325
- Web: [institutionkit.com/support](https://institutionkit.com)

When contacting support, include:

1. WordPress version
2. PHP version
3. InstitutionKit version
4. Screenshot of the issue
5. Steps to reproduce
6. Any relevant error log entries
```
