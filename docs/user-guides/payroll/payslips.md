```markdown
# Payslips

The Payslips module generates detailed salary statements for each staff member — including earnings breakdown, deductions, attendance summary, tax calculation, and loan repayments. Payslips are available as interactive HTML views and downloadable PDFs.

---

## Accessing Payslips

Payslips are accessed from the Payroll page:

1. Navigate to **Payroll & Expenses → Payroll**
2. Load payroll for a campus and month
3. Click **Payslip** on any staff row

---

## Payslip Layout

### Header

- Institution name
- "Payslip" title
- Pay period (month and year)

### Employee Information

| Field | Source |
|-------|--------|
| Employee Name | `full_name` from staff table |
| Employee Code | `employee_code` |
| Role | Role label |
| Campus | Primary campus name |
| Contract Type | Monthly Fixed / Hourly / Per Lecture |
| Bank | Bank name (if provided) |

### Attendance Summary

Five attendance badges:

| Badge | Count |
|-------|-------|
| Present | Days marked present |
| Absent | Days marked absent |
| Half Days | Days marked half day |
| Leave | Days on approved leave |
| Working Days | Total working days in month |
| Lectures | Total lectures delivered (if applicable) |

### Earnings Section

Table of all earnings with amounts:

| Description | Amount |
|-------------|--------|
| Basic Salary | 50,000 |
| House Rent Allowance | 10,000 |
| Medical Allowance | 5,000 |
| **Gross Pay** | **65,000** |

### Deductions Section

Table of all deductions:

| Description | Amount |
|-------------|--------|
| Absent Days (2 days) | 3,704 |
| Income Tax | 7,500 |
| Loan Installment (#1) | 4,375 |
| Provident Fund | 3,250 |
| **Total Deductions** | **18,829** |

### Net Payable

```
┌─────────────────────────────────────────┐
│  Net Payable Amount        Rs 46,171    │
└─────────────────────────────────────────┘
```

### Footer

- "This is a computer-generated payslip"
- Generation date and time
- Institution copyright

---

## Payslip Actions

| Button | Action |
|--------|--------|
| 🖨️ **Print Payslip** | Opens browser print dialog |
| 📄 **Download PDF** | Downloads formatted PDF |
| ← **Back to Payroll** | Returns to payroll list |

---

## PDF Payslip Generation

### Technology

PDFs are generated using Dompdf with the DejaVu Sans font.

### PDF Content

The PDF includes all the same sections as the HTML view but optimized for A4 paper:

- Header with institution branding
- Employee information grid
- Attendance badges
- Earnings table
- Deductions table
- Net payable highlight
- Footer with generation date
- "PAYSLIP" watermark

### PDF Generation Code

```php
private function stream_pdf($html, $filename) {
    $pdf_content = $this->generate_pdf($html, $filename);
    
    // Clear all output buffers
    while (ob_get_level()) {
        ob_end_clean();
    }
    
    // Send PDF headers
    header('Content-Type: application/pdf');
    header('Content-Disposition: attachment; filename="' . $filename . '.pdf"');
    header('Content-Length: ' . strlen($pdf_content));
    
    echo $pdf_content;
    exit;
}
```

---

## How Payslip Data is Calculated

### Step 1: Attendance Collection

```php
$attendance = $wpdb->get_results($wpdb->prepare(
    "SELECT status, check_in, check_out, lectures_count 
     FROM {$wpdb->prefix}institutionkit_staff_attendance 
     WHERE staff_id = %d AND attendance_date BETWEEN %s AND %s",
    $staff_id, $month_start, $month_end
));
```

### Step 2: Base Salary Calculation

Based on contract type:

| Contract | Formula |
|----------|---------|
| Monthly Fixed | Base Salary - Absent Deduction |
| Hourly | Hours Worked × Hourly Rate |
| Per Lecture | Lectures × Lecture Rate |

### Step 3: Apply Salary Components

All active salary components are added:

```php
$components = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_staff_salary_components 
     WHERE staff_id = %d AND is_active = 1",
    $staff_id
));

