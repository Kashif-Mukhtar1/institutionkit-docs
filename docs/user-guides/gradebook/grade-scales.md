```markdown
# Grade Scales

Grade Scales define how percentage scores are converted to letter grades and GPA points. InstitutionKit ships with a comprehensive default scale that can be customized per campus.

---

## Accessing Grade Scales

Grade scales are configured in the database via the `ik_grade_scales` table. Default scales are inserted on plugin activation.

To customize scales, use the Settings page or manage them programmatically.

---

## Default Grade Scale

InstitutionKit's default 11-point scale:

| Min % | Max % | Grade | GPA | Performance Level |
|-------|-------|-------|-----|-------------------|
| 90.00 | 100.00 | A+ | 4.00 | Outstanding |
| 85.00 | 89.99 | A | 4.00 | Excellent |
| 80.00 | 84.99 | A- | 3.70 | Very Good |
| 77.00 | 79.99 | B+ | 3.30 | Good Plus |
| 73.00 | 76.99 | B | 3.00 | Good |
| 70.00 | 72.99 | B- | 2.70 | Above Average |
| 67.00 | 69.99 | C+ | 2.30 | Average Plus |
| 63.00 | 66.99 | C | 2.00 | Average |
| 60.00 | 62.99 | C- | 1.70 | Below Average |
| 50.00 | 59.99 | D | 1.00 | Marginal |
| 0.00 | 49.99 | F | 0.00 | Fail |

---

## Frontend Gradebook Scale

The frontend gradebook uses a simplified scale for quick visual feedback:

| Percentage | Grade |
|------------|-------|
| 98%+ | A++ |
| 95%+ | A+ |
| 90%+ | A |
| 80%+ | B |
| 65%+ | C |
| 55%+ | D |
| 40%+ | D |
| Below 40% | F |

This is used for real-time grade display as marks are entered.

---

## Grade Scale Database

```sql
CREATE TABLE wp_ik_grade_scales (
    scale_id INT(11) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    min_percent DECIMAL(5,2) NOT NULL,
    max_percent DECIMAL(5,2) NOT NULL,
    letter_grade VARCHAR(5) NOT NULL,
    gpa_points DECIMAL(3,2),
    scale_type VARCHAR(10) DEFAULT 'default',
    campus_id BIGINT(20) UNSIGNED,
    KEY scale_type (scale_type),
    KEY campus_id (campus_id)
);
```

### Scale Types

| Type | Description |
|------|-------------|
| `default` | System-wide default scale (inserted on activation) |
| Campus-specific | Custom scale for a particular campus (uses `campus_id`) |

---

## How Grade Lookup Works

### Finding the Right Scale

1. Check if campus has a custom scale (query with `campus_id`)
2. If not, use the `default` scale
3. Scale is cached for 1 hour using WordPress object cache

```php
public function get_grade_scale() {
    $cache_key = 'ik_grade_scales';
    $scales = wp_cache_get($cache_key, 'institutionkit');
    
    if (false === $scales) {
        $scales = $wpdb->get_results($wpdb->prepare(
            "SELECT * FROM {$table} 
             WHERE scale_type = %s OR campus_id = %d 
             ORDER BY min_percent DESC",
            'default',
            $this->get_current_campus_id()
        ), ARRAY_A);
        
        wp_cache_set($cache_key, $scales, 'institutionkit', HOUR_IN_SECONDS);
    }
    
    return $scales ?: [];
}
```

### Grade Assignment Logic

The scale is sorted by `min_percent DESC`, and the first matching range is used:

```php
public function calculate_grade($marks_obtained, $max_marks) {
    if ($max_marks <= 0) return 'N/A';
    
    $percentage = ($marks_obtained / $max_marks) * 100;
    $grade_scale = $this->get_grade_scale();
    
    foreach ($grade_scale as $grade) {
        if ($percentage >= $grade['min_percent'] && $percentage <= $grade['max_percent']) {
            return $grade['letter_grade'];
        }
    }
    return 'F'; // Fallback
}
```

---

## GPA Calculation

GPA is calculated alongside letter grades:

```php
public function calculate_gpa($marks_obtained, $max_marks) {
    if ($max_marks <= 0) return 0.00;
    
    $percentage = ($marks_obtained / $max_marks) * 100;
    $grade_scale = $this->get_grade_scale();
    
    foreach ($grade_scale as $grade) {
        if ($percentage >= $grade['min_percent'] && $percentage <= $grade['max_percent']) {
            return (float) $grade['gpa_points'];
        }
    }
    return 0.00;
}
```

---

## Creating Custom Scales

### Add a Campus-Specific Scale

```php
global $wpdb;
$campus_id = 3;

