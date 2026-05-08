```markdown
# Staff Management

The Staff Management module handles the complete employee lifecycle — from adding new staff members to managing contracts, salary structures, loans, and employment status.

---

## Accessing Staff Management

Navigate to **InstitutionKit → Teacher Management → All Staff** or **Payroll & Expenses → Staff**.

---

## Staff List View

The main staff page displays all employees in a table with key information:

| Column | Description |
|--------|-------------|
| Photo | Profile photo thumbnail |
| Code | Auto-generated employee code (e.g., `EMP260001`) |
| Name | Full name |
| Email | Email address |
| Phone | Contact number |
| Role | Staff role with label |
| Campus | Primary campus assignment |
| Status | Employment status badge |
| Actions | Edit button |

---

## Adding a New Staff Member

Click **+ Add Staff Member** to open the Add Staff modal.

### Form Sections

The Add Staff form is organized into 7 professional sections:

#### Section 1: Personal Information

| Field | Required | Description |
|-------|----------|-------------|
| Full Name | Yes | Full legal name |
| Email Address | Yes | Used for login credentials if applicable |
| Phone Number | No | Contact number |
| Join Date | Yes | Employment start date, defaults to today |
| Role | Yes | Select from 8 predefined roles |
| Primary Campus | Yes | The campus where they primarily work |
| Address | No | Full physical address |

#### Section 2: Profile Photo

| Feature | Description |
|---------|-------------|
| Upload | WordPress media library integration |
| Preview | Shows selected photo immediately |
| Remove | Clear the photo selection |

#### Section 3: Salary Information

| Field | Required | Description |
|-------|----------|-------------|
| Contract Type | Yes | Monthly Fixed / Hourly / Per Lecture |

**Dynamic fields based on contract type:**

=== "Monthly Fixed"
    | Field | Description |
    |-------|-------------|
    | Monthly Salary | Fixed monthly amount (e.g., 50,000) |

=== "Hourly"
    | Field | Description |
    |-------|-------------|
    | Hourly Rate | Rate per hour (e.g., 500) |
    | Hours/Month | Standard working hours (default: 160) |

=== "Per Lecture"
    | Field | Description |
    |-------|-------------|
    | Rate Per Lecture | Rate per lecture delivered (e.g., 1,000) |

#### Section 4: Bank Details

| Field | Description |
|-------|-------------|
| Bank Name | Bank for salary transfer |
| Account Number | Bank account number |
| IFSC / Branch Code | Branch identifier |
| Tax ID / CNIC | Tax identification number |

#### Section 5: Emergency Contact

| Field | Description |
|-------|-------------|
| Contact Name | Emergency contact person |
| Contact Phone | Emergency contact number |

#### Section 6: Notes

Free-text field for additional information.

---

## Editing a Staff Member

Click the **Edit** button on any staff row to open the Edit Staff modal.

### Additional Edit Fields

The Edit modal includes all Add fields plus:

| Field | Description |
|-------|-------------|
| **Employee Code** | Read-only display |
| **Employment Status** | Active / On Leave / Terminated / Retired |
| **Termination Date** | Shown when status is "Terminated" |
| **Termination Reason** | Shown when status is "Terminated" |
| **Salary Components** | Flexible earnings and deductions |
| **Loans** | Staff loan management |

---

## Employment Status Management

### Status Options

| Status | Badge Color | Effect |
|--------|-------------|--------|
| **Active** | Green | Fully operational, appears in all lists |
| **On Leave** | Yellow | Temporarily absent, still in system |
| **Terminated** | Red | No longer employed, hidden from active lists |
| **Retired** | Gray | Retired, historical data preserved |

### Termination Workflow

When terminating a staff member:

1. Select **Terminated** from the status dropdown
2. Enter the **Termination Date**
3. Enter the **Termination Reason** (appended to notes)
4. Click **Update Staff**

The termination reason is stored in the `notes` field prefixed with "Termination Reason:".

---

## Salary Components

Each staff member can have flexible salary components beyond their base pay.

### Accessing Salary Components

Edit a staff member and scroll to the **Salary Structure** section.

### Component Types

| Type | Description | Example |
|------|-------------|---------|
| **Earnings** | Additional pay | House Rent Allowance, Medical Allowance, Transport Allowance |
| **Deductions** | Regular deductions | Provident Fund, Professional Tax, Insurance |

### Adding a Component

| Field | Description |
|-------|-------------|
| Type | Earning or Deduction |
| Label | Description (e.g., "House Rent Allowance") |
| Amount | Fixed amount |
| Taxable | Whether this component is subject to income tax |

Components are stored in `institutionkit_staff_salary_components` and automatically included in payroll calculations.

---

## Staff Loans

InstitutionKit includes a complete loan management system for staff.

### Loan Features

- Interest-bearing loans with configurable rates
- Auto-calculated monthly installments (EMI)
- Automatic deduction from payroll
- Loan status tracking (Active / Paid / Written Off)

### Adding a Loan

| Field | Description |
|-------|-------------|
| Loan Date | Date of loan issuance |
| Principal | Loan amount (e.g., 50,000) |
| Tenure | Repayment period in months (e.g., 12) |
| Interest Rate | Annual interest rate percentage |

### Loan Calculation

```
Total Interest = Principal × (Rate / 100) × (Tenure / 12)
Total Payable = Principal + Total Interest
Monthly EMI = Total Payable ÷ Tenure
```

**Example:**
- Principal: 50,000
- Rate: 5%
- Tenure: 12 months
- Interest: 50,000 × 0.05 × 1 = 2,500
- Total Payable: 52,500
- Monthly EMI: 52,500 ÷ 12 = 4,375

### Loan Deduction Process

When payroll is generated:

1. System identifies active loans for each staff member
2. Monthly EMI is deducted from the payslip
3. Interest and principal portions are calculated
4. Loan remaining balance is updated
5. When balance reaches zero, loan status changes to "Paid"

---

## Staff List Filters

The staff table supports filtering:

| Filter | Description |
|--------|-------------|
| Campus | Filter by primary campus |
| Role | Filter by staff role |
| Status | Filter by employment status |
| Search | Search by name or employee code |

---

## Export Staff Data

Click **Export** to download staff data as CSV. The export includes all staff fields.

---

## Programmatic Staff Operations

### Add Staff via Code

```php
global $wpdb;
$staff_id = $wpdb->insert(
    "{$wpdb->prefix}institutionkit_staff",
    [
        'employee_code'    => 'EMP260042',
        'full_name'        => 'Jane Smith',
        'email'            => 'jane@school.edu',
        'phone'            => '+1234567890',
        'role'             => 'teacher_permanent',
        'primary_campus_id' => 3,
        'contract_type'    => 'monthly_fixed',
        'base_salary'      => 50000,
        'join_date'        => '2026-01-15',
        'employment_status' => 'active',
        'created_by'       => get_current_user_id(),
    ]
);
```

### Update Staff Status

```php
global $wpdb;
$wpdb->update(
    "{$wpdb->prefix}institutionkit_staff",
    [
        'employment_status' => 'terminated',
        'termination_date'  => '2026-06-30',
        'notes'             => 'Termination Reason: End of contract',
        'updated_at'        => current_time('mysql'),
    ],
    ['staff_id' => $staff_id]
);
```

### Add Salary Component

```php
global $wpdb;
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_staff_salary_components",
    [
        'staff_id'       => $staff_id,
        'component_type' => 'earnings',
        'label'          => 'House Rent Allowance',
        'amount'         => 10000,
        'is_taxable'     => 1,
    ]
);
```

### Add Staff Loan

```php
global $wpdb;
$principal = 50000;
$rate = 5;
$tenure = 12;
$total_interest = $principal * ($rate / 100) * ($tenure / 12);
$total_payable = $principal + $total_interest;
$monthly = round($total_payable / $tenure, 2);

$wpdb->insert(
    "{$wpdb->prefix}institutionkit_staff_loans",
    [
        'staff_id'             => $staff_id,
        'loan_date'            => '2026-06-01',
        'principal_amount'     => $principal,
        'annual_interest_rate' => $rate,
        'tenure_months'        => $tenure,
        'monthly_deduction'    => $monthly,
        'remaining_balance'    => $total_payable,
    ]
);
```
```
