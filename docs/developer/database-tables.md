```markdown
# Database Tables Reference

InstitutionKit creates 43 custom database tables. This reference provides a quick lookup for developers extending or integrating with the system.

---

## Quick Reference Table

| # | Table Name | Records | Key Indexes |
|---|-----------|---------|-------------|
| 1 | `institutionkit_fee_types` | Fee categories | `fee_type_id` PK |
| 2 | `institutionkit_fee_structures` | Fee structure groups | `structure_id` PK |
| 3 | `institutionkit_fee_structure_items` | Structure line items | `structure_id` KEY |
| 4 | `institutionkit_student_fees` | Student fee assignments | `student_id`, `fee_type_id` KEY |
| 5 | `institutionkit_invoices` | Invoice records | `student_id`, `class_id`, `campus_id` KEY |
| 6 | `institutionkit_invoice_items` | Invoice line items | `invoice_id` KEY |
| 7 | `institutionkit_transactions` | Payment records | `invoice_id`, `campus_id` KEY |
| 8 | `institutionkit_attendance` | Student attendance | `campus_id` KEY |
| 9 | `institutionkit_teacher_attendance` | Legacy teacher attendance | `campus_id` KEY |
| 10 | `institutionkit_staff_attendance` | Staff attendance | UNIQUE `(staff_id, attendance_date)` |
| 11 | `ik_periods` | Grading periods | `period_type`, `academic_year`, `campus_id` KEY |
| 12 | `ik_grades_v2` | Grade records (current) | UNIQUE `(student_id, subject_id, period_id)` |
| 13 | `institutionkit_gradebook` | Grade records (legacy) | `student_id`, `exam_id`, `subject_id` KEY |
| 14 | `ik_grade_scales` | Grade-to-letter mapping | `scale_type`, `campus_id` KEY |
| 15 | `ik_exam_types` | Exam categories | — |
| 16 | `ik_exam_schedules` | Exam date sheets | `exam_type_id`, `class_id`, `exam_date` KEY |
| 17 | `ik_exam_results` | Student results | UNIQUE `(schedule_id, student_id)` |
| 18 | `ik_report_cards` | Generated report cards | `student_id`, `exam_type_id`, `academic_year` KEY |
| 19 | `institutionkit_staff` | Staff/employee records | UNIQUE `employee_code` |
| 20 | `institutionkit_staff_salary_components` | Salary structure | FK to `staff` |
| 21 | `institutionkit_staff_loans` | Staff loans | FK to `staff` |
| 22 | `institutionkit_loan_installments` | Loan repayments | FK to `loans` |
| 23 | `institutionkit_payroll` | Payroll records | UNIQUE `(staff_id, campus_id, payroll_month)` |
| 24 | `institutionkit_expense_heads` | Expense categories | UNIQUE `(head_name, campus_id)` |
| 25 | `institutionkit_expenses` | Expense ledger | `campus_id`, `head_id`, `status` KEY |
| 26 | `institutionkit_expense_approvals` | Approval workflow | UNIQUE `(expense_id, approval_level)` |
| 27 | `institutionkit_expense_budgets` | Budget planning | UNIQUE `(campus_id, head_id, budget_month)` |
| 28 | `institutionkit_campus_collections` | Revenue tracking | `campus_id`, `collection_date` KEY |
| 29 | `institutionkit_campuses` | Campus definitions | UNIQUE `campus_code` |
| 30 | `institutionkit_campus_users` | User-campus assignments | UNIQUE `(user_id, campus_id)` |
| 31 | `institutionkit_campus_transfers` | Transfer audit trail | `(entity_type, entity_id)`, `(from_campus_id, to_campus_id)` KEY |
| 32 | `institutionkit_announcements` | Announcements | `target_audience`, `class_id`, `campus_id` KEY |
| 33 | `institutionkit_events` | Events calendar | `event_type`, `start_date`, `campus_id` KEY |
| 34 | `institutionkit_notifications` | Notification log | `(recipient_type, recipient_id)`, `type` KEY |
| 35 | `institutionkit_meeting_slots` | Meeting availability | `teacher_id`, `slot_date`, `status` KEY |
| 36 | `institutionkit_meeting_bookings` | Meeting bookings | `slot_id`, `student_id`, `status` KEY |
| 37 | `institutionkit_meeting_topics` | Discussion topics | — |
| 38 | `institutionkit_homework` | Homework assignments | `class_id`, `due_date`, `status`, `campus_id` KEY |
| 39 | `institutionkit_homework_submissions` | Homework submissions | `homework_id`, `student_id`, `status` KEY |
| 40 | `institutionkit_certificates` | Certificates | — |
| 41 | `institutionkit_parent_child` | Parent-student links | UNIQUE `(parent_id, student_id)` |
| 42 | `institutionkit_teacher_comments` | Teacher comments | `student_id`, `teacher_id` KEY |
| 43 | `institutionkit_ratings` | Performance ratings | `student_id` KEY |
| 44 | `ik_student_promotions` | Student promotions | `student_id`, `from_class_id`, `to_class_id`, `academic_year` KEY |
| 45 | `institutionkit_performance` | Performance summaries | `student_id`, `exam_id` KEY |
| 46 | `institutionkit_email_log` | Email delivery log | `email_type`, `sent_at` KEY |

---

## Table Prefix

All tables use the WordPress table prefix:

```php
global $wpdb;
$table = $wpdb->prefix . 'institutionkit_invoices';
// Results in: wp_institutionkit_invoices
```

For multisite, each site gets its own set of tables with the site-specific prefix (e.g., `wp_2_institutionkit_invoices`).

---

## Key Relationships

```
institutionkit_campuses
    └── campus_id used in 30+ tables

