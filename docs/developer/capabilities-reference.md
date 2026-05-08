```markdown
# Capabilities Reference

InstitutionKit defines 62 custom capabilities across 5 user roles. This reference lists every capability, its purpose, and which roles have it by default.

---

## Administrator (Super Admin)

**All 62 capabilities** — unrestricted access to every feature.

```php
if (current_user_can('manage_options')) {
    // Administrators bypass all InstitutionKit capability checks
}
```

---

## Complete Capability List by Module

### Student Management

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_students` | ✅ | ✅ | — | — | — |
| `ik_view_students` | ✅ | ✅ | ✅ | — | — |
| `ik_edit_students` | ✅ | ✅ | — | — | — |
| `ik_delete_students` | ✅ | — | — | — | — |
| `ik_manage_admissions` | ✅ | ✅ | — | — | — |

### Staff Management

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_staff` | ✅ | ✅ | — | — | — |
| `ik_manage_staff_attendance` | ✅ | ✅ | — | — | — |
| `ik_view_attendance_reports` | ✅ | ✅ | — | — | — |

### Student Attendance

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_student_attendance` | ✅ | ✅ | ✅ | — | — |

### Fee & Invoice Management

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_fees` | ✅ | ✅ | — | ✅ | — |
| `ik_view_invoices` | ✅ | ✅ | — | ✅ | ✅ |
| `ik_generate_invoices` | ✅ | ✅ | — | ✅ | — |
| `ik_record_payments` | ✅ | ✅ | — | ✅ | — |
| `ik_email_invoices` | ✅ | — | — | ✅ | — |

### Exam Management

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_exam_types` | ✅ | — | — | — | — |
| `ik_manage_exam_schedules` | ✅ | ✅ | ✅ | — | — |
| `ik_enter_exam_results` | ✅ | ✅ | ✅ | — | — |
| `ik_verify_exam_results` | ✅ | — | — | — | — |
| `ik_publish_exam_results` | ✅ | — | — | — | — |
| `ik_generate_report_cards` | ✅ | ✅ | ✅ | — | — |
| `ik_view_exam_analytics` | ✅ | ✅ | ✅ | — | — |

### Payroll & Expenses

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_expenses` | ✅ | ✅ | — | ✅ | — |
| `ik_approve_expenses` | ✅ | — | — | ✅ | — |
| `ik_view_expense_reports` | ✅ | ✅ | — | ✅ | — |
| `ik_manage_expense_heads` | ✅ | — | — | ✅ | — |
| `ik_manage_staff` | ✅ | ✅ | — | — | — |
| `ik_manage_payroll` | ✅ | ✅ | — | ✅ | — |
| `ik_approve_payroll` | ✅ | — | — | ✅ | — |
| `ik_view_payroll_reports` | ✅ | ✅ | — | ✅ | — |
| `ik_manage_budgets` | ✅ | — | — | ✅ | — |
| `ik_view_financial_comparison` | ✅ | — | — | ✅ | — |
| `ik_view_profit_loss` | ✅ | ✅ | — | ✅ | — |
| `ik_view_all_campus_finances` | ✅ | — | — | — | — |
| `ik_export_financial_data` | ✅ | — | — | ✅ | — |

### Gradebook

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_gradebook` | ✅ | ✅ | ✅ | — | — |
| `ik_view_gradebook` | ✅ | ✅ | ✅ | — | ✅ |

### Homework

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_assign_homework` | ✅ | ✅ | ✅ | — | — |
| `ik_edit_homework` | ✅ | — | ✅ | — | — |
| `ik_delete_homework` | ✅ | — | — | — | — |
| `ik_view_all_homework` | ✅ | ✅ | ✅ | — | — |
| `ik_grade_submissions` | ✅ | ✅ | ✅ | — | — |

### Meetings

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_meeting_slots` | ✅ | ✅ | ✅ | — | — |
| `ik_view_meeting_bookings` | ✅ | ✅ | ✅ | — | — |
| `ik_manage_meeting_bookings` | ✅ | ✅ | — | — | — |
| `ik_book_meeting_slots` | ✅ | — | — | — | ✅ |
| `ik_view_own_meetings` | ✅ | — | — | — | ✅ |

### Campus Management

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_campuses` | ✅ | — | — | — | — |
| `ik_view_all_campuses` | ✅ | — | — | — | — |
| `ik_manage_campus` | ✅ | ✅ | — | — | — |
| `ik_switch_campus` | ✅ | — | — | — | — |
| `ik_view_campus_dashboard` | ✅ | ✅ | — | — | — |
| `ik_manage_campus_students` | ✅ | ✅ | — | — | — |
| `ik_manage_campus_teachers` | ✅ | ✅ | — | — | — |
| `ik_manage_campus_fees` | ✅ | ✅ | — | — | — |
| `ik_manage_campus_attendance` | ✅ | ✅ | — | — | — |
| `ik_manage_campus_gradebook` | ✅ | ✅ | — | — | — |
| `ik_manage_campus_settings` | ✅ | ✅ | — | — | — |

### Student Promotions

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_student_promotions` | ✅ | ✅ | — | — | — |
| `ik_view_promotion_history` | ✅ | ✅ | ✅ | — | — |
| `ik_bulk_promote_students` | ✅ | ✅ | — | — | — |

### System Settings

| Capability | Admin | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_settings` | ✅ | ✅ | — | — | — |
| `ik_manage_backup` | ✅ | — | — | — | — |
| `ik_manage_license` | ✅ | — | — | — | — |

---

## Role Summary

| Role | Capabilities | Typical User |
|------|:---:|-------------|
| **Administrator** | 62 | School owner, IT manager |
| **Campus Admin** | 48 | Campus principal, branch head |
| **Teacher** | 14 | Classroom teacher |
| **Finance Manager** | 16 | Accountant, finance officer |
| **Parent** | 4 | Student's parent/guardian |

---

## Checking Capabilities in Code

### WordPress Standard

```php
if (current_user_can('ik_manage_payroll')) {
    // Allow access
}
```

### InstitutionKit Pattern

```php
// Single capability
$this->verify_access('ik_enter_exam_results');

// Any of multiple capabilities
$this->verify_access(['ik_manage_payroll', 'ik_view_payroll_reports']);

// Admin override always applies
// manage_options users pass all checks
```

### In Menu Registration

```php
add_submenu_page(
    'ik-dashboard',
    'Payroll & Expenses',
    '💵 Payroll & Expenses',
    ['ik_manage_payroll', 'ik_manage_expenses', 'ik_view_payroll_reports'],
    'ik-payroll-expenses',
    [$this, 'render_dashboard']
);
```

---

## Adding Custom Capabilities

```php
// 1. Add to the master list
add_filter('ik_capabilities_list', function($caps) {
    $caps[] = 'ik_manage_custom_feature';
    return $caps;
});

// 2. Assign to roles
$admin = get_role('administrator');
$admin->add_cap('ik_manage_custom_feature');

// 3. Use in access checks
$this->verify_access('ik_manage_custom_feature');
```

---

## Granting Capabilities to Individual Users

```php
$user = new WP_User($user_id);
$user->add_cap('ik_manage_expenses');
$user->add_cap('ik_view_financial_comparison');

// Remove a capability
$user->remove_cap('ik_manage_expenses');
```
