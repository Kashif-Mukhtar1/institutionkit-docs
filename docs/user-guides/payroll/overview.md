```markdown
# Payroll & Expenses Overview

The Payroll & Expenses module is a comprehensive financial management system handling staff payroll, expense tracking, budget planning, and financial reporting — all with multi-campus support.

---

## Accessing Payroll & Expenses

Navigate to **InstitutionKit → Payroll & Expenses** in the admin menu.

---

## Dashboard Cards

The Payroll & Expenses landing page presents 8 action cards:

| Card | Destination | Description |
|------|-------------|-------------|
| 📊 **Dashboard** | Financial dashboard | Stats, charts, and financial insights |
| 💸 **Expenses** | Expense management | Record and manage campus expenses |
| 🏷️ **Expense Heads** | Expense categories | Configure expense categories |
| 💳 **Payroll** | Payroll management | Generate and manage staff payroll |
| 📊 **Budgets** | Budget planning | Set and track expense budgets |
| 👥 **Staff** | Staff management | Manage staff records and contracts |
| ✅ **Pending Approvals** | Approval workflow | Review and approve expenses |
| 📊 **P&L Report** | Profit & loss | Revenue and expense breakdown |

---

## Module Architecture

The Payroll & Expenses module uses a modular trait-based architecture:

```php
class InstitutionKit_Payroll_Expenses {
    use InstitutionKit_Core_Setup;       // Hooks, assets, modal scripts
    use InstitutionKit_Render_Dashboard; // Financial dashboard UI
    use InstitutionKit_Render_Expenses;  // Expense management UI
    use InstitutionKit_Render_Staff;     // Staff management UI
    use InstitutionKit_Render_Payroll;   // Payroll processing UI
    use InstitutionKit_Render_Financial; // Collections, budgets, comparison
    use InstitutionKit_Ajax_Handlers;    // All AJAX endpoints
    use InstitutionKit_Helpers;          // User creation, emails, utilities
    use InstitutionKit_Salary_Structure; // Salary components CRUD
    use InstitutionKit_Loans;            // Staff loan management
    use InstitutionKit_Payslips;         // Payslip generation
    use InstitutionKit_PDF_Export;       // PDF generation (Dompdf)
}
```

---

## Financial Workflow

```
Staff Management
    │
    ├── Staff Records (contracts, salaries)
    ├── Attendance Tracking (daily)
    ├── Salary Components (earnings + deductions)
    └── Loans (optional)
        │
        ▼
Payroll Generation
    │
    ├── Auto-calculate from attendance
    ├── Apply salary components
    ├── Deduct loans and taxes
    └── Generate payslips (PDF)
        │
        ▼
Expense Management
    │
    ├── Record expenses by head
    ├── Multi-level approval workflow
    ├── Track against budgets
    └── Campus cost allocation
        │
        ▼
Financial Reporting
    │
    ├── Profit & Loss statements
    ├── Campus comparison
    ├── Budget vs Actual analysis
    └── Monthly trends
```

---

## Key Concepts

### Expense Heads

Categories for classifying expenses. 15 default heads are created on installation:

| Head | Type |
|------|------|
| Building Rent | Fixed |
| Utilities | Variable |
| Maintenance & Repairs | Variable |
| Cleanliness & Sanitation | Variable |
| Security Services | Fixed |
| Office Supplies | Variable |
| Staff Welfare | Variable |
| Entertainment & Events | Variable |
| Transportation | Variable |
| IT & Software | Variable |
| Marketing & Advertising | Variable |
| Insurance | Fixed |
| Professional Fees | Variable |
| Staff Training | Variable |
| Miscellaneous | Variable |

### Payment Sources

Expenses can be paid from three sources:

| Source | Description |
|--------|-------------|
| Campus Petty Cash | Paid entirely by the campus |
| Central Admin | Paid entirely by head office |
| Split | Shared between campus and central (configurable ratio) |

### Approval Workflow

Expenses follow a multi-level approval based on amount:

| Tier | Amount Range | Approvers |
|------|-------------|-----------|
| Level 1 | Up to 10,000 | Campus Manager |
| Level 2 | 10,001 – 50,000 | Campus Manager + Finance Manager |
| Level 3 | Above 50,000 | Campus Manager + Finance Manager + Director |

---

## Dashboard Statistics

The Payroll & Expenses dashboard shows:

| Stat | Description |
|------|-------------|
| Total Revenue (MTD) | Month-to-date collections |
| Total Expenses (MTD) | Month-to-date approved expenses |
| Payroll (MTD) | Month-to-date payroll costs |
| Net Surplus | Revenue - Expenses - Payroll |
| Monthly Trend Chart | 6-month revenue vs expenses line chart |
| Expense Breakdown | Top 5 expense heads pie chart |
| Campus Comparison | Side-by-side campus financials |
| Financial Alerts | Budget overruns, anomalies |
| Pending Approvals | Expenses awaiting approval |

---

## 18 AJAX Endpoints

The module registers 18 AJAX handlers:

```php
$actions = [
    'ik_add_expense',
    'ik_get_expenses',
    'ik_approve_expense',
    'ik_reject_expense',
    'ik_add_staff',
    'ik_get_staff',
    'ik_generate_payroll',
    'ik_get_payroll',
    'ik_get_financial_summary',
    'ik_get_comparison_data',
    'ik_get_dashboard_data',
    'ik_add_expense_head',
    'ik_record_attendance',
    'ik_add_collection',
    'ik_add_salary_component',
    'ik_remove_salary_component',
    'ik_add_loan',
    'ik_get_payslip',
];
```

---

## Capabilities

| Capability | Who Has It | Purpose |
|-----------|------------|---------|
| `ik_manage_expenses` | Admin, Campus Admin, Finance | Record expenses |
| `ik_approve_expenses` | Admin, Finance | Approve pending expenses |
| `ik_view_expense_reports` | Admin, Campus Admin, Finance | View expense reports |
| `ik_manage_expense_heads` | Admin, Finance | Configure expense categories |
| `ik_manage_staff` | Admin, Campus Admin | Manage staff records |
| `ik_manage_payroll` | Admin, Campus Admin, Finance | Generate payroll |
| `ik_approve_payroll` | Admin, Finance | Approve payroll |
| `ik_view_payroll_reports` | Admin, Campus Admin, Finance | View payroll reports |
| `ik_manage_budgets` | Admin, Finance | Set and manage budgets |
| `ik_view_financial_comparison` | Admin, Finance | Cross-campus comparison |
| `ik_view_profit_loss` | Admin, Campus Admin, Finance | View P&L reports |
| `ik_view_all_campus_finances` | Admin only | View all campus finances |
| `ik_export_financial_data` | Admin, Finance | Export financial data |

---

## Dashboard Caching

Dashboard data is cached for 5 minutes using WordPress transients:

```php
$cache_key = 'ik_dashboard_data_' . ($campus_id ?? 'all');
$cached = get_transient($cache_key);

if ($cached !== false) {
    wp_send_json_success($cached);
    return;
}

// Generate fresh data...
set_transient($cache_key, $data, 300); // 5 minutes
```

Cache is cleared whenever:
- An expense is added or updated
- Payroll is generated
- A payment is recorded
- An expense is approved or rejected
```