foreach ($components as $comp) {
    if ($comp['component_type'] === 'earnings') {
        $earnings[] = ['label' => $comp['label'], 'amount' => $comp['amount']];
    } else {
        $deductions[] = ['label' => $comp['label'], 'amount' => $comp['amount']];
    }
}
```

### Step 4: Attendance Deductions

For monthly fixed staff with absences:

```php
if ($contract_type === 'monthly_fixed' && $absent_days > 0) {
    $per_day = $base_salary / $working_days;
    $deduction = $per_day * $absent_days;
    // First absence is free; deduct from second onward
}
```

### Step 5: Loan Deductions

Active loans are deducted from the payslip:

```php
$loans = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_staff_loans 
     WHERE staff_id = %d AND status = 'active' AND remaining_balance > 0",
    $staff_id
));

foreach ($loans as $loan) {
    $deduct_amount = min($loan->monthly_deduction, $loan->remaining_balance);
    $deductions[] = ['label' => 'Loan Installment', 'amount' => $deduct_amount];
}
```

### Step 6: Income Tax Calculation

```php
private function calculate_income_tax($taxable_income) {
    if ($taxable_income <= 50000) return 0;
    if ($taxable_income <= 100000) return ($taxable_income - 50000) * 0.025;
    if ($taxable_income <= 166667) return 1250 + ($taxable_income - 100000) * 0.125;
    if ($taxable_income <= 233333) return 9583 + ($taxable_income - 166667) * 0.175;
    if ($taxable_income <= 300000) return 21250 + ($taxable_income - 233333) * 0.225;
    return 36250 + ($taxable_income - 300000) * 0.275;
}
```

### Step 7: Final Calculation

```php
$gross_pay = array_sum(array_column($earnings, 'amount'));
$total_deductions = array_sum(array_column($deductions, 'amount'));
$net_pay = max(0, $gross_pay - $total_deductions);
```

---

## Currency Formatting

Amounts on payslips are rounded to the nearest 10:

```php
private function format_payslip_amount($amount) {
    $rounded = round(floatval($amount) / 10) * 10;
    return number_format($rounded, 0);
}
```

---

## Print Styles

Payslips include `@media print` CSS for clean printing:

- Hides all control buttons
- Removes admin UI elements
- Sets white background
- Optimizes for A4 paper
- Includes page margins

---

## AJAX Payslip Retrieval

```php
public function ik_get_payslip() {
    check_ajax_referer('ik_payroll_nonce', 'nonce');
    
    $staff_id = intval($_POST['staff_id']);
    $month = sanitize_text_field($_POST['month']);
    
    $payslip = $this->get_payslip($staff_id, $month);
    
    if (isset($payslip['error'])) {
        wp_send_json_error(['message' => $payslip['error']]);
    }
    
    wp_send_json_success($payslip);
}
```

---

## Programmatic Usage

### Get Payslip Data

```php
$payslip = $payslips_trait->get_payslip($staff_id, '2026-06');

// Returns:
[
    'staff'            => (object) staff data,
    'month'            => '2026-06',
    'month_name'       => 'June 2026',
    'attendance'       => ['present' => 22, 'absent' => 2, ...],
    'earnings'         => [['label' => 'Basic Salary', 'amount' => 50000], ...],
    'deductions'       => [['label' => 'Income Tax', 'amount' => 7500], ...],
    'gross_pay'        => 65000,
    'total_deductions' => 18829,
    'net_pay'          => 46171,
]
```

### Download Payslip PDF

```php
$payslips_trait->download_payslip_pdf();
// Reads staff_id and month from $_GET parameters
// Streams PDF to browser
```

### Get Working Days in Month

```php
$working_days = $payslips_trait->get_working_days_in_month('2026-06');
// Returns number of days excluding Sundays
```
