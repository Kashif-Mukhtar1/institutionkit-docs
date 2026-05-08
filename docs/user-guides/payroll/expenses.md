```markdown
# Expense Management

The Expense Management module handles all campus expenditures — recording expenses with categorization, tracking payment sources, managing multi-level approvals, and generating reports.

---

## Accessing Expenses

Navigate to **Payroll & Expenses → Expenses**.

---

## Expenses List View

### Filter Bar

| Filter | Description |
|--------|-------------|
| Campus | Filter by campus |
| Status | All / Pending / Approved / Reimbursed / Rejected |
| Month | Filter by month |
| Apply Filters | Submit |
| Reset | Clear all filters |

### Expense Table

| Column | Description |
|--------|-------------|
| ID | Expense number |
| Date | Expense date |
| Campus | Campus name |
| Head | Expense category |
| Description | First 40 characters |
| Amount | Formatted amount |
| Paid By | Campus / Central / Split |
| Status | Status badge |
| Actions | Approve / Reject (for pending) |

---

## Adding an Expense

Click **+ Add New Expense** to open the Add Expense modal.

### Form Sections

#### Basic Information

| Field | Required | Description |
|-------|----------|-------------|
| Campus | Yes | Select campus |
| Expense Head | Yes | Select from configured heads |
| Amount | Yes | Expense amount |
| Expense Date | Yes | Date of expense |
| Paid By | Yes | Campus Petty Cash / Central Admin / Split |
| Campus Share % | If Split | Percentage paid by campus |
| Payment Method | No | Cash / Bank Transfer / Cheque / Card / Online |
| Severity | No | Routine / Important / Emergency |

#### Vendor & Reference

| Field | Required | Description |
|-------|----------|-------------|
| Vendor/Payee | No | Name of vendor or payee |
| Invoice Number | No | Vendor invoice reference |

#### Description

| Field | Required | Description |
|-------|----------|-------------|
| Description | Yes | Detailed explanation of the expense |

#### Internal Notes

| Field | Required | Description |
|-------|----------|-------------|
| Admin/Finance Only | No | Notes visible only to finance team |

---

## Payment Sources

### Campus Petty Cash

Entire amount is allocated to the campus budget:

```
campus_amount = total_amount
central_amount = 0
```

### Central Admin

Entire amount is allocated to central administration:

```
campus_amount = 0
central_amount = total_amount
```

### Split Payment

Amount is divided between campus and central:

```
campus_amount = total_amount × (split_ratio / 100)
central_amount = total_amount × ((100 - split_ratio) / 100)
```

**Example:** 100,000 expense with 60% campus share
- Campus: 60,000
- Central: 40,000

---

## Approval Workflow

### Approval Levels

When an expense is created, the system automatically generates approval requests:

```php
private function create_expense_approvals($expense_id, $campus_id, $amount) {
    $levels_needed = 1;
    if ($amount > 10000) $levels_needed = 2;
    if ($amount > 50000) $levels_needed = 3;
    
    $approvers = [
        1 => $campus_manager_id,
        2 => $finance_manager_id,
        3 => $director_id,
    ];
    
    for ($level = 1; $level <= $levels_needed; $level++) {
        $wpdb->insert($approvals_table, [
            'expense_id'     => $expense_id,
            'approver_id'    => $approvers[$level],
            'approval_level' => $level,
            'status'         => 'pending',
        ]);
    }
}
```

### Approving an Expense

```php
public function approve_expense($expense_id, $approver_id) {
    // Mark this approver's record as approved
    $wpdb->update($approvals_table, [
        'status' => 'approved',
        'action_date' => current_time('mysql'),
    ], [
        'expense_id' => $expense_id,
        'approver_id' => $approver_id,
        'status' => 'pending',
    ]);
    
    // Check if all levels are approved
    $pending = $wpdb->get_var("SELECT COUNT(*) FROM {$approvals_table} 
        WHERE expense_id = $expense_id AND status = 'pending'");
    
    if ($pending == 0) {
        // Fully approved
        $wpdb->update($expenses_table, [
            'status' => 'approved',
            'approved_by' => $approver_id,
            'approved_at' => current_time('mysql'),
        ], ['expense_id' => $expense_id]);
        
        return 'fully_approved';
    }
    
    return 'partially_approved';
}
```

### Rejecting an Expense

```php
public function reject_expense($expense_id, $approver_id, $reason) {
    $wpdb->update($approvals_table, [
        'status' => 'rejected',
        'comments' => $reason,
    ], ['expense_id' => $expense_id, 'approver_id' => $approver_id]);
    
    $wpdb->update($expenses_table, [
        'status' => 'rejected',
        'rejection_reason' => $reason,
    ], ['expense_id' => $expense_id]);
}
```

---

## Expense Status Flow

```
Pending Approval → Approved → Reimbursed
       │
       └── Rejected (with reason)
```

| Status | Meaning | Can Edit? |
|--------|---------|:---:|
| Pending Approval | Awaiting review | Yes |
| Approved | Approved, awaiting payment | No |
| Reimbursed | Payment completed | No |
| Rejected | Denied with reason | No |

---

## Severity Flags

| Flag | Color | Use For |
|------|-------|---------|
| 🟢 Routine | Green | Regular operational expenses |
| 🟡 Important | Yellow | Significant but not urgent |
| 🔴 Emergency | Red | Urgent, requires immediate attention |

---

## Expense Heads

Expense heads categorize all expenditures. Access at **Payroll & Expenses → Expense Heads**.

### Default Heads

15 system-default expense heads are created on installation:

| Head | Type | Icon |
|------|------|------|
| Building Rent | Fixed | 🏢 |
| Utilities | Variable | ⚡ |
| Maintenance & Repairs | Variable | 🔧 |
| Cleanliness & Sanitation | Variable | 🧹 |
| Security Services | Fixed | 🛡️ |
| Office Supplies | Variable | 📎 |
| Staff Welfare | Variable | ❤️ |
| Entertainment & Events | Variable | 🎉 |
| Transportation | Variable | 🚌 |
| IT & Software | Variable | 💻 |
| Marketing & Advertising | Variable | 📢 |
| Insurance | Fixed | 🔒 |
| Professional Fees | Variable | ⚖️ |
| Staff Training | Variable | 👨‍🏫 |
| Miscellaneous | Variable | ••• |

### Adding Custom Heads

Click **+ Add Custom Head**:

| Field | Description |
|-------|-------------|
| Head Name | e.g., "Laboratory Supplies" |
| Head Type | Fixed / Variable / Emergency |
| Campus | Global (all campuses) or campus-specific |
| Icon | Font Awesome class |
| Color | Hex color for charts and indicators |

### Deleting Heads

System-default heads cannot be deleted — only deactivated. Custom heads can be soft-deleted (set to inactive).

---

## Expense Database

```sql
CREATE TABLE wp_institutionkit_expenses (
    expense_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    campus_id BIGINT(20) UNSIGNED NOT NULL,
    head_id BIGINT(20) UNSIGNED NOT NULL,
    amount DECIMAL(12,2) NOT NULL,
    paid_by VARCHAR(20) DEFAULT 'campus_petty',
    split_ratio DECIMAL(5,2),
    campus_amount DECIMAL(12,2) DEFAULT 0.00,
    central_amount DECIMAL(12,2) DEFAULT 0.00,
    description TEXT NOT NULL,
    vendor_name VARCHAR(255),
    invoice_number VARCHAR(100),
    status VARCHAR(20) DEFAULT 'pending_approval',
    severity_flag VARCHAR(10) DEFAULT 'green',
    expense_date DATE NOT NULL,
    payment_method VARCHAR(50),
    approved_by BIGINT(20) UNSIGNED,
    approved_at DATETIME,
    rejection_reason TEXT,
    internal_notes TEXT,
    created_by BIGINT(20) UNSIGNED NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## Programmatic Usage

### Add Expense

```php
$expense_id = $db->add_expense([
    'campus_id'      => 3,
    'head_id'        => 2,
    'amount'         => 15000,
    'paid_by'        => 'campus_petty',
    'description'    => 'Monthly electricity bill',
    'vendor_name'    => 'Power Company',
    'invoice_number' => 'INV-2026-0642',
    'expense_date'   => '2026-06-15',
    'payment_method' => 'bank_transfer',
    'severity_flag'  => 'green',
]);
```

### Get Expenses

```php
$expenses = $db->get_expenses([
    'campus_id'    => 3,
    'status'       => 'pending_approval',
    'date_from'    => '2026-06-01',
    'date_to'      => '2026-06-30',
    'severity_flag' => 'red',
]);
```

### Approve Expense

```php
$result = $db->approve_expense($expense_id, get_current_user_id());
// Returns 'fully_approved' or 'partially_approved'
```

### Reject Expense

```php
$db->reject_expense($expense_id, get_current_user_id(), 'Insufficient documentation');
```

### Get Expense Breakdown

```php
$breakdown = $db->get_expense_breakdown('2026-06-01', '2026-06-30', $campus_id);
// Returns top 5 expense heads with amounts
```