institutionkit_staff
    ├── institutionkit_staff_attendance (staff_id)
    ├── institutionkit_staff_salary_components (staff_id)
    ├── institutionkit_staff_loans (staff_id)
    ├── institutionkit_payroll (staff_id)
    └── institutionkit_meeting_slots (teacher_id → staff_id)

wp_posts (ik_student)
    ├── institutionkit_attendance (student_id → ID)
    ├── institutionkit_invoices (student_id → ID)
    ├── institutionkit_gradebook / ik_grades_v2 (student_id → ID)
    ├── ik_exam_results (student_id → ID)
    ├── ik_report_cards (student_id → ID)
    ├── institutionkit_parent_child (student_id → ID)
    └── institutionkit_teacher_comments (student_id → ID)

institutionkit_invoices
    ├── institutionkit_invoice_items (invoice_id)
    └── institutionkit_transactions (invoice_id)

ik_exam_schedules
    └── ik_exam_results (schedule_id)
```

---

## Important Unique Constraints

| Table | Constraint | Prevents |
|-------|-----------|---------|
| `institutionkit_staff_attendance` | `(staff_id, attendance_date)` | Duplicate attendance per day |
| `ik_grades_v2` | `(student_id, subject_id, period_id)` | Duplicate grades per period |
| `ik_exam_results` | `(schedule_id, student_id)` | Duplicate results per exam |
| `institutionkit_payroll` | `(staff_id, campus_id, payroll_month)` | Duplicate payroll per month |
| `institutionkit_expense_budgets` | `(campus_id, head_id, budget_month)` | Duplicate budgets per month |
| `institutionkit_campus_users` | `(user_id, campus_id)` | Duplicate user assignments |
| `institutionkit_parent_child` | `(parent_id, student_id)` | Duplicate parent-student links |

---

## Campus ID Pattern

Most transactional tables include `campus_id INT DEFAULT 1` for multi-campus data partitioning:

```sql
-- Always filter by campus
SELECT * FROM wp_institutionkit_invoices WHERE campus_id = 3;

-- Or use the campus helper
$where = IK_Campus_Manager::get_campus_where_clause('i');
$sql = "SELECT * FROM wp_institutionkit_invoices i WHERE 1=1 {$where}";
```

---

## Checking Table Existence

```php
global $wpdb;
$table_name = $wpdb->prefix . 'institutionkit_invoices';

$exists = $wpdb->get_var("SHOW TABLES LIKE '{$table_name}'") === $table_name;

if ($exists) {
    // Table exists — safe to query
}
```

---

## Listing All InstitutionKit Tables

```php
global $wpdb;
$tables = $wpdb->get_results(
    "SELECT TABLE_NAME FROM information_schema.TABLES 
     WHERE TABLE_SCHEMA = DATABASE() 
       AND TABLE_NAME LIKE '{$wpdb->prefix}institutionkit%'
        OR TABLE_NAME LIKE '{$wpdb->prefix}ik_%'
     ORDER BY TABLE_NAME"
);

foreach ($tables as $table) {
    echo $table->TABLE_NAME . "\n";
}
```
