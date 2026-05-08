```markdown
# Staff Management (Payroll)

The Staff Management module within Payroll & Expenses handles all employee records, contracts, salary structures, attendance, and loans — integrated directly with payroll processing.

---

## Accessing Staff Management

Navigate to **Payroll & Expenses → Staff**.

---

## Staff List View

| Column | Description |
|--------|-------------|
| Photo | Profile photo thumbnail |
| Code | Auto-generated employee code |
| Name | Full name |
| Email | Email address |
| Phone | Contact number |
| Role | Staff role with label |
| Campus | Primary campus |
| Status | Employment status badge |
| Actions | Edit button |

---

## Adding a Staff Member

Click **+ Add Staff Member** to open the comprehensive Add Staff modal with 7 sections.

### Section 1: Personal Information

| Field | Required | Description |
|-------|----------|-------------|
| Full Name | Yes | Full legal name |
| Email Address | Yes | Used for login if applicable |
| Phone Number | No | Contact number |
| Join Date | Yes | Defaults to today |
| Role | Yes | 8 predefined roles |
| Primary Campus | Yes | Campus assignment |
| Address | No | Full address |

### Section 2: Profile Photo

WordPress media library integration for staff photos.

### Section 3: Salary Information

Dynamic fields based on contract type selection:

| Contract Type | Fields |
|---------------|--------|
| **Monthly Fixed** | Base Salary (e.g., 50,000) |
| **Hourly** | Hourly Rate + Standard Hours/Month (default 160) |
| **Per Lecture** | Rate Per Lecture |

### Section 4: Bank Details

| Field | Description |
|-------|-------------|
| Bank Name | For salary transfer |
| Account Number | Bank account |
| IFSC / Branch Code | Branch identifier |
| Tax ID / CNIC | Tax identification |

### Section 5: Emergency Contact

| Field | Description |
|-------|-------------|
| Contact Name | Emergency person |
| Contact Phone | Emergency number |

### Section 6: Notes

Free-text additional information.

### Section 7: Form Actions

- **Add Staff Member** — Submit
- **Cancel** — Close modal

---

## Auto-Account Creation

When adding a staff member with a teaching or administrative role, InstitutionKit automatically:

1. Creates a WordPress user account
2. Assigns the appropriate role (`teacher` or `campus_admin`)
3. Generates a secure random password
4. Sends a welcome email with login credentials
5. Links the WordPress user ID to the staff record

### Roles Requiring Accounts

| Staff Role | WordPress Role |
|------------|---------------|
| `teacher_permanent` | `teacher` |
| `teacher_visiting` | `teacher` |
| `campus_head` | `campus_admin` |
| `admin_roving` | `campus_admin` |

### Non-Account Roles

These roles do not get WordPress accounts:
- `office_staff`
- `maintenance`
- `security`
- `other`

---

## Welcome Email

```text
Subject: [School Name] Your Staff Account

Dear [Full Name],

Your staff account has been created at [School Name].

You can log in to the portal using the following credentials:

Username: john.doe
Password: aB3xK9mP2qR7

Login URL: https://school.edu/teachers-portal.php

Please change your password after logging in.

Thank you,
[School Name] Administration
```

---

## Editing a Staff Member

Click **Edit** on any staff row to open the Edit modal with additional features:

### Employment Status Management

| Status | Description |
|--------|-------------|
| Active | Currently employed |
| On Leave | Temporarily absent |
| Terminated | Employment ended |
| Retired | Retired from service |

### Termination Fields

When status is changed to "Terminated":

| Field | Description |
|-------|-------------|
| Termination Date | Date of termination |
| Termination Reason | Reason (appended to notes) |

### Salary Components

Flexible earnings and deductions beyond base salary:

| Type | Example |
|------|---------|
| Earnings | House Rent Allowance, Medical Allowance, Transport Allowance |
| Deductions | Provident Fund, Professional Tax, Insurance |

### Staff Loans

Complete loan management within the edit modal:

| Feature | Description |
|---------|-------------|
| View Loans | Table of all active and paid loans |
| Add Loan | Form with principal, tenure, interest rate |
| Auto-calculation | EMI calculated on entry |
| Payroll Integration | Automatic deduction during payroll |

---

## Employee Code Generation

Employee codes are auto-generated in the format `EMP{YY}{XXXX}`:

```php
private function generate_employee_code() {
    $year = date('y');
    $last_code = $wpdb->get_var(
        "SELECT MAX(employee_code) FROM {$table} WHERE employee_code LIKE 'EMP{$year}%'"
    );
    
    if ($last_code) {
        $last_number = (int)substr($last_code, 5);
        $new_number = $last_number + 1;
    } else {
        $new_number = 1;
    }
    
    return sprintf('EMP%s%04d', $year, $new_number);
}
```

Example codes: `EMP260001`, `EMP260042`

---

## Staff Database

```sql
CREATE TABLE wp_institutionkit_staff (
    staff_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT(20) UNSIGNED,
    employee_code VARCHAR(50) UNIQUE NOT NULL,
    full_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    phone VARCHAR(50),
    role VARCHAR(30) NOT NULL,
    primary_campus_id BIGINT(20) UNSIGNED NOT NULL,
    assigned_campuses LONGTEXT,
    bank_name VARCHAR(255),
    bank_account_number VARCHAR(255),
    bank_ifsc VARCHAR(20),
    tax_id VARCHAR(50),
    contract_type VARCHAR(20) DEFAULT 'monthly_fixed',
    base_salary DECIMAL(12,2) DEFAULT 0.00,
    hourly_rate DECIMAL(10,2),
    lecture_rate DECIMAL(10,2),
    standard_hours_per_month INT DEFAULT 160,
    standard_lectures_per_month INT,
    join_date DATE NOT NULL,
    contract_end_date DATE,
    termination_date DATE,
    employment_status VARCHAR(20) DEFAULT 'active',
    emergency_contact_name VARCHAR(255),
    emergency_contact_phone VARCHAR(50),
    address TEXT,
    photo_id BIGINT(20) UNSIGNED,
    created_by BIGINT(20) UNSIGNED NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## Programmatic Usage

### Add Staff

```php
global $wpdb;
$staff_id = $wpdb->insert("{$wpdb->prefix}institutionkit_staff", [
    'employee_code'     => 'EMP260042',
    'full_name'         => 'Jane Smith',
    'email'             => 'jane@school.edu',
    'role'              => 'teacher_permanent',
    'primary_campus_id' => 3,
    'contract_type'     => 'monthly_fixed',
    'base_salary'       => 50000,
    'join_date'         => '2026-01-15',
    'employment_status' => 'active',
    'created_by'        => get_current_user_id(),
]);
```

### Get Staff by ID

```php
$staff = $wpdb->get_row($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_staff WHERE staff_id = %d",
    $staff_id
));
```

### Update Staff Status

```php
$wpdb->update(
    "{$wpdb->prefix}institutionkit_staff",
    [
        'employment_status' => 'terminated',
        'termination_date'  => '2026-06-30',
        'updated_at'        => current_time('mysql'),
    ],
    ['staff_id' => $staff_id]
);
```

### Get Staff by Campus

```php
$staff = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_staff 
     WHERE primary_campus_id = %d AND employment_status = 'active'
     ORDER BY full_name ASC",
    $campus_id
));
```
