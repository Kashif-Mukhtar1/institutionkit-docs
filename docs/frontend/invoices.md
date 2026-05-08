```markdown
# Frontend Invoices

The Frontend Invoices module enables staff members to view, filter, and manage student invoices from the frontend of your website — with payment recording, PDF downloads, and campus-based access control.

---

## Setup

### 1. Install the Plugin

Upload and activate the `institutionkit-frontend-invoices` plugin.

### 2. Create an Invoices Page

1. Create a new WordPress page (e.g., "Invoices" or "Fee Management")
2. Add the shortcode:

```
[ik_invoices]
```

3. Publish the page
4. Restrict access to logged-in staff

---

## Authentication

### Access Control

Only administrators and staff members with campus assignments can access:

```php
private function user_can_view_invoices() {
    if (current_user_can('manage_options')) return true;
    return $this->get_current_user_campus_id() !== null;
}
```

### Campus Assignment Check

Staff campus is determined from the staff table:

```php
private function get_current_user_campus_id() {
    $user_id = get_current_user_id();
    if (!$user_id) return null;
    
    global $wpdb;
    return $wpdb->get_var($wpdb->prepare(
        "SELECT primary_campus_id FROM {$wpdb->prefix}institutionkit_staff 
         WHERE user_id = %d AND employment_status = 'active' LIMIT 1",
        $user_id
    ));
}
```

---

## Invoice List View

### Filter Bar

| Filter | Description |
|--------|-------------|
| Status Tabs | All / Unpaid / Partial |
| Class | Dropdown of all classes |
| Section | Dropdown of all sections |
| Search | Search by student name or roll number |

### Status Tab Styling

| Tab | Active Color |
|-----|-------------|
| All | Blue |
| Unpaid | Red |
| Partial | Orange |

### Invoice Table

| Column | Description |
|--------|-------------|
| ID | Invoice number |
| Title | Invoice title (linked to detail view) |
| Student | Student name |
| Roll # | Student roll number |
| Guardian Contact | Emergency contact |
| Total | Total invoice amount |
| Paid | Amount received |
| Balance | Outstanding amount (bold) |
| Status | Color-coded badge |
| Date | Creation date |
| Actions | View button |

---

## Single Invoice View

### Invoice Header

```
┌─────────────────────────────────────────┐
│  [Logo]          ← Back    🖨️ Print    📄 PDF  │
└─────────────────────────────────────────┘
```

### Invoice Details

```
┌──────────────────────────────────────────┐
│  Student: Ahmed Khan                     │
│  Class: Grade 5                          │
│  Roll #: 42                              │
│  Guardian: Muhammad Khan                 │
│                                           │
│  Invoice #: 127          Date: Jun 1, 26 │
│  Status: Unpaid                          │
│                                           │
│  ┌──────────────────────────────────┐    │
│  │ Fee Breakdown                    │    │
│  │ Tuition Fee             3,500.00 │    │
│  │ Transport Fee           1,000.00 │    │
│  │ Library Fee               200.00 │    │
│  │                          ─────── │    │
│  │ Total                  4,700.00 │    │
│  │ Paid                       0.00 │    │
│  │ Balance Due            4,700.00 │    │
│  └──────────────────────────────────┘    │
│                                           │
│  Payment History:                        │
│  No payments recorded yet.               │
└──────────────────────────────────────────┘
```

---

## Recording Payments

### Payment Form

| Field | Required | Description |
|-------|:---:|-------------|
| Payment Amount | Yes | Cannot exceed balance due |
| Payment Date | Yes | Defaults to today |
| Payment Method | Yes | Cash, Bank, Mobile |
| Notes | No | Optional reference |

### AJAX Payment Recording

Payments are recorded via AJAX without page reload:

```javascript
$('#ik-record-payment-form').on('submit', function(e) {
    e.preventDefault();
    
    $.ajax({
        url: ajaxurl,
        type: 'POST',
        data: {
            action: 'ik_add_invoice_payment',
            nonce: ikInvoicesNonce,
            form_data: $(this).serialize()
        },
        success: function(response) {
            if (response.success) {
                // Update payment history table
                $('#payment-history-tbody').append(response.data.new_payment_row_html);
                
                // Update balance display
                $('.total-paid-amount').text(response.data.new_total_paid_formatted);
                $('.outstanding-balance-amount').text(response.data.new_outstanding_balance_formatted);
                
                // If fully paid, show paid message
                if (response.data.is_fully_paid) {
                    $('#ik-record-payment-form').html(
                        '<div class="ik-notice ik-notice-success">' +
                        '<p><strong>This invoice is fully paid.</strong></p>' +
                        '</div>'
                    );
                }
            }
        }
    });
});
```

### Payment Validation

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

The invoice display updates dynamically:

1. Payment history table gets a new row
2. Amount paid updates
3. Balance due updates
4. If fully paid, the payment form is replaced with a success message

---

## Campus Data Restriction

Staff can only view invoices for students in their campus:

```php
if (!current_user_can('manage_options')) {
    $student_campus = get_post_meta($invoice->student_id, '_ik_campus_id', true);
    if ($student_campus != $this->get_current_user_campus_id()) {
        echo 'Access Denied';
        return;
    }
}
```

### List View Filtering

The invoice list is pre-filtered by campus:

```php
if ($is_restricted && $campus_id) {
    $student_ids_in_campus = $wpdb->get_col($wpdb->prepare(
        "SELECT post_id FROM {$wpdb->postmeta} 
         WHERE meta_key = '_ik_campus_id' AND meta_value = %d",
        $campus_id
    ));
    
    if (empty($student_ids_in_campus)) {
        echo 'No students found in your campus.';
        return;
    }
    
    $where_clauses[] = "i.student_id IN (" . implode(',', array_fill(0, count($student_ids_in_campus), '%d')) . ")";
}
```

---

## PDF Download

### Download Button

Click **Download PDF** on any invoice to generate a formatted PDF:

```
https://school.edu/invoices/?view=single&invoice_id=127&ik_pdf=1
```

### PDF Generation

The frontend uses the same PDF generator as the admin panel:

```php
if (class_exists('IK_Invoices_Page')) {
    $invoices_page = new IK_Invoices_Page();
    $pdf_content = $invoices_page->generate_invoice_pdf_raw($invoice, $items);
    
    header('Content-Type: application/pdf');
    header('Content-Disposition: attachment; filename="Invoice-127-ahmed-khan.pdf"');
    echo $pdf_content;
    exit;
}
```

---

## Print Styles

The invoice view includes `@media print` CSS for clean printing:

- Hides navigation, buttons, and sidebar
- Removes admin UI elements
- Sets white background
- Shows watermark (PAID or DUE)
- Optimizes layout for A4 paper

```css
@media print {
    #wpadminbar, header, nav, footer,
    .ik-main-header, .ik-invoice-sidebar,
    .ik-main-header-actions { 
        display: none !important; 
    }
    
    .ik-watermark {
        font-size: 60pt;
        opacity: 0.15;
        position: absolute;
        top: 40%;
        left: 20%;
        transform: rotate(-30deg);
    }
}
```

---

## Status Colors

| Status | Color | CSS Class |
|--------|-------|-----------|
| Paid | Green (#28a745) | `status-paid` |
| Unpaid | Red (#dc3545) | `status-unpaid` |
| Partial | Yellow (#ffc107) | `status-partial` |

---

## Programmatic Usage

### Render Invoice List

```php
$manager = IK_Frontend_Invoices_Manager::instance();
$manager->render_view();
```

### Record Payment Programmatically

```php
$manager->handle_form_actions();
// Listens for $_POST['ik_action'] === 'record_payment'
```

### Handle AJAX Payment

```php
add_action('wp_ajax_ik_add_invoice_payment', function() {
    $manager = IK_Frontend_Invoices_Manager::instance();
    $manager->handle_ajax_add_payment();
});
```