$custom_scales = [
    ['min_percent' => 93.00, 'max_percent' => 100.00, 'letter_grade' => 'A+', 'gpa_points' => 4.00],
    ['min_percent' => 85.00, 'max_percent' => 92.99, 'letter_grade' => 'A', 'gpa_points' => 3.70],
    ['min_percent' => 75.00, 'max_percent' => 84.99, 'letter_grade' => 'B', 'gpa_points' => 3.00],
    ['min_percent' => 65.00, 'max_percent' => 74.99, 'letter_grade' => 'C', 'gpa_points' => 2.00],
    ['min_percent' => 50.00, 'max_percent' => 64.99, 'letter_grade' => 'D', 'gpa_points' => 1.00],
    ['min_percent' => 0.00,  'max_percent' => 49.99, 'letter_grade' => 'F', 'gpa_points' => 0.00],
];

foreach ($custom_scales as $scale) {
    $wpdb->insert(
        "{$wpdb->prefix}ik_grade_scales",
        [
            'min_percent'  => $scale['min_percent'],
            'max_percent'  => $scale['max_percent'],
            'letter_grade' => $scale['letter_grade'],
            'gpa_points'   => $scale['gpa_points'],
            'scale_type'   => 'campus',
            'campus_id'    => $campus_id,
        ],
        ['%f', '%f', '%s', '%f', '%s', '%d']
    );
}
```

### Clear Grade Scale Cache After Changes

```php
wp_cache_delete('ik_grade_scales', 'institutionkit');
```

---

## Grade Scale Examples

### Traditional A-F Scale

| Min % | Grade | GPA |
|-------|-------|-----|
| 90 | A | 4.0 |
| 80 | B | 3.0 |
| 70 | C | 2.0 |
| 60 | D | 1.0 |
| 0 | F | 0.0 |

### International Baccalaureate (IB) Style

| Min % | Grade | GPA |
|-------|-------|-----|
| 85 | 7 | 4.0 |
| 75 | 6 | 3.5 |
| 65 | 5 | 3.0 |
| 55 | 4 | 2.5 |
| 45 | 3 | 2.0 |
| 35 | 2 | 1.5 |
| 0 | 1 | 0.0 |

### UK-Style Scale

| Min % | Grade |
|-------|-------|
| 80 | A* |
| 70 | A |
| 60 | B |
| 50 | C |
| 40 | D |
| 0 | U |

---

## Programmatic Usage

### Get All Scales

```php
global $wpdb;
$scales = $wpdb->get_results(
    "SELECT * FROM {$wpdb->prefix}ik_grade_scales 
     WHERE scale_type = 'default' 
     ORDER BY min_percent DESC",
    ARRAY_A
);
```

### Get Scales for Campus

```php
global $wpdb;
$scales = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}ik_grade_scales 
     WHERE campus_id = %d OR scale_type = 'default'
     ORDER BY min_percent DESC",
    $campus_id
), ARRAY_A);
```

### Delete Campus Scales (Reset to Default)

```php
global $wpdb;
$wpdb->delete(
    "{$wpdb->prefix}ik_grade_scales",
    ['campus_id' => $campus_id]
);
wp_cache_delete('ik_grade_scales', 'institutionkit');
```
