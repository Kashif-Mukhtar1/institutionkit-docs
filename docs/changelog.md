```markdown
# Changelog

All notable changes to InstitutionKit are documented here.

---

## Version 1.2.1 (Current)

**Release Date:** 2026

### Changes

- License banner rendering improvements
- Campus switcher styling enhancements
- Dashboard performance optimizations
- Various bug fixes and stability improvements

---

## Version 1.2.0

**Release Date:** 2025

### Major Changes

#### Teacher CPT Migration to Staff Table

The legacy `ik_teacher` custom post type has been **completely removed**. All teacher data now resides in the `institutionkit_staff` table managed by the Payroll & Expenses module.

**Migration details:**

- All `ik_teacher` posts migrated to `institutionkit_staff` rows
- Teacher attendance migrated to `institutionkit_staff_attendance`
- Teacher comment references updated from post IDs to staff IDs
- Meeting slot references updated from post IDs to staff IDs
- Migrated posts flagged with `_ik_migrated_to_staff` meta key

**Post-migration verification:**

```sql
-- Check migration status
SELECT COUNT(*) as migrated FROM wp_postmeta WHERE meta_key = '_ik_migrated_to_staff';

-- Verify staff table
SELECT COUNT(*) FROM wp_institutionkit_staff WHERE role LIKE 'teacher%';
```

### New Features

- **Payroll Module**: Complete payroll processing with automatic generation from attendance
- **Expense Management**: Full expense ledger with multi-level approval workflow
- **Budget Planning**: Set and track budgets per expense head per campus
- **Staff Loans**: Loan management with interest calculation and automatic payroll deduction
- **Payslips**: Detailed HTML and PDF payslips with attendance summary and tax calculation
- **Salary Components**: Flexible salary structure with custom earnings and deductions
- **Multi-Campus Financial Comparison**: Side-by-side campus financial analysis
- **Financial Alerts**: Automatic alerts for budget overruns, anomalies, and missing expenses
- **Collections Tracking**: Revenue tracking by campus and source

### Enhancements

- Enhanced dashboard with campus overview cards
- Attendance percentage tracking on student profiles
- Teacher comments system with parent responses
- Parent-Teacher meeting management with topic selection
- Certificate generation and public verification
- Automated monthly invoice generation via cron
- Invoice reminder emails for overdue payments
- Staff photo upload with WordPress media library integration
- Emergency contact display for absent staff

### Database Changes

Added 12 new tables:

| Table | Purpose |
|-------|---------|
| `institutionkit_staff` | Central staff records |
| `institutionkit_staff_attendance` | Staff daily attendance |
| `institutionkit_staff_salary_components` | Flexible salary structure |
| `institutionkit_staff_loans` | Staff loan records |
| `institutionkit_loan_installments` | Loan repayment tracking |
| `institutionkit_payroll` | Monthly payroll records |
| `institutionkit_expense_heads` | Expense categories |
| `institutionkit_expenses` | Expense ledger |
| `institutionkit_expense_approvals` | Multi-level approval workflow |
| `institutionkit_expense_budgets` | Budget planning |
| `institutionkit_campus_collections` | Revenue tracking |
| `ik_student_promotions` | Promotion history |

---

## Version 1.1.0

**Release Date:** 2024

### New Features

- Multi-campus support with campus switcher
- Campus admin role with data isolation
- Gradebook Pro (v2) with multi-period grading
- Grading periods with type and weight system
- Grade scales with campus-specific overrides
- Exam Pro module with full workflow
- Report card generation with PDF export
- Homework module with submissions
- Parent-Teacher meeting slots
- Meeting topics management
- Announcements and events system
- Notification logging
- Student performance summaries
- Student star ratings system
- Campus transfer logging
- Parent-child linking system
- Teacher comments with parent responses
- Certificate management

### Database Changes

Added 20+ new tables including:

- `ik_periods` — Grading periods
- `ik_grades_v2` — Enhanced gradebook
- `ik_grade_scales` — Grade configuration
- `ik_exam_types` — Exam categories
- `ik_exam_schedules` — Exam date sheets
- `ik_exam_results` — Student results
- `ik_report_cards` — Generated cards
- `institutionkit_homework` — Assignments
- `institutionkit_homework_submissions` — Submissions
- `institutionkit_meeting_slots` — Availability
- `institutionkit_meeting_bookings` — Bookings
- `institutionkit_meeting_topics` — Topics
- `institutionkit_announcements` — Announcements
- `institutionkit_events` — Events
- `institutionkit_notifications` — Notifications
- `institutionkit_campuses` — Campus definitions
- `institutionkit_campus_users` — Assignments
- `institutionkit_campus_transfers` — Transfers
- `institutionkit_parent_child` — Parent links
- `institutionkit_teacher_comments` — Comments
- `institutionkit_certificates` — Certificates

---

## Version 1.0.0

**Release Date:** 2023

### Initial Release

- Student management (CPT with meta fields)
- Teacher management (CPT — now deprecated in 1.2.0)
- Class and section management
- Subject taxonomy
- Fee types and structures
- Student fee assignment
- Invoice generation and payment tracking
- Student attendance tracking
- Basic gradebook
- Email notifications
- Basic dashboard with statistics
- Settings page
- Backup and restore
- WordPress role integration
- Default grade scales
- Default meeting topics
- Default expense heads

### Initial Database Tables

- `institutionkit_fee_types`
- `institutionkit_fee_structures`
- `institutionkit_fee_structure_items`
- `institutionkit_student_fees`
- `institutionkit_invoices`
- `institutionkit_invoice_items`
- `institutionkit_transactions`
- `institutionkit_attendance`
- `institutionkit_teacher_attendance`
- `institutionkit_gradebook`
- `institutionkit_email_log`

---

## Upgrade Guide

### Upgrading from 1.1.x to 1.2.x

1. **Backup your database** before upgrading
2. Deactivate InstitutionKit
3. Upload and activate version 1.2.x
4. The teacher-to-staff migration runs automatically
5. Verify migration at **InstitutionKit → Settings → System Status**
6. Review staff records at **Payroll & Expenses → Staff**

### Upgrading from 1.0.x to 1.1.x

1. Backup your database
2. Update the plugin
3. New tables are created automatically
4. Configure grading periods at **Grading Periods**
5. Set up multi-campus at **Campuses** (if needed)

---

## Compatibility

| InstitutionKit Version | WordPress | PHP |
|----------------------|-----------|-----|
| 1.2.x | 5.8+ | 7.4+ |
| 1.1.x | 5.6+ | 7.4+ |
| 1.0.x | 5.4+ | 7.2+ |

---

## Deprecation Notices

### Removed in 1.2.0

- `ik_teacher` custom post type — use `institutionkit_staff` table
- Teacher post meta fields — use staff table columns
- Legacy teacher attendance — use staff attendance table

### Planned for Future Removal

No current deprecation notices for future versions.
```
