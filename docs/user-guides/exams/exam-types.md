```markdown
# Exam Types

Exam Types define the categories of examinations conducted at your institution. Each type has a name, category, default maximum marks, and weight for final grade calculation.

---

## Accessing Exam Types

Navigate to **InstitutionKit → Exams → Exam Types**.

---

## Exam Types List View

| Column | Description |
|--------|-------------|
| Checkbox | For bulk actions |
| Exam Name | Display name |
| Category | Term, Midterm, Final, Monthly, Board, Quiz, Assessment |
| Max Marks | Default maximum marks |
| Weight % | Weight in final grade calculation |
| Status | Active or Inactive |
| Created | Creation date |
| Actions | Edit / Delete |

---

## Adding an Exam Type

Click **Add New** to open the exam type form.

| Field | Required | Description |
|-------|----------|-------------|
| Exam Name | Yes | e.g., "First Term Exam 2025" |
| Category | Yes | Select from 7 predefined categories |
| Maximum Marks | Yes | Default: 100. Valid range: 1-9999 |
| Weight Percentage | Yes | Default: 100. Range: 0-100 |
| Status | Yes | Active / Inactive |

### Weight Percentage Explained

The weight determines how much this exam type contributes to final grade calculations:

| Scenario | Weights | Effect |
|----------|---------|--------|
| Monthly Test + Term Exam | Monthly: 30%, Term: 70% | Term exam counts more than double |
| Multiple Quizzes | Quiz 1: 20%, Quiz 2: 20%, Final: 60% | Final dominates the average |
| Equal Weight | All 100% | Simple average |

**Formula:**
```
Weighted Average = SUM(Percentage × Weight) ÷ SUM(Weights)
```

### Example Calculation

| Exam | Marks | Max | % | Weight | Weighted |
|------|-------|-----|---|--------|----------|
| Monthly | 72 | 100 | 72% | 30 | 21.6 |
| Term | 85 | 100 | 85% | 70 | 59.5 |
| **Weighted Average** | | | | | **81.1%** |

---

## Bulk Actions

Select multiple exam types and apply:

| Action | Effect |
|--------|--------|
| Activate | Set selected types to Active |
| Deactivate | Set selected types to Inactive |
| Delete | Remove selected types (soft delete if in use) |

---

## Soft Delete Protection

If an exam type is used in any exam schedule, it cannot be permanently deleted. Instead, it is deactivated:

```php
public function delete_exam_type($exam_type_id) {
    // Check if in use
    $usage_count = $wpdb->get_var($wpdb->prepare(
        "SELECT COUNT(*) FROM {$schedules_table} WHERE exam_type_id = %d",
        $exam_type_id
    ));
    
    if ($usage_count > 0) {
        // Soft delete — deactivate instead
        return $this->toggle_exam_type_status($exam_type_id, false);
    }
    
    // Hard delete
    return $wpdb->delete($table, ['exam_type_id' => $exam_type_id]);
}
```

---

## Exam Type Database

```sql
CREATE TABLE wp_ik_exam_types (
    exam_type_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    exam_name VARCHAR(100) NOT NULL,
    exam_category VARCHAR(30) DEFAULT 'term',
    max_marks DECIMAL(6,2) DEFAULT 100.00,
    weight_percentage DECIMAL(5,2) DEFAULT 100.00,
    is_active TINYINT(1) DEFAULT 1,
    campus_id BIGINT UNSIGNED,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## Category Reference

```php
public function get_exam_category_labels() {
    return [
        'term'       => 'Term Exam',
        'midterm'    => 'Mid-Term Exam',
        'final'      => 'Final Exam',
        'monthly'    => 'Monthly Test',
        'board'      => 'Board Exam',
        'quiz'       => 'Quiz',
        'assessment' => 'Assessment',
    ];
}
```

---

## Filtering by Category

The Exam Types page includes a category filter dropdown:

1. Select a category from the dropdown
2. Click **Filter**
3. Only exam types in that category are shown

---

## Programmatic Usage

### Create an Exam Type

```php
$exam_module = IK_Exam_Module::get_instance();

$result = $exam_module->create_exam_type([
    'exam_name'         => 'First Term Exam 2026',
    'exam_category'     => 'term',
    'max_marks'         => 100.00,
    'weight_percentage' => 70.00,
    'is_active'         => 1,
    'campus_id'         => $campus_id,
]);
```

### Get Active Exam Types

```php
$exam_types = $exam_module->get_exam_types([
    'campus_id'   => $campus_id,
    'active_only' => true,
    'category'    => 'term',
]);
```

### Toggle Exam Type Status

```php
$exam_module->toggle_exam_type_status($exam_type_id, false); // Deactivate
$exam_module->toggle_exam_type_status($exam_type_id, true);  // Activate
```

### Delete an Exam Type

```php
$exam_module->delete_exam_type($exam_type_id);
// Returns true if hard-deleted, false if soft-deleted (in use)
```

### Get Exam Type Count

```php
$count = $exam_module->get_exam_types_count($campus_id);
```
