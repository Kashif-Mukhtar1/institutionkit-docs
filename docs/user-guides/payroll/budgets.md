```markdown
# Budget Management

The Budgets module enables campus-level budget planning — setting monthly spending limits per expense head, tracking actual spending against budgets, and generating variance reports with utilization alerts.

---

## Accessing Budgets

Navigate to **Payroll & Expenses → Budgets**.

---

## Budget Page Layout

### Filter Section

| Filter | Description |
|--------|-------------|
| Campus | Select the campus |
| Month | Select budget month |

Click **Load Budgets** to view or manage budgets for the selected campus and month.

---

## Budget Table

After loading, the budget table displays:

| Column | Description |
|--------|-------------|
| Expense Head | Name with color indicator |
| Budget Amount | Planned spending limit |
| Actual Spent | Real spending to date |
| Variance | Budget - Actual (±) |
| Status | On Track / Near Limit / Over Budget |
| Actions | Set Budget / Edit / Delete |

### Status Indicators

| Status | Trigger | Color |
|--------|---------|-------|
| On Track | Less than 80% used | Green |
| Near Limit | 80% – 99% used | Yellow |
| Over Budget | 100% or more used | Red |

---

## Setting a Budget

Click **Set Budget** (or **Edit**) on any expense head row.

### Budget Form

| Field | Required | Description |
|-------|----------|-------------|
| Expense Head | Auto-filled | The expense category |
| Budget Amount | Yes | Planned spending for the month |
| Notes | No | Optional notes |

Click **Save Budget** to create or update.

### Upsert Logic

The system checks if a budget already exists for this campus/head/month combination:

```php
$existing = $wpdb->get_var($wpdb->prepare(
    "SELECT budget_id FROM {$table} 
     WHERE campus_id = %d AND head_id = %d AND budget_month = %s",
    $campus_id, $head_id, $budget_month . '-01'
));

if ($existing) {
    // Update existing budget
    $wpdb->update($table, [
        'budget_amount' => $budget_amount,
        'notes'         => $notes,
        'updated_at'    => current_time('mysql'),
    ], ['budget_id' => $existing]);
} else {
    // Insert new budget
    $wpdb->insert($table, [
        'campus_id'     => $campus_id,
        'head_id'       => $head_id,
        'budget_month'  => $budget_month . '-01',
        'budget_amount' => $budget_amount,
        'notes'         => $notes,
        'created_by'    => get_current_user_id(),
    ]);
}
```

---

## Deleting a Budget

Click **Delete** on any budget row. You'll be asked to confirm before the budget is permanently removed.

Deleted budgets no longer generate alerts or appear in comparisons.

---

## Budget Summary Cards

Above the table, four summary cards display totals:

| Card | Value |
|------|-------|
| Total Budget | Sum of all budget amounts |
| Actual Spent | Sum of all actual spending |
| Remaining | Total Budget - Actual Spent |
| Budget Utilization | (Actual ÷ Budget) × 100 |

---

## Actual vs Budget Tracking

Actual spending is calculated from approved and reimbursed expenses in the same month:

```sql
SELECT 
    h.head_id,
    h.head_name,
    COALESCE(b.budget_amount, 0) as budget_amount,
    COALESCE(SUM(e.campus_amount), 0) as actual_amount
FROM wp_institutionkit_expense_heads h
LEFT JOIN wp_institutionkit_expense_budgets b 
    ON h.head_id = b.head_id 
    AND b.campus_id = ? 
    AND b.budget_month = ?
LEFT JOIN wp_institutionkit_expenses e 
    ON h.head_id = e.head_id 
    AND e.campus_id = ? 
    AND e.expense_date BETWEEN ? AND ?
    AND e.status IN ('approved', 'reimbursed')
WHERE h.is_active = 1
GROUP BY h.head_id
ORDER BY h.head_name
```

---

## Financial Alerts from Budgets

The system generates alerts when budgets are at risk:

| Alert | Trigger |
|-------|---------|
| Budget at 90% | Actual ≥ 90% of budget |
| Budget exceeded | Actual ≥ 100% of budget |
| Unbudgeted expense | Spending on a head with no budget |
| Large variance | Actual significantly differs from budget |

These alerts appear on the Payroll & Expenses dashboard.

---

## Budget Database

```sql
CREATE TABLE wp_institutionkit_expense_budgets (
    budget_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    campus_id BIGINT(20) UNSIGNED NOT NULL,
    head_id BIGINT(20) UNSIGNED NOT NULL,
    budget_month DATE NOT NULL,
    budget_amount DECIMAL(12,2) NOT NULL,
    actual_amount DECIMAL(12,2) DEFAULT 0.00,
    variance DECIMAL(12,2) DEFAULT 0.00,
    variance_percentage DECIMAL(5,2) DEFAULT 0.00,
    notes TEXT,
    created_by BIGINT(20) UNSIGNED NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY unique_campus_head_month (campus_id, head_id, budget_month)
);
```

**Unique constraint**: One budget per combination of campus + expense head + month.

---

## Budget Modal

The Set/Edit Budget modal includes:

- Expense head name (read-only display)
- Budget amount input (numeric)
- Notes textarea (optional)
- Save and Cancel buttons

The modal supports:
- Click outside to close
- Escape key to close
- Responsive design

---

## Redirect Handling

Form submissions use safe redirects with transient-based notices:

```php
private function safe_redirect_financial($url) {
    if (headers_sent()) {
        // Fallback meta refresh
        echo '<meta http-equiv="refresh" content="0;url=' . esc_url($url) . '">';
        exit;
    }
    wp_redirect($url);
    exit;
}
```

Success/error messages are stored in transients and displayed after redirect:

```php
set_transient('ik_budget_notice', [
    'type'    => 'success',
    'message' => 'Budget updated successfully!',
], 60);
```

---

## Best Practices

1. **Set budgets at the start of each month** — Before any spending occurs
2. **Review over-budget items weekly** — Address variances early
3. **Set realistic budgets** — Use historical data from previous months
4. **Budget for all fixed expenses** — Rent, utilities, insurance should always have budgets
5. **Include a contingency buffer** — 5-10% above expected spending for variable heads
6. **Review unbudgeted alerts** — Heads with spending but no budget need attention

---

## Programmatic Usage

### Set a Budget

```php
global $wpdb;
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_expense_budgets",
    [
        'campus_id'     => 3,
        'head_id'       => 2,
        'budget_month'  => '2026-06-01',
        'budget_amount' => 50000,
        'created_by'    => get_current_user_id(),
    ]
);
```

### Get Budget vs Actual

```php
global $wpdb;
$budgets = $wpdb->get_results($wpdb->prepare(
    "SELECT 
        h.head_name,
        b.budget_amount,
        COALESCE(SUM(e.campus_amount), 0) as actual_amount
     FROM {$wpdb->prefix}institutionkit_expense_heads h
     LEFT JOIN {$wpdb->prefix}institutionkit_expense_budgets b 
         ON h.head_id = b.head_id 
         AND b.campus_id = %d 
         AND b.budget_month = %s
     LEFT JOIN {$wpdb->prefix}institutionkit_expenses e 
         ON h.head_id = e.head_id 
         AND e.campus_id = %d 
         AND e.expense_date BETWEEN %s AND %s
         AND e.status IN ('approved', 'reimbursed')
     WHERE h.is_active = 1
     GROUP BY h.head_id",
    $campus_id, $budget_month,
    $campus_id, $month_start, $month_end
));
```

### Delete a Budget

```php
global $wpdb;
$wpdb->delete(
    "{$wpdb->prefix}institutionkit_expense_budgets",
    ['budget_id' => $budget_id]
);
```
