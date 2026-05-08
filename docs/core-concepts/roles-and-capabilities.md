# Roles & Capabilities

InstitutionKit implements a comprehensive role-based access control (RBAC) system with **62 custom capabilities** distributed across **5 user roles**. Every page, every AJAX endpoint, and every database query enforces capability checks before execution.

---

## Capability Architecture

### Design Principles

| Principle | Implementation |
|-----------|---------------|
| **Granular Control** | Capabilities are specific: `ik_enter_exam_results`, not generic `edit_exams` |
| **Module-Aligned** | Capabilities mirror the module structure: payroll capabilities are distinct from exam capabilities |
| **Role-Appropriate** | Teachers receive teaching capabilities only. Parents receive viewing capabilities only. |
| **Admin Override** | `manage_options` (WordPress administrator capability) bypasses all checks |
| **Campus-Scoped** | Campus-specific capabilities (`ik_manage_campus_students`) enforce data boundaries |

---

## The Five Roles

### Role Comparison Matrix

| Capability Area | Administrator | Campus Admin | Teacher | Finance Manager | Parent |
|----------------|:---:|:---:|:---:|:---:|:---:|
| **Total Capabilities** | 62 | 48 | 14 | 16 | 4 |
| Student Management | Full | Full | View | — | — |
| Staff Management | Full | Attendance | — | — | — |
| Fee Management | Full | View + Payments | — | Full | View Own |
| Exam Management | Full | Entry + Cards | Entry + Cards | — | — |
| Payroll Processing | Full | Reports | — | Full | — |
| Expense Management | Full | Basic | — | Full | — |
| Budget Planning | Full | — | — | Full | — |
| Financial Reports | All Campuses | Campus Only | — | All | — |
| Gradebook | Full | Full | Full | — | View Own |
| Attendance Marking | Full | Full | Student Only | — | — |
| Homework | Full | Full | Assign + View | — | — |
| Meeting Slots | Full | Full | Create + View | — | Book Own |
| System Settings | Full | Campus Settings | — | — | — |
| Backups | Full | — | — | — | — |

---

## Complete Capability Reference

### Student Capabilities

| Capability | Administrator | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_students` | :material-check: | :material-check: | — | — | — |
| `ik_view_students` | :material-check: | :material-check: | :material-check: | — | — |
| `ik_edit_students` | :material-check: | :material-check: | — | — | — |
| `ik_delete_students` | :material-check: | — | — | — | — |
| `ik_manage_admissions` | :material-check: | :material-check: | — | — | — |

### Staff Capabilities

| Capability | Administrator | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_staff` | :material-check: | :material-check: | — | — | — |
| `ik_manage_staff_attendance` | :material-check: | :material-check: | — | — | — |
| `ik_view_attendance_reports` | :material-check: | :material-check: | — | — | — |

### Fee & Invoice Capabilities

| Capability | Administrator | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_fees` | :material-check: | :material-check: | — | :material-check: | — |
| `ik_view_invoices` | :material-check: | :material-check: | — | :material-check: | :material-check: |
| `ik_generate_invoices` | :material-check: | :material-check: | — | :material-check: | — |
| `ik_record_payments` | :material-check: | :material-check: | — | :material-check: | — |
| `ik_email_invoices` | :material-check: | — | — | :material-check: | — |

### Exam Capabilities

| Capability | Administrator | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_exam_types` | :material-check: | — | — | — | — |
| `ik_manage_exam_schedules` | :material-check: | :material-check: | :material-check: | — | — |
| `ik_enter_exam_results` | :material-check: | :material-check: | :material-check: | — | — |
| `ik_verify_exam_results` | :material-check: | — | — | — | — |
| `ik_publish_exam_results` | :material-check: | — | — | — | — |
| `ik_generate_report_cards` | :material-check: | :material-check: | :material-check: | — | — |
| `ik_view_exam_analytics` | :material-check: | :material-check: | :material-check: | — | — |

### Payroll & Expense Capabilities

