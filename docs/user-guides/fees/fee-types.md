```markdown
# Fee Types

Fee Types are the foundational building blocks of the fee management system. They define the categories of fees your institution charges — tuition, transport, library, exam fees, and more.

---

## Accessing Fee Types

Navigate to **InstitutionKit → Fee Management → Fee Types**.

---

## Fee Types List View

The Fee Types page displays all defined fee types in a table:

| Column | Description |
|--------|-------------|
| Name | Fee type name |
| Actions | Edit / Delete |

---

## Adding a Fee Type

Click **Add New Fee Type** or the **+ Add** button.

| Field | Required | Description |
|-------|----------|-------------|
| Fee Name | Yes | Unique name (e.g., "Tuition Fee", "Transport Fee") |

### Naming Conventions

| Good Names | Avoid |
|------------|-------|
| Tuition Fee | Fee 1 |
| Transport Fee | Monthly |
| Library Fee | Misc |
| Exam Fee | Other Fee |

Use descriptive, specific names. These appear on invoices and reports.

---

## Editing a Fee Type

Click **Edit** on any fee type row to rename it.

!!! warning "Rename with Caution"
    Renaming a fee type affects all existing fee structures and student assignments that use it. Historical invoices will show the new name.

---

## Deleting a Fee Type

Click **Delete** to remove a fee type.

!!! danger "Cannot Delete In-Use Fee Types"
    If a fee type is currently used in any fee structure or assigned to any student, it cannot be deleted. Remove it from all structures and assignments first.

---

## Fee Type Database

Fee types are stored in a simple table:

```sql
CREATE TABLE wp_institutionkit_fee_types (
    fee_type_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    fee_name VARCHAR(255) NOT NULL
);
```

---

## Fee Type Usage

Fee types flow through the system:

```
Fee Types (definition)
    │
    ▼
Fee Structures (grouped with amounts)
    │
    ▼
Student Fee Assignments (individual students)
    │
    ▼
Invoice Items (on generated invoices)
```

---

## Recommended Fee Types

### Standard School Fees

| Fee Type | Frequency | Typical Use |
|----------|----------|-------------|
| Tuition Fee | Monthly / Term | Core academic instruction |
| Admission Fee | One-time | New student enrollment |
| Registration Fee | Annual | Yearly registration |
| Exam Fee | Per-term | Examination costs |

### Optional Service Fees

| Fee Type | Frequency | Typical Use |
|----------|----------|-------------|
| Transport Fee | Monthly | Bus/transport service |
| Library Fee | Annual | Library access |
| Computer Lab Fee | Monthly / Term | Technology usage |
| Sports Fee | Term / Annual | Sports activities |
| Music Fee | Term | Music lessons |
| Art Fee | Term | Art supplies |

### Miscellaneous Fees

| Fee Type | Frequency | Typical Use |
|----------|----------|-------------|
| Late Payment Fee | Per occurrence | Penalty for overdue payment |
| Certificate Fee | Per request | Duplicate certificate issuance |
| Development Fee | Annual | Infrastructure development |
| Activity Fee | Per event | Field trips, events |
| Uniform Fee | One-time | School uniform |

---

## Programmatic Usage

### Add a Fee Type

```php
global $wpdb;
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_fee_types",
    ['fee_name' => 'Tuition Fee']
);
$fee_type_id = $wpdb->insert_id;
```

### Get All Fee Types

```php
global $wpdb;
$fee_types = $wpdb->get_results(
    "SELECT * FROM {$wpdb->prefix}institutionkit_fee_types ORDER BY fee_name ASC"
);
```

### Get Fee Type by ID

```php
global $wpdb;
$fee_type = $wpdb->get_row($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_fee_types WHERE fee_type_id = %d",
    $fee_type_id
));
```

### Update a Fee Type

```php
global $wpdb;
$wpdb->update(
    "{$wpdb->prefix}institutionkit_fee_types",
    ['fee_name' => 'Updated Fee Name'],
    ['fee_type_id' => $fee_type_id]
);
```

### Delete a Fee Type

```php
global $wpdb;
$wpdb->delete(
    "{$wpdb->prefix}institutionkit_fee_types",
    ['fee_type_id' => $fee_type_id]
);
```

---

## Next Steps

After defining fee types:

1. **[Create Fee Structures](fee-structures.md)** — Group fee types with amounts
2. **[Assign Fees](fees/../fee-structures.md)** — Apply structures to students
3. **[Generate Invoices](invoices.md)** — Create billable invoices
```
