```markdown
# Frontend Admissions

The Frontend Admissions module allows parents to submit admission applications online through a public-facing form on your website.

---

## Setup

### 1. Install the Plugin

Upload and activate the `institutionkit-frontend-admissions` plugin.

### 2. Create an Admissions Page

1. Create a new WordPress page (e.g., "Admissions" or "Apply Now")
2. Add the shortcode:

```
[institutionkit_admission_form]
```

3. Publish the page

### 3. Optional: Customize the Template

Copy the template to your theme:

```
/wp-content/plugins/institutionkit-frontend-admissions/templates/admission-form-template.php
↓ Copy to ↓
/wp-content/themes/your-theme/institutionkit/admission-form-template.php
```

InstitutionKit will automatically use your theme's version.

---

## Form Fields

### Student Information

| Field | Required | Validation |
|-------|:---:|------------|
| Student Name | Yes | Text, min 2 characters |
| Date of Birth | Yes | Date picker |
| Gender | Yes | Select: Male / Female / Other |
| Guardian Email | Yes | Valid email format |
| ID Card Number | Yes | 13-digit number, no dashes |
| Class Applying For | Yes | Dropdown of all published `ik_class` posts |

### Guardian Information

| Field | Required | Validation |
|-------|:---:|------------|
| Guardian Title | Yes | Select: Father / Mother / Guardian |
| Guardian Name | Yes | Text |
| Father's Contact | At least one | Phone number |
| Mother's Contact | At least one | Phone number |
| Father's Occupation | No | Text |
| Mother's Occupation | No | Text |
| Father's Qualification | No | Text |
| Mother's Qualification | No | Text |

### Address

| Field | Required |
|-------|:---:|
| Full Address | No |

---

## Validation Rules

All validation happens server-side:

```php
private function validate_form_data() {
    if (empty($this->form_data['student_name'])) {
        $this->form_errors[] = 'Student\'s name is required.';
    }
    if (empty($this->form_data['dob'])) {
        $this->form_errors[] = 'Date of birth is required.';
    }
    if (empty($this->form_data['gender'])) {
        $this->form_errors[] = 'Gender is required.';
    }
    if (!is_email($this->form_data['email'])) {
        $this->form_errors[] = 'A valid guardian email address is required.';
    }
    if (empty($this->form_data['id_card_number'])) {
        $this->form_errors[] = 'Guardian\'s ID card number is required.';
    } elseif (!preg_match('/^\d{13}$/', $this->form_data['id_card_number'])) {
        $this->form_errors[] = 'ID card number must be a 13-digit number without dashes.';
    }
    if (empty($this->form_data['guardian_name'])) {
        $this->form_errors[] = 'Guardian\'s name is required.';
    }
    if (empty($this->form_data['father_contact']) && empty($this->form_data['mother_contact'])) {
        $this->form_errors[] = 'At least one contact number is required.';
    }
    if (empty($this->form_data['class_id'])) {
        $this->form_errors[] = 'Please select a class to apply for.';
    }
}
```

---

## Processing a Submission

When the form is submitted successfully:

1. A WordPress post of type `ik_student` is created with status `publish`
2. All 16 meta fields are saved as post meta
3. Emergency contact is set from the father's contact number
4. The user is redirected with `?admission-success=true`

```php
private function process_application() {
    $post_id = wp_insert_post([
        'post_type'    => 'ik_student',
        'post_title'   => $this->form_data['student_name'],
        'post_status'  => 'publish',
    ]);
    
    update_post_meta($post_id, '_ik_student_class_id', $this->form_data['class_id']);
    update_post_meta($post_id, '_ik_date_of_birth', $this->form_data['dob']);
    update_post_meta($post_id, '_ik_gender', $this->form_data['gender']);
    update_post_meta($post_id, '_ik_email', $this->form_data['email']);
    update_post_meta($post_id, '_ik_cnic_number', $this->form_data['id_card_number']);
    update_post_meta($post_id, '_ik_guardian_title', $this->form_data['guardian_title']);
    update_post_meta($post_id, '_ik_guardian_name', $this->form_data['guardian_name']);
    update_post_meta($post_id, '_ik_father_contact', $this->form_data['father_contact']);
    update_post_meta($post_id, '_ik_mother_contact', $this->form_data['mother_contact']);
    update_post_meta($post_id, '_ik_father_occupation', $this->form_data['father_occupation']);
    update_post_meta($post_id, '_ik_mother_occupation', $this->form_data['mother_occupation']);
    update_post_meta($post_id, '_ik_father_qualification', $this->form_data['father_qualification']);
    update_post_meta($post_id, '_ik_mother_qualification', $this->form_data['mother_qualification']);
    update_post_meta($post_id, '_ik_address', $this->form_data['address']);
    update_post_meta($post_id, '_ik_emergency_contact', $this->form_data['father_contact']);
    
    wp_safe_redirect(add_query_arg('admission-success', 'true', get_permalink()));
    exit;
}
```

---

## Form Security

| Layer | Implementation |
|-------|---------------|
| Nonce | `wp_nonce_field('ik_submit_admission', 'ik_admissions_nonce')` |
| Server Validation | All fields validated regardless of client-side checks |
| Sanitization | `sanitize_text_field()`, `sanitize_email()`, `sanitize_textarea_field()`, `absint()` |
| ID Card Format | Strict 13-digit numeric regex validation |

---

## CSS Classes

| Class | Target |
|-------|--------|
| `.ik-admission-form` | Form wrapper |
| `.ik-form-group` | Each field container |
| `.ik-form-label` | Field labels |
| `.ik-form-input` | Input fields |
| `.ik-form-error` | Validation error messages |
| `.ik-form-success` | Success message banner |
| `.ik-submit-btn` | Submit button |

---

## Error Display Example

```
┌──────────────────────────────────────────┐
│ ⚠️ Student's name is required.           │
│ ⚠️ A valid guardian email is required.   │
└──────────────────────────────────────────┘

┌──────────────────────────────────────────┐
│ Student Name: [_______________] *        │
│ Date of Birth: [__/__/____] *           │
│ Gender: [Select ▼] *                    │
│ ...                                      │
└──────────────────────────────────────────┘
```

---

## Success State

After successful submission:

```
✅ Your application has been submitted successfully!

The school administration will review your application
and contact you at the provided email address.
```

---

## After Submission

The student is created as a published `ik_student` post. Administrators should:

1. Review the student in **Student Management → All Students**
2. Assign a roll number
3. Assign fee structures
4. Link parent WordPress account if applicable
5. Generate invoices

---

## Programmatic Usage

### Render the Form Anywhere

```php
$manager = IK_Frontend_Admissions_Manager::instance();
$manager->render_admission_form();
```

### Hook into Submission

```php
add_action('ik_admission_submitted', function($post_id, $form_data) {
    // Send notification email
    // Log to external system
    // Trigger workflow
}, 10, 2);
```
