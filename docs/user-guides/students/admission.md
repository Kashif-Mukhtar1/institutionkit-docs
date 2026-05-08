File name: `docs/user-guides/students/admission.md`

```markdown
# New Admission

The admission process enrolls new students into InstitutionKit, capturing all required personal, guardian, and academic information in a single workflow.

---

## Accessing the Admission Form

There are two ways to start a new admission:

| Method | Path | Best For |
|--------|------|----------|
| **Admin Admission** | InstitutionKit → Student Management → New Admission | Staff processing walk-in or paper applications |
| **Frontend Form** | Public page with `[institutionkit_admission_form]` shortcode | Parents self-registering online |

---

## Admin Admission Form

Navigate to **InstitutionKit → Student Management → New Admission**.

### Form Sections

The admin form collects data across four sections:

#### 1. Student Information

| Field | Required | Validation |
|-------|----------|------------|
| Student Full Name | Yes | Text, min 2 characters |
| Date of Birth | Yes | Date picker, must be valid date |
| Gender | Yes | Select: Male / Female / Other |
| Class Applying For | Yes | Dropdown of all published `ik_class` posts |
| Photo | No | Image upload |

#### 2. Guardian Information

| Field | Required | Validation |
|-------|----------|------------|
| Guardian Title | Yes | Select: Father / Mother / Guardian |
| Guardian Full Name | Yes | Text |
| Guardian Email | Yes | Valid email format |
| ID Card Number | Yes | 13-digit number, no dashes |
| Guardian Occupation | No | Text |
| Guardian Qualification | No | Text |

#### 3. Parent Contact Information

| Field | Required | Validation |
|-------|----------|------------|
| Father's Contact Number | At least one | Phone number |
| Mother's Contact Number | At least one | Phone number |

#### 4. Address

| Field | Required |
|-------|----------|
| Full Address | No |

---

## Frontend Admission Form

The frontend form is rendered via shortcode on any WordPress page:

```
[institutionkit_admission_form]
```

### Form Security

The frontend form implements multiple security layers:

```php
// 1. Nonce verification
if (!wp_verify_nonce($_POST['ik_admissions_nonce'], 'ik_submit_admission')) {
    return;
}

// 2. CSRF protection via WordPress nonce
wp_nonce_field('ik_submit_admission', 'ik_admissions_nonce');

// 3. Server-side validation for all fields
if (empty($this->form_data['student_name'])) {
    $this->form_errors[] = 'Student\'s name is required.';
}

// 4. Email format validation
if (!is_email($this->form_data['email'])) {
    $this->form_errors[] = 'A valid guardian email address is required.';
}

// 5. ID card format validation (13-digit)
if (!preg_match('/^\d{13}$/', $this->form_data['id_card_number'])) {
    $this->form_errors[] = 'ID card number must be a 13-digit number without dashes.';
}
```

### Validation Rules

| Rule | Error Message |
|------|--------------|
| Name required | "Student's name is required." |
| DOB required | "Date of birth is required." |
| Gender required | "Gender is required." |
| Valid email | "A valid guardian email address is required." |
| ID card required | "Guardian's ID card number is required." |
| ID card format | "ID card number must be a 13-digit number without dashes." |
| Guardian name | "Guardian's name is required." |
| At least one contact | "At least one contact number (Father or Mother) is required." |
| Class selected | "Please select a class to apply for." |

---

## Processing an Admission

When the form is submitted, the system:

1. **Validates all fields** — returns errors if validation fails
2. **Creates a WordPress post** of type `ik_student` with status `publish`
3. **Stores all meta fields** as post meta
4. **Sets emergency contact** from the father's contact number
5. **Redirects** to the same page with `?admission-success=true`

### Data Stored

```php
// Post creation
$post_data = [
    'post_type'    => 'ik_student',
    'post_title'   => $student_name,
    'post_status'  => 'publish',
];
$post_id = wp_insert_post($post_data);

// Meta fields stored
update_post_meta($post_id, '_ik_student_class_id', $class_id);
update_post_meta($post_id, '_ik_date_of_birth', $dob);
update_post_meta($post_id, '_ik_gender', $gender);
update_post_meta($post_id, '_ik_email', $email);
update_post_meta($post_id, '_ik_cnic_number', $id_card);
update_post_meta($post_id, '_ik_guardian_title', $guardian_title);
update_post_meta($post_id, '_ik_guardian_name', $guardian_name);
update_post_meta($post_id, '_ik_father_contact', $father_contact);
update_post_meta($post_id, '_ik_mother_contact', $mother_contact);
update_post_meta($post_id, '_ik_father_occupation', $father_occupation);
update_post_meta($post_id, '_ik_mother_occupation', $mother_occupation);
update_post_meta($post_id, '_ik_father_qualification', $father_qualification);
update_post_meta($post_id, '_ik_mother_qualification', $mother_qualification);
update_post_meta($post_id, '_ik_address', $address);
update_post_meta($post_id, '_ik_emergency_contact', $father_contact);
```

---

## After Admission

Once a student is enrolled, the next steps are:

| Step | Action | Location |
|------|--------|----------|
| 1 | Assign a roll number | Edit Student → Roll Number field |
| 2 | Assign fees | Fee Management → Assign Fees |
| 3 | Link parent account | Users → Parent Management |
| 4 | Generate invoice | Fee Management → Invoices |
| 5 | Mark initial attendance | Student Management → Attendance |

---

## Frontend Form Template

The admission form template is located at:

```
/wp-content/plugins/institutionkit-frontend-admissions/templates/admission-form-template.php
```

### Customizing the Form

To override the template in your theme:

1. Copy the template file to your theme directory:
   ```
   /wp-content/themes/your-theme/institutionkit/admission-form-template.php
   ```

2. InstitutionKit will automatically detect and use your theme's version

### CSS Classes

| Class | Target |
|-------|--------|
| `.ik-admission-form` | Form wrapper |
| `.ik-form-group` | Each field container |
| `.ik-form-label` | Field labels |
| `.ik-form-input` | Input fields |
| `.ik-form-error` | Validation error messages |
| `.ik-form-success` | Success message after submission |
| `.ik-submit-btn` | Submit button |

---

## Bulk Import (Planned)

!!! info "Coming Soon"
    CSV bulk import for admissions is planned for a future release. It will support:
    
    - Upload a CSV with student data
    - Map CSV columns to InstitutionKit fields
    - Preview before import
    - Error handling for invalid rows

---

## Admission Applications CPT

Frontend admissions create posts of type `ik_admission_app` with post status `publish`. These are distinct from enrolled students (`ik_student`). The application CPT is used for tracking applications before enrollment.

### Application vs Student

| Type | Post Type | Status |
|------|-----------|--------|
| Application | `ik_admission_app` | Pending review |
| Enrolled Student | `ik_student` | Active |
```
