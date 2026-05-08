```markdown
# Payroll Processing

The Payroll module generates monthly payroll for all staff based on their contract type, attendance records, salary components, and active loans — producing detailed payslips with automatic tax calculation.

---

## Accessing Payroll

Navigate to **Payroll & Expenses → Payroll**.

---

## Payroll Page Layout

### Filter Section

| Filter | Description |
|--------|-------------|
| Campus | Select the campus |
| Month | Select payroll month |

Click **Load Payroll** to view or generate records.

### Action Buttons

| Button | Action |
|--------|--------|
| ⚙️ **Generate Payroll** | Create payroll records for the selected campus/month |
| 🗑️ **Delete Batch** | Remove all payroll records for this month |
| ✅ **Mark All Paid** | Set all records to paid status |
| 📊 **Export CSV** | Download payroll data as CSV |

---

## Generating Payroll

### What Happens During Generation

1. System finds all active staff for the selected campus
2. For each staff member, it:
    - Checks if payroll already exists (skips if so)
    - Retrieves monthly attendance summary
    - Calculates pay based on contract type
    - Applies salary components (earnings + deductions)
    - Deducts active loan installments
    - Calculates income tax
    - Computes net pay
3. Creates payroll records in the database
4. Auto-processes loan installment deductions

### Contract Type Calculation

#### Monthly Fixed Salary

```
Per Day Salary = Base Salary ÷ Working Days in Month
Absent Deduction = (Absent Days - 1) × Per Day Salary
Gross Pay = Base Salary - Absent Deduction
Perfect Attendance Bonus = +1 day salary (0 absences)
```

**Example:**
- Base Salary: 50,000
- Working Days: 27
- Per Day: 1,852
- Absent: 2 days
- Deduction: 1 × 1,852 = 1,852
- Gross Pay: 48,148

#### Hourly Rate

```
Hours Worked = Sum of hours from attendance records
Gross Pay = Hours Worked × Hourly Rate
```

**Example:**
- Hourly Rate: 500
- Hours Worked: 160
- Gross Pay: 80,000

#### Per Lecture Rate

```
Total Lectures = Sum of lectures from attendance records
Gross Pay = Total Lectures × Lecture Rate
```

**Example:**
- Lecture Rate: 1,000
- Lectures Delivered: 45
- Gross Pay: 45,000

---

## Payroll Table

After loading or generating, the payroll table shows:

| Column | Description |
|--------|-------------|
| Code | Employee code |
| Staff Name | Full name |
| Role | Role label |
| Campus | Campus name |
| Gross Pay | Total earnings |
| Deductions | Total deductions |
| Net Pay | Final payable amount |
| Status | Pending or Paid |
| Actions | Payslip / Mark Paid / Unmark / Delete |

---

## Summary Cards

Above the table, four summary cards display:

| Card | Calculation |
|------|-------------|
| 💰 Total Gross | Sum of all gross pay |
| ➖ Total Deductions | Sum of all deductions |
| 💵 Net Payable | Sum of all net pay |
| ✅ Paid | Sum of net pay where status = paid |

---

## Payroll Actions

### View Payslip

Click **Payslip** to view the detailed payslip for a staff member.

### Mark as Paid

Click **Paid** to mark a single record as paid.

### Unmark

Click **Unmark** to revert a paid record back to pending.

### Delete Single

Click **🗑️** to delete a single payroll record. Associated loan installments are also deleted.

### Mark All Paid

Click **Mark All Paid** to set all records in the current view to paid status.

### Export CSV

Click **Export CSV** to download the payroll table as a CSV file:

```javascript
function exportPayrollCSV() {
    var rows = document.querySelectorAll('#payroll-table tr');
    var csv = [];
    rows.forEach(function(row) {
        var cols = row.querySelectorAll('th, td');
        var rowData = [];
        cols.forEach(function(col, index) {
            if (index < cols.length - 1) {
                rowData.push('"' + col.innerText.replace(/,/g, '').trim() + '"');
            }
        });
        csv.push(rowData.join(','));
    });
    // Download as CSV file
}
```

---

## Loan Installment Processing

When payroll is generated, active loans are automatically deducted:

```php
private function process_loan_installments($payroll_id, $staff_id, $month, $payslip) {
    $loans = $wpdb->get_results($wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}institutionkit_staff_loans 
         WHERE staff_id = %d AND status = 'active' AND remaining_balance > 0",
        $staff_id
    ));
    
    foreach ($loans as $loan) {
        $monthly_amount = floatval($loan->monthly_deduction);
        $remaining = floatval($loan->remaining_balance);
        $deduct_amount = min($monthly_amount, $remaining);
        
        // Calculate interest portion
        $monthly_interest = ($remaining * $loan->annual_interest_rate / 100) / 12;
        $interest_portion = round(min($monthly_interest, $deduct_amount), 2);
        $principal_portion = $deduct_amount - $interest_portion;
        
        // Record installment
        $wpdb->insert("{$wpdb->prefix}institutionkit_loan_installments", [
            'loan_id'        => $loan->loan_id,
            'payroll_id'     => $payroll_id,
            'amount'         => $deduct_amount,
            'principal_paid' => $principal_portion,
            'interest_paid'  => $interest_portion,
        ]);
        
        // Update remaining balance
        $new_balance = $remaining - $deduct_amount;
        $new_status = $new_balance <= 0 ? 'paid' : 'active';
        
        $wpdb->update("{$wpdb->prefix}institutionkit_staff_loans", [
            'remaining_balance' => $new_balance,
            'status' => $new_status,
        ], ['loan_id' => $loan->loan_id]);
    }
}
```

---

## Income Tax Calculation

Monthly income tax is calculated progressively:

| Monthly Taxable Income | Tax Rate |
|------------------------|----------|
| Up to 50,000 | 0% |
| 50,001 – 100,000 | 2.5% of amount above 50,000 |
| 100,001 – 166,667 | 1,250 + 12.5% above 100,000 |
| 166,668 – 233,333 | 9,583 + 17.5% above 166,667 |
| 233,334 – 300,000 | 21,250 + 22.5% above 233,333 |
| Above 300,000 | 36,250 + 27.5% above 300,000 |

**Example:**
- Taxable Income: 150,000/month
- Tax: 1,250 + (50,000 × 12.5%) = 1,250 + 6,250 = 7,500

---

## Payroll Database

```sql
CREATE TABLE wp_institutionkit_payroll (
    payroll_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    staff_id BIGINT(20) UNSIGNED NOT NULL,
    campus_id BIGINT(20) UNSIGNED NOT NULL,
    payroll_month DATE NOT NULL,
    gross_pay DECIMAL(12,2) DEFAULT 0.00,
    total_deductions DECIMAL(12,2) DEFAULT 0.00,
    net_pay DECIMAL(12,2) DEFAULT 0.00,
    earnings_json LONGTEXT,
    deductions_json LONGTEXT,
    attendance_json LONGTEXT,
    status VARCHAR(20) DEFAULT 'pending',
    created_by BIGINT(20) UNSIGNED NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY unique_monthly_payroll (staff_id, campus_id, payroll_month)
);
```

---

## Programmatic Usage

### Generate Payroll

```php
global $wpdb;
// Via AJAX
$.post(ajaxurl, {
    action: 'ik_generate_payroll',
    nonce: ik_ajax.nonce,
    campus_id: 3,
    month: '2026-06'
});

// Via PHP
$generated = $payroll_module->generate_monthly_payroll($campus_id, '2026-06');
```

### Get Payroll Records

```php
$records = $db->get_payroll_records([
    'campus_id'     => 3,
    'payroll_month' => '2026-06-01',
]);
```

### Delete Payroll Batch

```php
$deleted = $payroll_module->delete_payroll_batch($campus_id, '2026-06');
```
