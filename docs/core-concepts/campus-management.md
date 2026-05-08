
```markdown
# Campus Management Deep-Dive

This document covers the technical architecture of the multi-campus system — how campus context is determined, how data is partitioned, and how the campus switcher operates at the code level.

---

## Campus Context Lifecycle

Every page load follows this sequence:

```
1. WordPress init (priority 5)
   └── IK_Campus_Manager::determine_current_campus()
       
2. Check user identity
   ├── Is user logged in?
   ├── Is user a Campus Admin? (has campus_admin, NOT administrator)
   └── Is user a Super Admin? (has manage_options)
       
3. Determine campus_id
   ├── Super Admin + ?campus_id=X in URL → Set to X (0 = All)
   ├── Super Admin + no URL param → Read from transient
   ├── Campus Admin + ?campus_id=X → Validate X is their campus
   └── Campus Admin + no URL → First assigned campus
       
4. Campus ID cached in class property
   └── IK_Campus_Manager::$current_campus_id
       
5. All subsequent queries use this context
```

---

## Core Class: `IK_Campus_Manager`

### Key Properties

```php
class IK_Campus_Manager {
    private static $current_campus_id = null;  // The active campus context
    private static $user_campuses = [];        // Cached per-user campus lists
}
```

### Key Methods

| Method | Returns | Purpose |
|--------|---------|---------|
| `get_current_campus_id()` | `int` | 0 = All Campuses, 1-99 = specific campus |
| `is_campus_admin($user_id)` | `bool` | Has `campus_admin` role, NOT `administrator` |
| `is_super_admin($user_id)` | `bool` | Has `manage_options`, NOT campus_admin only |
| `get_user_campuses($user_id)` | `array` | Campus IDs accessible to this user |
| `can_access_campus($user_id, $campus_id)` | `bool` | Access check for a specific campus |
| `get_campus_where_clause($alias)` | `string` | SQL WHERE fragment for filtering |
| `get_campus_sql_filter()` | `string` | Post-meta SQL filter |
| `get_campus_join_clause()` | `string` | Post-meta JOIN clause |

---

## SQL Filtering Methods

### For Custom Tables (Direct Column)

```php
// Returns: " AND i.campus_id = 3" or "" (empty for All Campuses)
$where = IK_Campus_Manager::get_campus_where_clause('i');

// Usage in queries:
$sql = "SELECT * FROM wp_institutionkit_invoices i WHERE 1=1 {$where}";
```

When `campus_id = 0` (All Campuses), the method returns an empty string — no filter applied.

When `campus_id = 3`, it returns ` AND i.campus_id = 3`.

### For WordPress Posts (Post Meta)

```php
$filter = IK_Campus_Manager::get_campus_sql_filter();
// Returns: " AND (pm.meta_key = '_ik_campus_id' AND pm.meta_value = 3)"

$join = IK_Campus_Manager::get_campus_join_clause();
// Returns: "LEFT JOIN wp_postmeta pm ON p.ID = pm.post_id"
```

Used together:

```php
$sql = "SELECT p.* FROM wp_posts p {$join} WHERE p.post_type = 'ik_student' {$filter}";
```

---

## Campus Switcher Implementation

### Admin Bar Node

```php
public static function add_campus_switcher_to_admin_bar($wp_admin_bar) {
    if (!self::is_super_admin()) return;  // Only for Super Admins
    
    $current = self::get_current_campus_id();
    $campuses = self::get_all_campuses();
    
    // Parent node
    $wp_admin_bar->add_node([
        'id'    => 'ik-campus-switcher',
        'title' => '🏛️ ' . ($current === 0 ? 'All Campuses' : self::get_campus_name($current)),
    ]);
    
    // All Campuses option
    $wp_admin_bar->add_node([
        'id'     => 'ik-campus-all',
        'parent' => 'ik-campus-switcher',
        'title'  => 'All Campuses',
        'href'   => admin_url('admin.php?page=' . $_GET['page'] . '&campus_id=0'),
    ]);
    
    // Individual campus options
    foreach ($campuses as $campus) {
        $wp_admin_bar->add_node([
            'id'     => 'ik-campus-' . $campus->campus_id,
            'parent' => 'ik-campus-switcher',
            'title'  => $campus->campus_name,
            'href'   => admin_url('admin.php?page=' . $_GET['page'] . '&campus_id=' . $campus->campus_id),
        ]);
    }
}
```

### Dashboard Dropdown

```php
public static function render_campus_switcher() {
    if (!self::is_super_admin()) return;
    
    $campuses = self::get_all_campuses();
    $current = self::get_current_campus_id();
    ?>
    <form method="get" action="<?php echo admin_url('admin.php'); ?>">
        <input type="hidden" name="page" value="<?php echo $_GET['page']; ?>">
        <select name="campus_id" onchange="this.form.submit()">
            <option value="0" <?php selected($current, 0); ?>>All Campuses</option>
            <?php foreach ($campuses as $campus): ?>
                <option value="<?php echo $campus->campus_id; ?>" <?php selected($current, $campus->campus_id); ?>>
                    <?php echo $campus->campus_name; ?>
                </option>
            <?php endforeach; ?>
        </select>
    </form>
    <?php
}
```

---

## Transient-Based Persistence

The selected campus persists across page loads using WordPress transients:

```php
// Save campus selection (1-hour expiry)
set_transient('ik_current_campus_' . $user_id, $campus_id, HOUR_IN_SECONDS);

// Retrieve saved campus
$saved_campus = get_transient('ik_current_campus_' . $user_id);
```

For Super Admins:

- If `?campus_id=X` is in the URL, it's saved to the transient
- If no URL parameter, the transient value is used
- If no transient exists, defaults to `0` (All Campuses)

For Campus Admins:

- Transients are ignored
- Their campus is always their assigned campus from `institutionkit_campus_users`

---

## Default Campus Fallback

When the campuses table doesn't exist yet (fresh install before activation completes):

```php
private static function get_default_campus_list() {
    $default = new stdClass();
    $default->campus_id = 1;
    $default->campus_name = 'Main Campus';
    $default->campus_code = 'MAIN';
    return [$default];
}
```

---

## Campus Tables Relationship

```
institutionkit_campuses (1)
    │
    ├── institutionkit_campus_users (N)
    │   └── Links WordPress users to campuses
    │
    ├── institutionkit_campus_transfers (N)
    │   └── Audit trail for student/staff movements
    │
    └── All other tables (campus_id FK)
        ├── institutionkit_attendance
        ├── institutionkit_invoices
        ├── institutionkit_payroll
        ├── institutionkit_expenses
        └── ... (30+ tables)
```