| Capability | Administrator | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_payroll` | :material-check: | :material-check: | — | :material-check: | — |
| `ik_approve_payroll` | :material-check: | — | — | :material-check: | — |
| `ik_view_payroll_reports` | :material-check: | :material-check: | — | :material-check: | — |
| `ik_manage_expenses` | :material-check: | :material-check: | — | :material-check: | — |
| `ik_approve_expenses` | :material-check: | — | — | :material-check: | — |
| `ik_view_expense_reports` | :material-check: | :material-check: | — | :material-check: | — |
| `ik_manage_expense_heads` | :material-check: | — | — | :material-check: | — |
| `ik_manage_budgets` | :material-check: | — | — | :material-check: | — |
| `ik_view_financial_comparison` | :material-check: | — | — | :material-check: | — |
| `ik_view_profit_loss` | :material-check: | :material-check: | — | :material-check: | — |
| `ik_view_all_campus_finances` | :material-check: | — | — | — | — |
| `ik_export_financial_data` | :material-check: | — | — | :material-check: | — |

### Gradebook Capabilities

| Capability | Administrator | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_gradebook` | :material-check: | :material-check: | :material-check: | — | — |
| `ik_view_gradebook` | :material-check: | :material-check: | :material-check: | — | :material-check: |

### Homework Capabilities

| Capability | Administrator | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_assign_homework` | :material-check: | :material-check: | :material-check: | — | — |
| `ik_edit_homework` | :material-check: | — | :material-check: | — | — |
| `ik_delete_homework` | :material-check: | — | — | — | — |
| `ik_view_all_homework` | :material-check: | :material-check: | :material-check: | — | — |
| `ik_grade_submissions` | :material-check: | :material-check: | :material-check: | — | — |

### Meeting Capabilities

| Capability | Administrator | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_meeting_slots` | :material-check: | :material-check: | :material-check: | — | — |
| `ik_view_meeting_bookings` | :material-check: | :material-check: | :material-check: | — | — |
| `ik_manage_meeting_bookings` | :material-check: | :material-check: | — | — | — |
| `ik_book_meeting_slots` | :material-check: | — | — | — | :material-check: |
| `ik_view_own_meetings` | :material-check: | — | — | — | :material-check: |

### Campus Capabilities

| Capability | Administrator | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_campuses` | :material-check: | — | — | — | — |
| `ik_view_all_campuses` | :material-check: | — | — | — | — |
| `ik_manage_campus` | :material-check: | :material-check: | — | — | — |
| `ik_switch_campus` | :material-check: | — | — | — | — |
| `ik_view_campus_dashboard` | :material-check: | :material-check: | — | — | — |
| `ik_manage_campus_students` | :material-check: | :material-check: | — | — | — |
| `ik_manage_campus_teachers` | :material-check: | :material-check: | — | — | — |
| `ik_manage_campus_fees` | :material-check: | :material-check: | — | — | — |
| `ik_manage_campus_attendance` | :material-check: | :material-check: | — | — | — |
| `ik_manage_campus_gradebook` | :material-check: | :material-check: | — | — | — |
| `ik_manage_campus_settings` | :material-check: | :material-check: | — | — | — |

### Promotion Capabilities

| Capability | Administrator | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_student_promotions` | :material-check: | :material-check: | — | — | — |
| `ik_view_promotion_history` | :material-check: | :material-check: | :material-check: | — | — |
| `ik_bulk_promote_students` | :material-check: | :material-check: | — | — | — |

### System Capabilities

| Capability | Administrator | Campus Admin | Teacher | Finance | Parent |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `ik_manage_settings` | :material-check: | :material-check: | — | — | — |
| `ik_manage_backup` | :material-check: | — | — | — | — |
| `ik_manage_license` | :material-check: | — | — | — | — |

---

## How Capability Checks Work

### Page-Level Enforcement

Every admin page verifies capabilities before rendering:

```php
// From IK_Admin_Menu
public function render_exam_result_entry() {
    $this->verify_access('ik_enter_exam_results');
    // ... render page
}

private function verify_access($caps) {
    if (!current_user_can('manage_options') && !current_user_can($caps)) {
        wp_die('You do not have sufficient permissions to access this page.');
    }
}
