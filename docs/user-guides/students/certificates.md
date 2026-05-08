```markdown
# Certificates

The Certificates module generates official school certificates for students and staff. Four certificate types are supported: Leaving, Character, Achievement, and Employment.

---

## Accessing Certificates

Navigate to **InstitutionKit → Student Management → Certificates**.

---

## Certificate Types

| Type | Recipient | Purpose |
|------|-----------|---------|
| **Leaving Certificate** | Student | Issued when a student leaves the institution |
| **Character Certificate** | Student | Attests to the student's conduct and character |
| **Achievement Certificate** | Student | Recognizes academic or extracurricular achievement |
| **Employment Certificate** | Teacher/Staff | Verifies employment history |

---

## Creating a Certificate

### Step 1: Select Certificate Type

Choose from the four certificate types. Each type has different fields.

### Step 2: Select Recipient

| For Students | For Teachers |
|-------------|--------------|
| Search by name or roll number | Search by name or employee code |
| Select from enrolled students | Select from active staff |

### Step 3: Fill Certificate Details

#### Leaving Certificate Fields

| Field | Description |
|-------|-------------|
| Student Name | Auto-filled from student record |
| Father's Name | Auto-filled from `_ik_father_name` |
| Date of Birth | Auto-filled from `_ik_date_of_birth` |
| Admission Date | Date of admission |
| Conduct | Conduct assessment (e.g., "Good", "Satisfactory") |
| Duration | Period of study (e.g., "March 2020 to June 2025") |
| Last Month/Year | Final month and year of attendance |
| Remarks | Additional notes |

#### Character Certificate Fields

| Field | Description |
|-------|-------------|
| Student Name | Auto-filled |
| Father's Name | Auto-filled |
| Date of Birth | Auto-filled |
| Conduct | Character assessment |
| Achievement Details | Academic and behavioral achievements |
| Remarks | Additional notes |

#### Achievement Certificate Fields

| Field | Description |
|-------|-------------|
| Student Name | Auto-filled |
| Achievement Details | Description of the achievement |
| Issue Date | Certificate issue date |
| Remarks | Additional notes |

#### Employment Certificate Fields

| Field | Description |
|-------|-------------|
| Staff Name | Auto-filled from staff record |
| Designation | Job title |
| Monthly Salary | Final or average salary |
| Duration | Employment period |
| Conduct | Performance assessment |
| Remarks | Additional notes |

### Step 4: Generate Certificate

Click **Generate Certificate** to create the certificate record. The certificate is assigned a unique certificate number.

---

## Certificate Database

Certificates are stored in the `institutionkit_certificates` table:

```sql
CREATE TABLE wp_institutionkit_certificates (
    certificate_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    recipient_id BIGINT(20) UNSIGNED NOT NULL,
    student_name VARCHAR(255),
    father_name VARCHAR(255),
    dob DATE,
    admission_date DATE,
    conduct VARCHAR(50),
    duration_text VARCHAR(100),
    last_month_year VARCHAR(50),
    achievement_details TEXT,
    monthly_salary VARCHAR(100),
    designation VARCHAR(100),
    recipient_type ENUM('student', 'teacher') NOT NULL DEFAULT 'student',
    certificate_type ENUM('leaving', 'character', 'achievement', 'employment') NOT NULL,
    issue_date DATE NOT NULL,
    remarks TEXT,
    issued_by BIGINT(20) UNSIGNED,
    certificate_number VARCHAR(50),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## Certificate Templates

Certificate templates are managed as WordPress posts of type `ik_certificate`.

### Accessing Templates

Navigate to **InstitutionKit → Certificate Templates** in the admin menu.

### Template Features

- Custom HTML/CSS design
- Placeholder shortcodes for dynamic data
- Preview before printing
- Multiple templates for different certificate types

---

## Certificate Verification

### Public Verification

InstitutionKit provides a public certificate verification system. Anyone can verify a certificate by entering the certificate number.

**Shortcode:**

```
[ik_certificate_verify]
```

Place this shortcode on a public page to allow certificate verification.

### How Verification Works

1. Visitor enters a certificate number
2. System queries `institutionkit_certificates` for a match
3. If found, displays:
    - Certificate type
    - Recipient name
    - Issue date
    - Institution name
4. If not found, displays: "Certificate not found"

---

## Certificate List View

The Certificates page shows all issued certificates in a table:

| Column | Description |
|--------|-------------|
| Certificate # | Unique certificate number |
| Recipient | Student or staff name |
| Type | Leaving / Character / Achievement / Employment |
| Issue Date | Date of issuance |
| Issued By | Staff member who issued |
| Actions | View / Print / Delete |

### Filters

| Filter | Description |
|--------|-------------|
| Certificate Type | Filter by type |
| Date Range | Filter by issue date |
| Search | Search by recipient name or certificate number |

---

## Printing Certificates

### Print Single Certificate

Click the **Print** button on any certificate row. This opens a print-optimized view with:

- Institution letterhead
- Certificate content
- QR code or verification number
- Signature lines

### Print CSS

Certificates use `@media print` CSS for clean output:

```css
@media print {
    /* Hide admin UI elements */
    .ik-certificate-controls,
    #wpadminbar,
    .notice {
        display: none !important;
    }
    
    /* Clean certificate layout */
    .ik-certificate-printable {
        width: 100%;
        padding: 0;
        background: white;
    }
}
```

---

## Programmatic Certificate Creation

### Issue a Certificate via Code

```php
global $wpdb;
$cert_number = 'CERT-' . date('Y') . '-' . str_pad(rand(1, 9999), 4, '0', STR_PAD_LEFT);

$wpdb->insert(
    "{$wpdb->prefix}institutionkit_certificates",
    [
        'recipient_id'       => $student_id,
        'recipient_type'     => 'student',
        'certificate_type'   => 'leaving',
        'student_name'       => get_the_title($student_id),
        'father_name'        => get_post_meta($student_id, '_ik_father_name', true),
        'dob'                => get_post_meta($student_id, '_ik_date_of_birth', true),
        'conduct'            => 'Good',
        'duration_text'      => 'March 2020 to June 2025',
        'issue_date'         => current_time('Y-m-d'),
        'certificate_number' => $cert_number,
        'issued_by'          => get_current_user_id(),
    ]
);
```

### Verify a Certificate

```php
global $wpdb;
$cert = $wpdb->get_row($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_certificates 
     WHERE certificate_number = %s",
    $certificate_number
));

if ($cert) {
    echo "Certificate is valid.";
    echo "Issued to: " . esc_html($cert->student_name ?: $cert->staff_name);
    echo "Date: " . esc_html($cert->issue_date);
} else {
    echo "Certificate not found.";
}
```

---

## Capabilities

| Capability | Who Has It | Purpose |
|-----------|------------|---------|
| `manage_options` | Administrator | Full access to certificates |
| Campus admin capabilities | Campus Admin | View and issue certificates for their campus |
```
