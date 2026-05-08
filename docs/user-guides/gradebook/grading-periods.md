```markdown
# Grading Periods

Grading Periods define the time frames for assessing student performance. They are the backbone of the multi-period gradebook system, allowing grades to be organized by week, month, term, or custom examination period.

---

## Accessing Grading Periods

Navigate to **InstitutionKit → Grading Periods** or **Student Management → All Grading Periods**.

---

## Periods List View

| Column | Description |
|--------|-------------|
| Title | Period name |
| Type | Weekly / Monthly / Quarterly / Yearly / Exam |
| Start Date | Period start |
| End Date | Period end |
| Academic Year | e.g., "2025-2026" |
| Weight | Weight for averaging |
| Published | Yes / No |

---

## Adding a Grading Period

Navigate to **Grading Periods → Add Period**.

### Period Form

| Field | Description | Options |
|-------|-------------|---------|
| Title | Descriptive name | e.g., "October 2025 Monthly Assessment" |
| Period Type | Classification | Weekly, Monthly, Quarterly, Yearly, Exam |
| Start Date | Period start date | Date picker |
| End Date | Period end date | Date picker |
| Academic Year | Academic year | e.g., "2025-2026" |
| Weight | For averaging | Default 1.00. Exam periods typically 2.00–3.00 |
| Published | Visible to students | Checkbox |

---

## Period Types Explained

### Weekly

- **Frequency**: Every week
- **Typical Weight**: 0.25 – 0.50
- **Use**: Regular short assessments, quizzes
- **Example**: "Week 3 Assessment — September 15-19"

### Monthly

- **Frequency**: Once per month
- **Typical Weight**: 1.00
- **Use**: Monthly progress tests
- **Example**: "October 2025 Monthly Assessment"

### Quarterly

- **Frequency**: Every 3 months
- **Typical Weight**: 1.50 – 2.00
- **Use**: Term or quarterly examinations
- **Example**: "First Quarter Exam 2025"

### Yearly

- **Frequency**: Once per academic year
- **Typical Weight**: 3.00 – 5.00
- **Use**: Final examinations
- **Example**: "Annual Final Exam 2025-2026"

### Exam

- **Frequency**: As scheduled
- **Typical Weight**: 2.00 – 3.00
- **Use**: Specific named examinations
- **Example**: "Board Exam Preparation Test"

---

## Weight System

### How Weights Work

Weights determine the contribution of each period to overall academic performance:

```
Weighted Average = SUM(Grade % × Weight) / SUM(Weights)
```

### Example Configuration

| Period | Type | Weight | Student % | Weighted |
|--------|------|--------|-----------|----------|
| Week 1 Quiz | Weekly | 0.25 | 85% | 21.25 |
| Week 2 Quiz | Weekly | 0.25 | 78% | 19.50 |
| Week 3 Quiz | Weekly | 0.25 | 92% | 23.00 |
| Week 4 Quiz | Weekly | 0.25 | 80% | 20.00 |
| Monthly Test | Monthly | 1.00 | 76% | 76.00 |
| Term Exam | Exam | 3.00 | 88% | 264.00 |
| **Total** | | **5.00** | | **84.75%** |

### Best Practice

| Period Type | Recommended Weight |
|-------------|-------------------|
| Weekly Quiz | 0.25 |
| Monthly Test | 1.00 |
| Quarterly Exam | 2.00 |
| Final Exam | 3.00 – 5.00 |

---

## Publishing Periods

### Draft Periods

- Visible only to administrators and teachers
- Grades can be entered but not visible to students/parents
- Period can be edited freely

### Published Periods

- Grades become visible in the Parent Portal
- Students can view their performance
- Period details are locked

Check the **Published** checkbox to publish a period.

---

## Period Meta Box

Each period has a custom meta box with these fields stored as post meta:

| Meta Key | Type | Purpose |
|----------|------|---------|
| `_ik_period_type` | VARCHAR | One of 5 period types |
| `_ik_start_date` | DATE | Period start |
| `_ik_end_date` | DATE | Period end |
| `_ik_academic_year` | VARCHAR | Academic year identifier |
| `_ik_weight` | DECIMAL(5,2) | Weight for averaging |
| `_ik_is_published` | VARCHAR | "1" or "0" |

---

## Period Integration

Grading periods integrate with multiple modules:

### Gradebook

Grades in `ik_grades_v2` are linked to periods via `period_id` and `period_type`.

### Parent Portal

The parent portal dynamically shows grades based on the current period:

- **Weekly**: Shows current week's grades (if within date range)
- **Monthly**: Shows monthly averages at month end (4th week)
- **Yearly**: Shows final exam results at year end
- **Fallback**: Shows 5 most recent grades if no current period match

### Report Cards

Report cards aggregate grades by exam type periods.

---

## Campus Filtering

Grading periods are campus-scoped. The `ik_periods` table includes `campus_id`:

```sql
CREATE TABLE wp_ik_periods (
    period_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    period_type VARCHAR(20) DEFAULT 'monthly',
    start_date DATE,
    end_date DATE,
    academic_year VARCHAR(9),
    weight DECIMAL(5,2) DEFAULT 1.00,
    is_published TINYINT(1) DEFAULT 0,
    campus_id BIGINT(20) UNSIGNED,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME ON UPDATE CURRENT_TIMESTAMP
);
```

---

## Programmatic Usage

### Get Periods by Type

```php
$args = [
    'post_type'      => 'ik_period',
    'posts_per_page' => -1,
    'meta_key'       => '_ik_period_type',
    'meta_value'     => 'monthly',
    'meta_query'     => [
        [
            'key'     => '_ik_start_date',
            'value'   => date('Y-m-01'),
            'compare' => '<=',
        ],
        [
            'key'     => '_ik_end_date',
            'value'   => date('Y-m-t'),
            'compare' => '>=',
        ],
    ],
];
$periods = get_posts($args);
```

### Get Current Period

```php
$today = date('Y-m-d');
$args = [
    'post_type'      => 'ik_period',
    'posts_per_page' => 1,
    'meta_query'     => [
        ['key' => '_ik_start_date', 'value' => $today, 'compare' => '<='],
        ['key' => '_ik_end_date', 'value' => $today, 'compare' => '>='],
    ],
];
$current_period = get_posts($args);
```

### Create a Period Programmatically

```php
$period_id = wp_insert_post([
    'post_type'   => 'ik_period',
    'post_title'  => 'Monthly Assessment — June 2026',
    'post_status' => 'publish',
]);

update_post_meta($period_id, '_ik_period_type', 'monthly');
update_post_meta($period_id, '_ik_start_date', '2026-06-01');
update_post_meta($period_id, '_ik_end_date', '2026-06-30');
update_post_meta($period_id, '_ik_academic_year', '2025-2026');
update_post_meta($period_id, '_ik_weight', '1.00');
update_post_meta($period_id, '_ik_is_published', '0');
```
