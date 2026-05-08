```markdown
# Invoices

The Invoices module generates billable invoices from assigned student fees, tracks payments, and manages the complete billing lifecycle.

---

## Accessing Invoices

| Interface | Path | Who Uses It |
|-----------|------|-------------|
| **Admin Panel** | InstitutionKit → Fee Management → Invoices | Admins, Accountants |
| **Frontend** | Page with `[ik_invoices]` shortcode | Staff with campus access |

---

## Invoice List View

### Admin Panel

| Column | Description |
|--------|-------------|
| ID | Invoice number |
| Title | Invoice title/description |
| Student | Student name |
| Roll # | Student roll number |
| Guardian Contact | Emergency contact number |
| Total | Total invoice amount |
| Paid | Amount received |
| Balance | Outstanding amount |
| Status | Unpaid / Partial / Paid |
| Date | Creation date |
| Actions | View / Delete |

### Filters

| Filter | Description |
|--------|-------------|
| Class | Filter by student class |
| Status | All / Unpaid / Partial |
| Search | Search by student name or roll number |

---

## Generating Invoices

### Automatic Monthly Generation

InstitutionKit automatically generates invoices on the first day of each month:

```php
// Scheduled cron event
wp_schedule_event(
    strtotime('first day of next month 06:00:00'),
    'monthly',
    'ik_monthly_invoice_generation_event'
);
```

The auto-generation process:

1. Finds all students with assigned fees
2. Creates invoice records for the current month
3. Populates invoice items from assigned fee types
4. Sets due date based on configuration

### Manual Generation

Administrators can manually generate invoices:

1. Go to **Fee Management → Invoices**
2. Select class or campus filters
3. Click **Generate Invoices**
4. Review the preview
5. Confirm generation

---

## Single Invoice View

Click **View** on any invoice to see the detailed view.

### Invoice Details

The single invoice view displays:

```
┌─────────────────────────────────────────────┐
│  SCHOOL LOGO          INVOICE               │
│                                              │
│  Student: Ahmed Khan                         │
│  Class: Grade 5                              │
│  Roll #: 42                                  │
│  Guardian: Muhammad Khan                     │
│                                              │
│  Invoice #: 127            Date: June 1, 2026│
│  Status: Unpaid                              │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │ Fee Breakdown                        │    │
│  │ Tuition Fee                 3,500.00 │    │
│  │ Transport Fee               1,000.00 │    │
│  │ Library Fee                   200.00 │    │
│  │                              ─────── │    │
│  │ Total                      4,700.00 │    │
│  │ Paid                           0.00 │    │
│  │ Balance Due                4,700.00 │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  Payment History:                            │
│  No payments recorded yet.                   │
└─────────────────────────────────────────────┘
```

### Payment History

The payment history table shows:

| Column | Description |
|--------|-------------|
| Date & Time | Payment timestamp |
| Amount | Payment amount |
| Method | Cash, Bank, Mobile, etc. |
| Notes | Reference or transaction ID |
| Received By | Staff member who recorded |

---

## Recording Payments

### Admin Panel

On the invoice detail page, use the **Record Payment** form:

| Field | Required | Description |
|-------|----------|-------------|
| Payment Amount | Yes | Cannot exceed balance due |
| Payment Date | Yes | Defaults to today |
| Payment Method | Yes | Cash, Bank, Mobile, Cheque, Card |
| Notes | No | Optional reference number |

### Frontend (AJAX)

The frontend invoice view supports AJAX payment recording without page reload:

```javascript
// Form submission via AJAX
$.ajax({
    url: ajaxurl,
    type: 'POST',
    data: {
        action: 'ik_add_invoice_payment',
        nonce: ikInvoicesNonce,
        form_data: formData
    },
    success: function(response) {
        // Update payment history table
        // Update balance display
        // Show success message
    }
});
```

### Payment Validation

The system enforces these rules:

```php
// Amount must be positive
if ($payment_amount <= 0) {
    wp_send_json_error(['message' => 'Missing required fields.']);
}

// Cannot overpay
if (($payment_amount - $balance_due) > 0.001) {
    wp_send_json_error(['message' => 'Payment amount cannot exceed the balance due.']);
}
```

### After Payment

When a payment is recorded:

1. Invoice `amount_paid` is updated
2. If fully paid, status changes to `paid`
3. If partially paid, status changes to `partial`
4. Transaction record is created in `institutionkit_transactions`
5. Payment history table updates dynamically

---

## Invoice Status Flow

```
Unpaid → Partial → Paid
  │         │         │
  │         │         └── Full payment received
  │         │
  │         └── Some payment received, balance remaining
  │
  └── No payment received yet
