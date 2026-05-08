```markdown
# Fee Structures

Fee Structures group fee types with specific amounts, creating reusable billing templates that can be assigned to students by class, campus, or individually.

---

## Accessing Fee Structures

Navigate to **InstitutionKit → Fee Management → Fee Structures**.

---

## Fee Structures List View

| Column | Description |
|--------|-------------|
| Name | Structure name (e.g., "Primary Monthly Fees") |
| Items Count | Number of fee types included |
| Total Amount | Sum of all fee type amounts |
| Actions | Edit / Delete |

---

## Creating a Fee Structure

### Step 1: Basic Information

Click **Add New Fee Structure**.

| Field | Required | Description |
|-------|----------|-------------|
| Structure Name | Yes | Descriptive name, e.g., "Grade 1-5 Monthly Fees" |

### Step 2: Add Fee Items

For each fee type you want to include:

| Field | Description |
|-------|-------------|
| Fee Type | Select from your defined fee types |
| Amount | The amount to charge for this fee type |

### Example Structure

**Structure Name:** "Grade 1-5 Monthly Fees"

| Fee Type | Amount |
|----------|--------|
| Tuition Fee | 3,500 |
| Transport Fee | 1,000 |
| Library Fee | 200 |
| Computer Lab Fee | 500 |
| **Total** | **5,200** |

---

## Editing a Fee Structure

Click **Edit** on any structure to:

- Change the structure name
- Add new fee items
- Remove existing fee items
- Modify fee amounts

!!! warning "Changes Affect Future Invoices Only"
    Editing a fee structure does not retroactively change existing invoices. Only new invoices generated after the edit will use the updated amounts.

---

## Deleting a Fee Structure

Click **Delete** to remove a structure.

!!! danger "Check Assignments First"
    If the structure is assigned to students, those assignments become invalid. Unassign the structure from all students before deleting.

---

## Fee Structure Database

Fee structures use three related tables:

### Structure Header

```sql
CREATE TABLE wp_institutionkit_fee_structures (
    structure_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    structure_name VARCHAR(255) NOT NULL
);
```

### Structure Items

```sql
CREATE TABLE wp_institutionkit_fee_structure_items (
    item_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    structure_id BIGINT(20) UNSIGNED NOT NULL,
    fee_type_id BIGINT(20) UNSIGNED NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    KEY structure_id (structure_id)
);
```

---

## Organizing Fee Structures

### By Class Level

| Structure Name | Classes | Example Total |
|---------------|---------|---------------|
| Pre-Primary Fees | Nursery, KG | 3,000 |
| Primary Fees | Grade 1-5 | 5,200 |
| Middle School Fees | Grade 6-8 | 6,500 |
| Secondary Fees | Grade 9-10 | 8,000 |
| Senior Secondary Fees | Grade 11-12 | 10,000 |

### By Campus

Different campuses may have different fee amounts:

| Campus | Structure Name | Total |
|--------|---------------|-------|
| Main Campus | Primary Monthly Fees | 5,200 |
| Downtown Campus | Primary Monthly Fees | 4,800 |
| Northwest Campus | Primary Monthly Fees | 4,500 |

### By Academic Year

Create new structures each year for fee adjustments:

| Academic Year | Structure Name |
|---------------|---------------|
| 2025-2026 | Primary Fees 2025-26 |
| 2026-2027 | Primary Fees 2026-27 |

---

## Assigning Fee Structures

Once created, structures must be assigned to students. Navigate to **Fee Management → Assign Fees**.

### Assignment Methods

| Method | Steps |
|--------|-------|
| **Individual Student** | Search student → Select structure → Assign |
| **By Class** | Select class → Select structure → Assign to all |
| **By Campus** | Select campus → Select structure → Assign to all |
| **Bulk Select** | Manually check multiple students → Assign |

---

## Programmatic Usage

### Create a Fee Structure

```php
global $wpdb;

// Create structure header
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_fee_structures",
    ['structure_name' => 'Primary Monthly Fees']
);
$structure_id = $wpdb->insert_id;

// Add fee items
$items = [
    ['fee_type_id' => 1, 'amount' => 3500],  // Tuition
    ['fee_type_id' => 2, 'amount' => 1000],  // Transport
    ['fee_type_id' => 3, 'amount' => 200],   // Library
];

foreach ($items as $item) {
    $wpdb->insert(
        "{$wpdb->prefix}institutionkit_fee_structure_items",
        [
            'structure_id' => $structure_id,
            'fee_type_id'  => $item['fee_type_id'],
            'amount'       => $item['amount'],
        ]
    );
}
```

### Get Structure with Items

```php
global $wpdb;

// Get structure
$structure = $wpdb->get_row($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_fee_structures 
     WHERE structure_id = %d",
    $structure_id
));

// Get items with fee type names
$items = $wpdb->get_results($wpdb->prepare(
    "SELECT 
        si.*,
        ft.fee_name
     FROM {$wpdb->prefix}institutionkit_fee_structure_items si
     JOIN {$wpdb->prefix}institutionkit_fee_types ft ON si.fee_type_id = ft.fee_type_id
     WHERE si.structure_id = %d",
    $structure_id
));

// Calculate total
$total = array_sum(array_column($items, 'amount'));
```

### Get Total Amount of a Structure

```php
global $wpdb;
$total = $wpdb->get_var($wpdb->prepare(
    "SELECT SUM(amount) 
     FROM {$wpdb->prefix}institutionkit_fee_structure_items 
     WHERE structure_id = %d",
    $structure_id
));
```

---

## Next Steps

After creating fee structures:

1. **[Assign Fees to Students](../fees/../fee-structures.md)** — Apply structures to individuals or classes
2. **[Generate Invoices](invoices.md)** — Create bills from assigned fees
```
