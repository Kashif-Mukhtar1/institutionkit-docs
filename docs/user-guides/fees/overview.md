```markdown
# Fee Management Overview

The Fee Management module handles the complete financial lifecycle — from defining fee types and structures to generating invoices, tracking payments, and sending reminders.

---

## Accessing Fee Management

Navigate to **InstitutionKit → Fee Management** in the admin menu.

---

## Dashboard Cards

The Fee Management landing page presents 4 action cards:

| Card | Destination | Description |
|------|-------------|-------------|
| 🏷️ **Fee Types** | Fee types list | Define different fee categories |
| 📐 **Fee Structures** | Fee structures list | Create fee structure groups |
| 📌 **Assign Fees** | Fee assignment | Assign fee structures to students |
| 🧾 **Invoices** | Invoice management | Generate and manage invoices |

---

## Fee Management Workflow

```
1. Create Fee Types
   └── Define categories: Tuition, Transport, Library, etc.

2. Create Fee Structures
   └── Group fee types with amounts per class level

3. Assign Fees to Students
   └── Apply structures to individual students or entire classes

4. Generate Invoices
   └── Auto-generate monthly or create manual invoices

5. Track Payments
   └── Record payments, track balances, send reminders
```

---

## Fee Types

Fee types are the basic building blocks — categories of fees that your institution charges.

### Default Fee Types

Common fee types to create:

| Fee Type | Typical Use |
|----------|-------------|
| Tuition Fee | Monthly or term tuition |
| Admission Fee | One-time enrollment fee |
| Exam Fee | Per-term examination fee |
| Transport Fee | Monthly transport charges |
| Library Fee | Library access fee |
| Sports Fee | Sports and activities fee |
| Computer Lab Fee | Technology usage fee |
| Development Fee | Infrastructure development |

### Fee Type Properties

| Property | Description |
|----------|-------------|
| **Name** | Unique name (e.g., "Tuition Fee") |
| **Global** | Available across all campuses |

---

## Fee Structures

Fee structures group fee types with specific amounts, typically organized by class level.

### Example Structure

**Structure Name:** "Primary Monthly Fees"

| Fee Type | Amount |
|----------|--------|
| Tuition Fee | 3,500 |
| Transport Fee | 1,000 |
| Library Fee | 200 |
| **Total** | **4,700** |

### Structure Organization

Structures can be organized by:

- **Class level** — Different structures for Grade 1 vs Grade 5
- **Campus** — Different amounts per campus
- **Academic year** — Annual fee adjustments

---

## Fee Assignment

Once structures are created, they must be assigned to students.

### Assignment Methods

| Method | Description |
|--------|-------------|
| **Individual** | Assign fees to a single student |
| **By Class** | Assign fees to all students in a class |
| **By Campus** | Assign fees to all students in a campus |
| **Bulk Select** | Select multiple students manually |

### Student Fee Records

When fees are assigned, individual records are created in `institutionkit_student_fees`:

```sql
CREATE TABLE wp_institutionkit_student_fees (
    student_fee_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    student_id BIGINT(20) UNSIGNED NOT NULL,
    fee_type_id BIGINT(20) UNSIGNED NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    start_date DATE,
    end_date DATE,
    notes VARCHAR(255),
    KEY student_id (student_id),
    KEY fee_type_id (fee_type_id)
);
```

### Date-Range Fees

Fees can have optional start and end dates for:

- **Time-limited fees** — e.g., Exam fee for one term only
- **Pro-rated fees** — e.g., Mid-year admission with reduced tuition
- **Seasonal fees** — e.g., Summer camp fee

---

## Invoice Generation

Invoices transform assigned fees into billable documents.

### Auto Invoice Generation

InstitutionKit includes automatic monthly invoice generation:

```php
// Cron job: runs on the first day of each month at 06:00
wp_schedule_event(
    strtotime('first day of next month 06:00:00'),
    'monthly',
    'ik_monthly_invoice_generation_event'
);
```

### Manual Invoice Generation

Invoices can also be generated manually:

1. Select students or classes
2. Choose the billing period
3. Review the invoice preview
4. Click **Generate Invoices**

### Invoice Status

| Status | Description |
|--------|-------------|
| **Unpaid** | No payment received |
| **Partial** | Some payment received, balance remaining |
| **Paid** | Full payment received |

---

## Payment Tracking

### Recording Payments

Payments are recorded against invoices:

| Field | Description |
|-------|-------------|
| Invoice | The invoice being paid |
| Amount | Payment amount |
| Payment Date | Date of payment |
| Payment Method | Cash, Bank Transfer, Cheque, Online, Card |
| Notes | Optional reference or transaction ID |

### Payment Validation

The system prevents overpayment:

```php
if (($payment_amount - $balance_due) > 0.001) {
    wp_send_json_error([
        'message' => 'Payment amount cannot exceed the balance due.'
    ]);
}
```

### Transaction History

Every payment is logged in `institutionkit_transactions` with:

- Timestamp
- Amount
- Payment method
- Recording user
- Optional notes

---

## Fee Reminders

### Manual Reminders

From the dashboard, click **Send Fee Reminders** to email all parents with overdue invoices.

### Reminder Email Content

```
Subject: [School Name] Fee Payment Reminder - Invoice #42

Dear Parent/Guardian,

This is a gentle reminder regarding the following fee invoice:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Student Name: Ahmed Khan
Invoice #: 42
Due Date: June 5, 2026
Total Amount: $4,700.00
Amount Paid: $0.00
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Amount Due: $4,700.00

Please log in to the Parent Portal to view and pay this invoice.

Thank you,
Springfield Elementary Accounts Department
```

### Parent Email Lookup

The system looks up parent emails in this order:

1. `_ik_email` post meta on the student
2. `_ik_parent_email` post meta on the student
3. Parent user email from `institutionkit_parent_child` link
4. WordPress user email from parent account

---

## Data Relationships

```
institutionkit_fee_types (1)
    │
    ├── institutionkit_fee_structure_items (N)
    │   └── Links types to structures with amounts
    │
    ├── institutionkit_student_fees (N)
    │   └── Individual student fee assignments
    │
    └── institutionkit_invoice_items (N)
        └── Line items on invoices

institutionkit_fee_structures (1)
    │
    └── institutionkit_fee_structure_items (N)

institutionkit_invoices (1)
    │
    ├── institutionkit_invoice_items (N)
    ├── institutionkit_transactions (N)
    └── institutionkit_email_log (N)
```

---

## Quick Reference

### Get Outstanding Balance

```php
global $wpdb;
$outstanding = $wpdb->get_var($wpdb->prepare(
    "SELECT SUM(total_amount - amount_paid) 
     FROM {$wpdb->prefix}institutionkit_invoices 
     WHERE student_id = %d AND status != 'paid'",
    $student_id
));
```

### Get Monthly Collections

```php
global $wpdb;
$collections = $wpdb->get_var($wpdb->prepare(
    "SELECT SUM(amount) 
     FROM {$wpdb->prefix}institutionkit_transactions 
     WHERE payment_date BETWEEN %s AND %s
       AND campus_id = %d",
    $month_start, $month_end, $campus_id
));
```

### Get Overdue Invoices

```php
global $wpdb;
$overdue = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_invoices 
     WHERE due_date < %s AND status != 'paid'
       AND campus_id = %d
     ORDER BY due_date ASC",
    current_time('Y-m-d'), $campus_id
));
```