```

### Status Colors

| Status | Color | CSS Class |
|--------|-------|-----------|
| Paid | Green (#28a745) | `ik-status-paid` |
| Unpaid | Red (#dc3545) | `ik-status-unpaid` |
| Partial | Yellow (#ffc107) | `ik-status-partial` |

---

## PDF Invoice Download

Invoices can be downloaded as PDF from both the admin panel and frontend.

### Admin Panel

Click the **Download PDF** button on the single invoice view.

### Frontend

Add `?ik_pdf=1` to the invoice URL or click the **Download PDF** button.

### PDF Generation

PDFs are generated using Dompdf with:

- Institution letterhead
- Student and guardian details
- Full fee breakdown table
- Payment history
- Watermark (PAID or DUE)
- Computer-generated disclaimer

---

## Invoice Reminders

### Automatic Reminders

A scheduled cron job sends reminders for overdue invoices:

```php
wp_schedule_event(
    strtotime('2026-05-11 09:00:00'),
    'monthly',
    'ik_monthly_invoice_reminder_event'
);
```

### Manual Reminders

From the dashboard, click **Send Fee Reminders** to email all parents with overdue balances.

### Reminder Logic

For each overdue invoice:

1. Look up parent email from student meta or parent-child links
2. Skip students with no email address (counted in results)
3. Send email with invoice details and outstanding amount
4. Log the notification in `institutionkit_notifications`

---

## Invoice Database

### Invoices Table

```sql
CREATE TABLE wp_institutionkit_invoices (
    invoice_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    student_id BIGINT(20) UNSIGNED NOT NULL,
    class_id BIGINT(20) UNSIGNED NOT NULL,
    title VARCHAR(255) NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    amount_paid DECIMAL(10,2) DEFAULT 0.00,
    due_date DATE,
    status VARCHAR(20) DEFAULT 'unpaid',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    campus_id INT DEFAULT 1,
    KEY student_id (student_id),
    KEY class_id (class_id),
    KEY campus_id (campus_id)
);
```

### Invoice Items Table

```sql
CREATE TABLE wp_institutionkit_invoice_items (
    item_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    invoice_id BIGINT(20) UNSIGNED NOT NULL,
    fee_type_name VARCHAR(255) NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    KEY invoice_id (invoice_id)
);
```

### Transactions Table

```sql
CREATE TABLE wp_institutionkit_transactions (
    transaction_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    invoice_id BIGINT(20) UNSIGNED NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    payment_method VARCHAR(50) NOT NULL,
    payment_date DATE NOT NULL,
    notes TEXT,
    recorded_by BIGINT(20) UNSIGNED NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    campus_id INT DEFAULT 1,
    KEY invoice_id (invoice_id),
    KEY campus_id (campus_id)
);
```

---

## Programmatic Usage

### Generate an Invoice

```php
global $wpdb;

// Create invoice
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_invoices",
    [
        'student_id'   => $student_id,
        'class_id'     => $class_id,
        'title'        => 'Monthly Fees - June 2026',
        'total_amount' => 4700.00,
        'due_date'     => '2026-06-05',
        'status'       => 'unpaid',
        'campus_id'    => $campus_id,
    ]
);
$invoice_id = $wpdb->insert_id;

// Add line items
$items = [
    ['Tuition Fee', 3500],
    ['Transport Fee', 1000],
    ['Library Fee', 200],
];

foreach ($items as $item) {
    $wpdb->insert(
        "{$wpdb->prefix}institutionkit_invoice_items",
        [
            'invoice_id'    => $invoice_id,
            'fee_type_name' => $item[0],
            'amount'        => $item[1],
        ]
    );
}
```

### Record a Payment

```php
global $wpdb;

// Get current invoice
$invoice = $wpdb->get_row($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_invoices WHERE invoice_id = %d",
    $invoice_id
));

$payment_amount = 2500;
$new_amount_paid = $invoice->amount_paid + $payment_amount;
$new_status = ($new_amount_paid >= $invoice->total_amount) ? 'paid' : 'partial';

// Update invoice
$wpdb->update(
    "{$wpdb->prefix}institutionkit_invoices",
    [
        'amount_paid' => $new_amount_paid,
        'status'      => $new_status,
    ],
    ['invoice_id' => $invoice_id]
);

// Record transaction
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_transactions",
    [
        'invoice_id'     => $invoice_id,
        'amount'         => $payment_amount,
        'payment_method' => 'Cash',
        'payment_date'   => current_time('Y-m-d'),
        'recorded_by'    => get_current_user_id(),
    ]
);
```

### Get Outstanding Invoices

```php
global $wpdb;
$outstanding = $wpdb->get_results($wpdb->prepare(
    "SELECT 
        i.*,
        p.post_title as student_name
     FROM {$wpdb->prefix}institutionkit_invoices i
     JOIN {$wpdb->posts} p ON i.student_id = p.ID
     WHERE i.due_date < %s 
       AND i.status != 'paid'
       AND i.campus_id = %d
     ORDER BY i.due_date ASC",
    current_time('Y-m-d'), $campus_id
));
```
