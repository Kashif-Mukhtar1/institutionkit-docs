```markdown
# Payroll & Expenses Dashboard

The Payroll & Expenses Dashboard provides a comprehensive financial overview with real-time statistics, charts, campus comparison, alerts, and pending approvals — all in one view.

---

## Accessing the Dashboard

Navigate to **Payroll & Expenses → Dashboard**.

---

## Campus Selector

At the top of the dashboard, a campus selector allows:

| Selection | Effect |
|-----------|--------|
| All Campuses | Aggregated data across all campuses |
| Specific Campus | Data filtered to selected campus |

---

## Stats Grid

Four key financial metrics displayed as cards:

| Card | Description | Calculation |
|------|-------------|-------------|
| 💰 **Total Revenue (MTD)** | Month-to-date collections | Sum of all campus collections |
| 📉 **Total Expenses (MTD)** | Month-to-date approved expenses | Sum of campus direct + central allocated expenses |
| 👥 **Payroll (MTD)** | Month-to-date payroll costs | Sum of net payable for current month |
| 📈 **Net Surplus** | Revenue - Expenses - Payroll | Profitability indicator |

---

## Charts Section

### Monthly Trend Chart

A line chart showing the last 6 months of revenue vs expenses:

```
Monthly Trend
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Jan     ████████████░░░░  Revenue
        ████████░░░░░░░░  Expenses

Feb     ██████████████░░  Revenue
        ██████████░░░░░░  Expenses

Mar     ████████████████  Revenue
        ██████████░░░░░░  Expenses
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Data is fetched via AJAX:

```javascript
$.ajax({
    url: ik_ajax.ajax_url,
    data: {
        action: 'ik_get_dashboard_data',
        nonce: ik_ajax.nonce,
        campus_id: selectedCampus
    },
    success: function(response) {
        renderTrendChart(response.data.trend);
        renderBreakdownChart(response.data.breakdown);
        renderComparisonTable(response.data.comparison);
        renderAlerts(response.data.alerts);
        renderPendingApprovals(response.data.pending);
    }
});
```

### Expense Breakdown Chart

A donut/pie chart showing the top 5 expense heads for the current month:

```
Expense Breakdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    ┌──────────┐
   ╱    30%    ╲  Building Rent
  │  Utilities   │
  │     20%      │
  │  15%  ┌──────┤  Staff Welfare
   ╲ 25%  │ 10% ╱   Maintenance
    └──────┴──────┘
```

### Campus Comparison Chart (All Campuses view)

A bar chart comparing key metrics across campuses:

```
Campus Comparison
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Main Campus    ████████████████████ 85.2M
Downtown       ██████████████░░░░░░ 58.7M
Northwest      ████████████████░░░░ 72.1M
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Alerts Section

The dashboard generates automatic financial alerts:

| Alert Type | Trigger | Severity |
|------------|---------|----------|
| Budget Overrun | Actual spending exceeds 90% of budget | ⚠️ Warning |
| Budget Exceeded | Actual spending exceeds 100% of budget | 🔴 Critical |
| Unbudgeted Expense | Spending on head with no budget | ⚠️ Warning |
| Large Transaction | Single expense over 50,000 | 🔴 High |
| Missing Recurring | Fixed expense missing this month | ⚠️ Warning |
| Spending Spike | Current month > 150% of 6-month average | 🟡 Medium |

Each alert shows:
- Campus name
- Expense head
- Current amount vs budget/threshold
- Percentage used

---

## Pending Approvals Section

Displays the top 5 expenses awaiting approval:

| Column | Description |
|--------|-------------|
| Date | Expense date |
| Campus | Campus name |
| Head | Expense category |
| Description | Expense description |
| Amount | Expense amount |
| Status | Pending Approval badge |

---

## Recent Expenses Table

The bottom section shows the 10 most recent expenses:

| Column | Description |
|--------|-------------|
| Date | Expense date |
| Campus | Campus name |
| Head | Expense head with colored indicator |
| Description | Truncated description |
| Amount | Formatted amount |
| Status | Status badge |

---

## Dashboard Data Structure

The `ik_get_dashboard_data` AJAX endpoint returns:

```json
{
    "summary": {
        "revenue": { "total_collections": 500000 },
        "expenses": {
            "campus_direct": 150000,
            "central_allocated": 50000,
            "payroll_campus": 200000,
            "payroll_central": 0
        },
        "profit": {
            "net_surplus": 100000,
            "profit_margin": 20.0
        },
        "metrics": {
            "student_count": 250,
            "cost_per_student": 1600,
            "revenue_per_student": 2000
        }
    },
    "trend": [
        { "month": "Jan 2026", "revenue": 450000, "expenses": 380000 },
        { "month": "Feb 2026", "revenue": 480000, "expenses": 390000 }
    ],
    "breakdown": [
        { "head_name": "Building Rent", "amount": 80000, "color_code": "#4CAF50" },
        { "head_name": "Utilities", "amount": 45000, "color_code": "#FF9800" }
    ],
    "comparison": [
        { "campus_name": "Main Campus", "revenue": 300000, "expenses": 220000, "net_surplus": 80000 }
    ],
    "alerts": [
        { "type": "budget_overrun", "severity": "warning", "message": "..." }
    ],
    "pending": [
        { "expense_id": 42, "amount": 15000, "head_name": "Utilities", "status": "pending_approval" }
    ]
}
```

---

## Aggregated vs Single Campus

### All Campuses (campus_id = null)

```php
private function get_aggregated_summary($campus_id) {
    if ($campus_id !== null) {
        return $this->db->get_campus_financial_summary($campus_id, ...);
    }
    
    // Aggregate all campuses
    $campuses = $this->db->get_campuses();
    $summary = [/* initialized to zero */];
    
    foreach ($campuses as $campus) {
        $c_summary = $this->db->get_campus_financial_summary(...);
        // Sum all values
    }
    
    return $summary;
}
```

### Single Campus

Direct query to `get_campus_financial_summary()` for the selected campus.

---

## Performance Optimization

| Technique | Implementation |
|-----------|---------------|
| Transient Caching | 5-minute TTL on all dashboard data |
| Campus-Specific Queries | Indexed `campus_id` columns |
| Aggregated Queries | Single-pass summation for All Campuses |
| Chart Data | Localized JSON, no additional AJAX |
| Cache Clearing | On any financial data mutation |

---

## Programmatic Usage

### Get Dashboard Data

```php
// Via AJAX
$.post(ajaxurl, {
    action: 'ik_get_dashboard_data',
    nonce: ik_ajax.nonce,
    campus_id: 3
}, function(response) {
    console.log(response.data.summary);
    console.log(response.data.trend);
    console.log(response.data.alerts);
});

// Via PHP
$summary = $db->get_campus_financial_summary($campus_id, '2026-06-01', '2026-06-30');
$trend = $db->get_monthly_trend($campus_id, 6);
$breakdown = $db->get_expense_breakdown('2026-06-01', '2026-06-30', $campus_id);
$alerts = $db->get_financial_alerts($campus_id);
```
