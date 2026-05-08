# Database Schema Reference

InstitutionKit creates **40+ custom tables** to manage every aspect of school operations. This reference documents every table, its columns, data types, and relationships.

---

## Schema Architecture

InstitutionKit uses a hybrid storage model:

| Storage Type | Used For | Examples |
|-------------|----------|----------|
| **Custom Tables** | Transactional data, financial records, attendance | `institutionkit_invoices`, `institutionkit_attendance`, `institutionkit_payroll` |
| **WordPress Posts** | Content entities with admin UIs | `ik_student`, `ik_class`, `ik_exam`, `ik_period` |
| **WordPress Post Meta** | Entity attributes, campus assignments | `_ik_campus_id`, `_ik_student_class_id`, `_ik_roll_number` |
| **WordPress Taxonomies** | Classifications, groupings | `ik_subject`, `ik_section` |

---

## Table Organization

```
wp_posts / wp_postmeta
├── ik_student (Students)
├── ik_class (Classes)
├── ik_exam (Exam posts - legacy)
├── ik_certificate (Certificate templates)
└── ik_period (Grading periods)

wp_terms / wp_term_taxonomy
├── ik_subject (Subjects taxonomy)
└── ik_section (Sections taxonomy)

Custom Tables
├── Fee Management (5 tables)
├── Invoicing & Payments (3 tables)
├── Attendance (2 tables)
├── Gradebook Pro (3 tables)
├── Exams Pro (4 tables)
├── Staff & Payroll (7 tables)
├── Expenses (4 tables)
├── Campus Management (3 tables)
├── Communications (3 tables)
├── Meetings (3 tables)
├── Homework (2 tables)
└── Other (certificates, promotions, ratings)
```

---

## Fee Management

### `institutionkit_fee_types`

Defines categories of fees charged to students.

| Column | Type | Description |
|--------|------|-------------|
| `fee_type_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `fee_name` | VARCHAR(255) | Name, e.g., "Tuition Fee", "Transport Fee" |

### `institutionkit_fee_structures`

Named groups of fee types with amounts.

| Column | Type | Description |
|--------|------|-------------|
| `structure_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `structure_name` | VARCHAR(255) | Name, e.g., "Primary Monthly Fees" |

### `institutionkit_fee_structure_items`

Links fee types to structures with amounts.

| Column | Type | Description |
|--------|------|-------------|
| `item_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `structure_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_fee_structures` |
| `fee_type_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_fee_types` |
| `amount` | DECIMAL(10,2) | Fee amount |

**Index:** `KEY structure_id (structure_id)`

### `institutionkit_student_fees`

Individual student fee assignments with optional date ranges.

| Column | Type | Description |
|--------|------|-------------|
| `student_fee_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `student_id` | BIGINT(20) UNSIGNED | Student post ID |
| `fee_type_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_fee_types` |
| `amount` | DECIMAL(10,2) | Assigned fee amount |
| `start_date` | DATE | Optional start date |
| `end_date` | DATE | Optional end date |
| `notes` | VARCHAR(255) | Optional notes |

**Indexes:** `KEY student_id`, `KEY fee_type_id`

---

## Invoicing & Payments

### `institutionkit_invoices`

Core invoice records with payment tracking.

| Column | Type | Description |
|--------|------|-------------|
| `invoice_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `student_id` | BIGINT(20) UNSIGNED | Student post ID |
| `class_id` | BIGINT(20) UNSIGNED | Class post ID |
| `title` | VARCHAR(255) | Invoice title/description |
| `total_amount` | DECIMAL(10,2) | Total invoice amount |
| `amount_paid` | DECIMAL(10,2) DEFAULT 0.00 | Cumulative payments received |
| `due_date` | DATE | Payment deadline |
| `status` | VARCHAR(20) DEFAULT 'unpaid' | `unpaid`, `partial`, `paid` |
| `created_at` | TIMESTAMP DEFAULT CURRENT_TIMESTAMP | Creation timestamp |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

**Indexes:** `KEY student_id`, `KEY class_id`, `KEY campus_id`

### `institutionkit_invoice_items`

Line items within an invoice.

| Column | Type | Description |
|--------|------|-------------|
| `item_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `invoice_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_invoices` |
| `fee_type_name` | VARCHAR(255) | Description of the fee |
| `amount` | DECIMAL(10,2) | Line item amount |

**Index:** `KEY invoice_id`

### `institutionkit_transactions`

Payment records against invoices.

| Column | Type | Description |
|--------|------|-------------|
| `transaction_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `invoice_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_invoices` |
| `amount` | DECIMAL(10,2) | Payment amount |
| `payment_method` | VARCHAR(50) | Cash, Bank, Mobile, etc. |
| `payment_date` | DATE | Date of payment |
| `notes` | TEXT | Optional notes |
| `recorded_by` | BIGINT(20) UNSIGNED | User ID who recorded the payment |
| `created_at` | TIMESTAMP DEFAULT CURRENT_TIMESTAMP | Record timestamp |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

**Indexes:** `KEY invoice_id`, `KEY campus_id`

---

## Attendance

### `institutionkit_attendance`

Student daily attendance records.

| Column | Type | Description |
|--------|------|-------------|
| `attendance_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `student_id` | BIGINT(20) UNSIGNED | Student post ID |
| `class_id` | BIGINT(20) UNSIGNED | Class post ID |
| `attendance_date` | DATE | Date of record |
| `status` | VARCHAR(20) DEFAULT 'present' | `present`, `absent`, `late`, `leave` |
| `remarks` | TEXT | Optional notes |
| `marked_by` | BIGINT(20) UNSIGNED | User ID who marked attendance |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

**Index:** `KEY campus_id`

### `institutionkit_staff_attendance`

Staff daily attendance with time tracking.

| Column | Type | Description |
|--------|------|-------------|
| `attendance_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `staff_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_staff` |
| `campus_id` | BIGINT(20) UNSIGNED | Campus partition key |
| `attendance_date` | DATE | Date of record |
| `check_in` | TIME | Check-in time |
| `check_out` | TIME | Check-out time |
| `hours_worked` | DECIMAL(5,2) | Auto-calculated hours |
| `lectures_count` | INT DEFAULT 0 | Lectures delivered (for per-lecture staff) |
| `status` | VARCHAR(20) DEFAULT 'present' | `present`, `absent`, `half_day`, `leave` |
| `leave_type` | VARCHAR(50) | Type of leave if on leave |
| `is_approved` | TINYINT(1) DEFAULT 1 | Approval status |
| `remarks` | TEXT | Optional notes |

**Unique Key:** `UNIQUE KEY unique_daily_attendance (staff_id, attendance_date)` — one record per staff per day

---

## Gradebook Pro (v2)

### `ik_grades_v2`

Multi-period grade records. This is the **current** grade storage table.

| Column | Type | Description |
|--------|------|-------------|
| `grade_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `student_id` | BIGINT(20) UNSIGNED | Student post ID |
| `subject_id` | BIGINT(20) UNSIGNED | Subject term ID |
| `class_id` | BIGINT(20) UNSIGNED | Class post ID |
| `period_id` | BIGINT(20) UNSIGNED | FK to `ik_periods` (post ID) |
| `period_type` | VARCHAR(20) | `weekly`, `monthly`, `quarterly`, `yearly`, `exam` |
| `marks_obtained` | DECIMAL(6,2) | Scored marks |
| `max_marks` | DECIMAL(6,2) DEFAULT 100.00 | Maximum possible marks |
| `grade_letter` | VARCHAR(5) | Auto-calculated letter grade |
| `remarks` | TEXT | Teacher remarks |
| `teacher_id` | BIGINT(20) UNSIGNED | Staff ID of entering teacher |
| `created_at` | DATETIME DEFAULT CURRENT_TIMESTAMP | Entry timestamp |
| `updated_at` | DATETIME ON UPDATE CURRENT_TIMESTAMP | Last modified |

**Unique Key:** `UNIQUE KEY unique_grade (student_id, subject_id, period_id)` — one grade per student per subject per period

**Indexes:** `KEY student_id`, `KEY subject_id`, `KEY class_id`, `KEY period_id`

### `ik_periods`

Grading period definitions (synced with `ik_period` CPT).

| Column | Type | Description |
|--------|------|-------------|
| `period_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `title` | VARCHAR(255) | Period name |
| `period_type` | VARCHAR(20) DEFAULT 'monthly' | `weekly`, `monthly`, `quarterly`, `yearly`, `exam` |
| `start_date` | DATE | Period start |
| `end_date` | DATE | Period end |
| `academic_year` | VARCHAR(9) | e.g., "2025-2026" |
| `weight` | DECIMAL(5,2) DEFAULT 1.00 | Weight for averaging |
| `is_published` | TINYINT(1) DEFAULT 0 | Visible to students/parents |
| `campus_id` | BIGINT(20) UNSIGNED | Campus partition key |

### `ik_grade_scales`

Grade-to-letter/GPA mapping tables.

| Column | Type | Description |
|--------|------|-------------|
| `scale_id` | INT(11) UNSIGNED AUTO_INCREMENT | Primary key |
| `min_percent` | DECIMAL(5,2) | Lower bound (inclusive) |
| `max_percent` | DECIMAL(5,2) | Upper bound (inclusive) |
| `letter_grade` | VARCHAR(5) | Grade letter, e.g., "A+", "B", "F" |
| `gpa_points` | DECIMAL(3,2) | GPA value |
| `scale_type` | VARCHAR(10) DEFAULT 'default' | `default` or campus-specific |
| `campus_id` | BIGINT(20) UNSIGNED | Campus override (NULL = global) |

**Default scales inserted on activation:**

| Min % | Max % | Grade | GPA |
|-------|-------|-------|-----|
| 90.00 | 100.00 | A+ | 4.00 |
| 85.00 | 89.99 | A | 4.00 |
| 80.00 | 84.99 | A- | 3.70 |
| 77.00 | 79.99 | B+ | 3.30 |
| 73.00 | 76.99 | B | 3.00 |
| 70.00 | 72.99 | B- | 2.70 |
| 67.00 | 69.99 | C+ | 2.30 |
| 63.00 | 66.99 | C | 2.00 |
| 60.00 | 62.99 | C- | 1.70 |
| 50.00 | 59.99 | D | 1.00 |
| 0.00 | 49.99 | F | 0.00 |

---

## Exams Pro

### `ik_exam_types`

Exam categories with default marks and weights.

| Column | Type | Description |
|--------|------|-------------|
| `exam_type_id` | BIGINT UNSIGNED AUTO_INCREMENT | Primary key |
| `exam_name` | VARCHAR(100) | Display name |
| `exam_category` | VARCHAR(30) DEFAULT 'term' | `term`, `monthly`, `board`, `quiz`, `midterm`, `final`, `assessment` |
| `max_marks` | DECIMAL(6,2) DEFAULT 100.00 | Default max marks |
| `weight_percentage` | DECIMAL(5,2) DEFAULT 100.00 | Weight in final calculation |
| `is_active` | TINYINT(1) DEFAULT 1 | Active flag |
| `campus_id` | BIGINT UNSIGNED | Campus partition key |

### `ik_exam_schedules`

Individual exam scheduling with room and invigilator.

| Column | Type | Description |
|--------|------|-------------|
| `schedule_id` | BIGINT UNSIGNED AUTO_INCREMENT | Primary key |
| `exam_type_id` | BIGINT UNSIGNED | FK to `ik_exam_types` |
| `class_id` | BIGINT UNSIGNED | Class post ID |
| `subject_id` | BIGINT UNSIGNED | Subject term ID |
| `exam_date` | DATE | Date of exam |
| `start_time` | TIME | Start time |
| `end_time` | TIME | End time |
| `room_number` | VARCHAR(50) | Room/location |
| `invigilator_id` | BIGINT UNSIGNED | Staff ID of invigilator |
| `max_marks` | DECIMAL(6,2) | Max marks for this exam |
| `passing_marks` | DECIMAL(6,2) | Passing threshold |
| `is_published` | TINYINT(1) DEFAULT 0 | Published to students |
| `campus_id` | BIGINT UNSIGNED | Campus partition key |
| `created_by` | BIGINT UNSIGNED | User who created the schedule |

### `ik_exam_results`

Individual student results per exam schedule.

| Column | Type | Description |
|--------|------|-------------|
| `result_id` | BIGINT UNSIGNED AUTO_INCREMENT | Primary key |
| `schedule_id` | BIGINT UNSIGNED | FK to `ik_exam_schedules` |
| `student_id` | BIGINT UNSIGNED | Student post ID |
| `marks_obtained` | DECIMAL(6,2) | Scored marks (NULL if absent) |
| `marks_absent` | TINYINT(1) DEFAULT 0 | Student was absent |
| `grade_letter` | VARCHAR(5) | Auto-calculated grade |
| `remarks` | TEXT | Optional remarks |
| `entered_by` | BIGINT UNSIGNED | Teacher who entered marks |
| `verified_by` | BIGINT UNSIGNED | Verifier (NULL if unverified) |
| `is_verified` | TINYINT(1) DEFAULT 0 | Verification status |
| `status` | VARCHAR(20) DEFAULT 'draft' | `draft`, `submitted`, `verified`, `published` |
| `campus_id` | BIGINT UNSIGNED | Campus partition key |

**Unique Key:** `UNIQUE KEY unique_result (schedule_id, student_id)` — one result per student per exam schedule

**Workflow:** `draft` → `submitted` → `verified` → `published`

### `ik_report_cards`

Generated report card records with PDF URLs.

| Column | Type | Description |
|--------|------|-------------|
| `card_id` | BIGINT UNSIGNED AUTO_INCREMENT | Primary key |
| `student_id` | BIGINT UNSIGNED | Student post ID |
| `exam_type_id` | BIGINT UNSIGNED | Exam type |
| `academic_year` | VARCHAR(9) | Academic year |
| `total_marks` | DECIMAL(8,2) DEFAULT 0 | Total possible marks |
| `obtained_marks` | DECIMAL(8,2) DEFAULT 0 | Total scored marks |
| `percentage` | DECIMAL(5,2) DEFAULT 0 | Overall percentage |
| `gpa` | DECIMAL(3,2) | Calculated GPA |
| `grade_letter` | VARCHAR(5) | Overall grade |
| `rank_in_class` | INT | Class ranking |
| `attendance_percentage` | DECIMAL(5,2) | Attendance during exam period |
| `teacher_remarks` | TEXT | Teacher comments |
| `principal_remarks` | TEXT | Principal comments |
| `pdf_url` | VARCHAR(500) | Generated PDF file path |
| `is_published` | TINYINT(1) DEFAULT 0 | Published to students/parents |
| `campus_id` | BIGINT UNSIGNED | Campus partition key |

---

## Staff & Payroll

### `institutionkit_staff`

Central staff/employee records.

| Column | Type | Description |
|--------|------|-------------|
| `staff_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `user_id` | BIGINT(20) UNSIGNED | Linked WordPress user (NULL if no account) |
| `employee_code` | VARCHAR(50) UNIQUE | Auto-generated: `EMP{YY}{XXXX}` |
| `full_name` | VARCHAR(255) | Full legal name |
| `email` | VARCHAR(255) | Email address |
| `phone` | VARCHAR(50) | Contact number |
| `role` | VARCHAR(30) | `teacher_permanent`, `teacher_visiting`, `campus_head`, `office_staff`, `admin_roving`, `maintenance`, `security`, `other` |
| `primary_campus_id` | BIGINT(20) UNSIGNED | Primary campus assignment |
| `assigned_campuses` | LONGTEXT | JSON for multi-campus staff |
| `bank_name` | VARCHAR(255) | Bank for salary transfer |
| `bank_account_number` | VARCHAR(255) | Account number |
| `bank_ifsc` | VARCHAR(20) | Branch code |
| `tax_id` | VARCHAR(50) | Tax ID / CNIC |
| `contract_type` | VARCHAR(20) DEFAULT 'monthly_fixed' | `monthly_fixed`, `hourly`, `per_lecture` |
| `base_salary` | DECIMAL(12,2) DEFAULT 0.00 | Monthly fixed salary |
| `hourly_rate` | DECIMAL(10,2) | Rate per hour |
| `lecture_rate` | DECIMAL(10,2) | Rate per lecture |
| `standard_hours_per_month` | INT DEFAULT 160 | Standard working hours |
| `standard_lectures_per_month` | INT | Standard lecture count |
| `join_date` | DATE | Employment start |
| `contract_end_date` | DATE | Contract expiry |
| `termination_date` | DATE | Termination date |
| `employment_status` | VARCHAR(20) DEFAULT 'active' | `active`, `on_leave`, `terminated`, `retired` |
| `qualification` | TEXT | Educational qualifications |
| `experience_years` | INT DEFAULT 0 | Years of experience |
| `emergency_contact_name` | VARCHAR(255) | Emergency contact |
| `emergency_contact_phone` | VARCHAR(50) | Emergency phone |
| `address` | TEXT | Physical address |
| `photo_id` | BIGINT(20) UNSIGNED | WordPress attachment ID for photo |

### `institutionkit_staff_salary_components`

Flexible salary structure — earnings and deductions per staff member.

| Column | Type | Description |
|--------|------|-------------|
| `id` | INT AUTO_INCREMENT | Primary key |
| `staff_id` | INT | FK to `institutionkit_staff` |
| `component_type` | ENUM('earnings','deductions') | Component category |
| `label` | VARCHAR(100) | Description (e.g., "House Rent Allowance") |
| `amount` | DECIMAL(12,2) DEFAULT 0.00 | Amount |
| `is_taxable` | TINYINT(1) DEFAULT 0 | Subject to income tax |
| `is_active` | TINYINT(1) DEFAULT 1 | Active flag |

**Foreign Key:** `FOREIGN KEY (staff_id) REFERENCES institutionkit_staff(staff_id) ON DELETE CASCADE`

### `institutionkit_staff_loans`

Staff loan records with interest calculation.

| Column | Type | Description |
|--------|------|-------------|
| `loan_id` | INT AUTO_INCREMENT | Primary key |
| `staff_id` | INT | FK to `institutionkit_staff` |
| `loan_date` | DATE | Loan issue date |
| `principal_amount` | DECIMAL(12,2) | Original loan amount |
| `annual_interest_rate` | DECIMAL(5,2) DEFAULT 0.00 | Annual interest percentage |
| `tenure_months` | INT | Repayment period |
| `monthly_deduction` | DECIMAL(12,2) | Auto-calculated EMI |
| `remaining_balance` | DECIMAL(12,2) | Outstanding balance |
| `status` | ENUM('active','paid','written_off') DEFAULT 'active' | Loan status |

### `institutionkit_loan_installments`

Individual loan repayment tracking.

| Column | Type | Description |
|--------|------|-------------|
| `installment_id` | INT AUTO_INCREMENT | Primary key |
| `loan_id` | INT | FK to `institutionkit_staff_loans` |
| `payroll_id` | INT | FK to `institutionkit_payroll` (which payroll deducted it) |
| `due_date` | DATE | Due date |
| `amount` | DECIMAL(12,2) | Installment amount |
| `principal_paid` | DECIMAL(12,2) DEFAULT 0.00 | Principal portion |
| `interest_paid` | DECIMAL(12,2) DEFAULT 0.00 | Interest portion |
| `paid_date` | DATETIME | Payment timestamp |

### `institutionkit_payroll`

Monthly payroll records per staff member.

| Column | Type | Description |
|--------|------|-------------|
| `payroll_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `staff_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_staff` |
| `campus_id` | BIGINT(20) UNSIGNED | Campus partition key |
| `payroll_month` | DATE | Month (stored as first day: `2025-06-01`) |
| `gross_pay` | DECIMAL(12,2) DEFAULT 0.00 | Total earnings |
| `total_deductions` | DECIMAL(12,2) DEFAULT 0.00 | Total deductions |
| `net_pay` | DECIMAL(12,2) DEFAULT 0.00 | Net payable |
| `earnings_json` | LONGTEXT | JSON: breakdown of earnings |
| `deductions_json` | LONGTEXT | JSON: breakdown of deductions |
| `attendance_json` | LONGTEXT | JSON: attendance summary |
| `status` | VARCHAR(20) DEFAULT 'pending' | `pending`, `paid` |
| `created_by` | BIGINT(20) UNSIGNED | User who generated payroll |

**Unique Key:** `UNIQUE KEY unique_monthly_payroll (staff_id, campus_id, payroll_month)`

---

## Expenses

### `institutionkit_expense_heads`

Categories for expense classification.

| Column | Type | Description |
|--------|------|-------------|
| `head_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `head_name` | VARCHAR(255) | Name, e.g., "Building Rent", "Utilities" |
| `head_type` | VARCHAR(20) DEFAULT 'variable' | `fixed`, `variable` |
| `is_system_default` | TINYINT(1) DEFAULT 0 | Pre-installed default head |
| `campus_id` | BIGINT(20) UNSIGNED | NULL = global, specific = campus-only |
| `icon_class` | VARCHAR(100) | Font Awesome icon class |
| `color_code` | VARCHAR(7) | Hex color for charts |
| `is_active` | TINYINT(1) DEFAULT 1 | Active flag |

**Unique Key:** `UNIQUE KEY unique_head_per_campus (head_name, campus_id)`

**Default heads inserted on activation:** Building Rent, Utilities, Maintenance & Repairs, Cleanliness & Sanitation, Security Services, Office Supplies, Staff Welfare, Entertainment & Events, Transportation, IT & Software, Marketing & Advertising, Insurance, Professional Fees, Staff Training, Miscellaneous

### `institutionkit_expenses`

Core expense ledger.

| Column | Type | Description |
|--------|------|-------------|
| `expense_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `campus_id` | BIGINT(20) UNSIGNED | Campus partition key |
| `head_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_expense_heads` |
| `amount` | DECIMAL(12,2) | Total expense amount |
| `paid_by` | VARCHAR(20) DEFAULT 'campus_petty' | `campus_petty`, `central_admin`, `split` |
| `split_ratio` | DECIMAL(5,2) | Campus share percentage (if split) |
| `campus_amount` | DECIMAL(12,2) DEFAULT 0.00 | Campus's share of expense |
| `central_amount` | DECIMAL(12,2) DEFAULT 0.00 | Central's share of expense |
| `description` | TEXT | Expense description |
| `vendor_name` | VARCHAR(255) | Payee/vendor |
| `invoice_number` | VARCHAR(100) | Vendor invoice reference |
| `attachments` | LONGTEXT | JSON: attachment IDs |
| `status` | VARCHAR(20) DEFAULT 'pending_approval' | `pending_approval`, `approved`, `rejected`, `reimbursed` |
| `severity_flag` | VARCHAR(10) DEFAULT 'green' | `green` (routine), `yellow` (important), `red` (emergency) |
| `expense_date` | DATE | Date of expense |
| `payment_date` | DATE | Date payment was made |
| `payment_method` | VARCHAR(50) | Cash, Bank Transfer, Cheque, etc. |
| `approved_by` | BIGINT(20) UNSIGNED | Approver user ID |
| `approved_at` | DATETIME | Approval timestamp |
| `rejection_reason` | TEXT | Reason for rejection |
| `internal_notes` | TEXT | Admin/finance-only notes |
| `created_by` | BIGINT(20) UNSIGNED | Creator user ID |

### `institutionkit_expense_approvals`

Multi-level approval workflow.

| Column | Type | Description |
|--------|------|-------------|
| `approval_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `expense_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_expenses` |
| `approver_id` | BIGINT(20) UNSIGNED | Approver user ID |
| `approval_level` | INT DEFAULT 1 | 1, 2, or 3 based on amount |
| `status` | VARCHAR(20) DEFAULT 'pending' | `pending`, `approved`, `rejected` |
| `comments` | TEXT | Approver comments |
| `action_date` | DATETIME | Action timestamp |

**Approval tiers:** Level 1 (< 10,000), Level 2 (10,000–50,000), Level 3 (> 50,000)

### `institutionkit_expense_budgets`

Monthly budget planning per expense head.

| Column | Type | Description |
|--------|------|-------------|
| `budget_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `campus_id` | BIGINT(20) UNSIGNED | Campus partition key |
| `head_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_expense_heads` |
| `budget_month` | DATE | Budget month (first day) |
| `budget_amount` | DECIMAL(12,2) | Planned amount |
| `actual_amount` | DECIMAL(12,2) DEFAULT 0.00 | Actual spending |
| `variance` | DECIMAL(12,2) DEFAULT 0.00 | budget - actual |
| `variance_percentage` | DECIMAL(5,2) DEFAULT 0.00 | Percentage used |
| `notes` | TEXT | Optional notes |

**Unique Key:** `UNIQUE KEY unique_campus_head_month (campus_id, head_id, budget_month)`

---

## Campus Management

### `institutionkit_campuses`

| Column | Type | Description |
|--------|------|-------------|
| `campus_id` | INT AUTO_INCREMENT | Primary key |
| `campus_name` | VARCHAR(255) | Display name |
| `campus_code` | VARCHAR(20) UNIQUE | Unique short code |
| `address` | TEXT | Physical address |
| `phone` | VARCHAR(50) | Contact phone |
| `email` | VARCHAR(100) | Contact email |
| `principal_name` | VARCHAR(255) | Principal/head name |
| `is_active` | TINYINT(1) DEFAULT 1 | Active flag |
| `settings` | LONGTEXT | JSON: campus-specific settings |

### `institutionkit_campus_users`

| Column | Type | Description |
|--------|------|-------------|
| `id` | INT AUTO_INCREMENT | Primary key |
| `user_id` | BIGINT(20) | WordPress user ID |
| `campus_id` | INT | FK to `institutionkit_campuses` |
| `role` | VARCHAR(50) DEFAULT 'campus_admin' | Role within campus |
| `assigned_at` | DATETIME DEFAULT CURRENT_TIMESTAMP | Assignment timestamp |

**Unique Key:** `UNIQUE KEY user_campus (user_id, campus_id)`

### `institutionkit_campus_transfers`

| Column | Type | Description |
|--------|------|-------------|
| `transfer_id` | INT AUTO_INCREMENT | Primary key |
| `entity_type` | VARCHAR(20) | `student` or `staff` |
| `entity_id` | BIGINT(20) | Student/staff ID |
| `entity_name` | VARCHAR(255) | Name at time of transfer |
| `from_campus_id` | INT | Source campus |
| `to_campus_id` | INT | Destination campus |
| `transfer_date` | DATETIME DEFAULT CURRENT_TIMESTAMP | Timestamp |
| `transfer_reason` | TEXT | Optional reason |
| `transferred_by` | BIGINT(20) | User who performed transfer |
| `status` | VARCHAR(20) DEFAULT 'completed' | `completed`, `pending`, `reversed` |

---

## Communications

### `institutionkit_announcements`

| Column | Type | Description |
|--------|------|-------------|
| `announcement_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `title` | VARCHAR(255) | Announcement title |
| `content` | TEXT | Full content |
| `target_audience` | VARCHAR(50) DEFAULT 'all' | `all`, `teachers`, `parents`, `students` |
| `class_id` | BIGINT(20) UNSIGNED | Optional class filter |
| `created_by` | BIGINT(20) UNSIGNED | Author user ID |
| `is_active` | TINYINT(1) DEFAULT 1 | Active flag |
| `expires_at` | DATE | Auto-expiry date |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

### `institutionkit_events`

| Column | Type | Description |
|--------|------|-------------|
| `event_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `title` | VARCHAR(255) | Event title |
| `description` | TEXT | Event details |
| `event_type` | VARCHAR(50) | `exam`, `holiday`, `meeting`, `event`, `fee_due` |
| `start_date` | DATETIME | Start date/time |
| `end_date` | DATETIME | End date/time |
| `location` | VARCHAR(255) | Venue |
| `created_by` | BIGINT(20) UNSIGNED | Creator user ID |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

### `institutionkit_notifications`

| Column | Type | Description |
|--------|------|-------------|
| `notification_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `recipient_type` | VARCHAR(50) | `parent`, `teacher`, `student` |
| `recipient_id` | BIGINT(20) UNSIGNED | Recipient user ID |
| `type` | VARCHAR(50) | Notification type |
| `subject` | VARCHAR(255) | Subject line |
| `message` | TEXT | Full message body |
| `channel` | VARCHAR(20) | `email`, `sms` |
| `status` | VARCHAR(20) DEFAULT 'sent' | Delivery status |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

---

## Meetings

### `institutionkit_meeting_slots`

Teacher availability slots for parent-teacher meetings.

| Column | Type | Description |
|--------|------|-------------|
| `slot_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `teacher_id` | BIGINT(20) UNSIGNED | Staff ID (references `institutionkit_staff`) |
| `class_id` | BIGINT(20) UNSIGNED | Optional class restriction |
| `slot_date` | DATE | Date of slot |
| `start_time` | TIME | Start time |
| `end_time` | TIME | End time |
| `duration` | INT(11) DEFAULT 30 | Duration in minutes |
| `max_bookings` | INT(11) DEFAULT 1 | Max concurrent bookings |
| `current_bookings` | INT(11) DEFAULT 0 | Current booking count |
| `location` | VARCHAR(255) DEFAULT 'School Office' | Meeting location |
| `meeting_type` | VARCHAR(50) DEFAULT 'in_person' | `in_person`, `online` |
| `status` | VARCHAR(20) DEFAULT 'available' | `available`, `full`, `cancelled` |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

### `institutionkit_meeting_bookings`

Parent bookings against available slots.

| Column | Type | Description |
|--------|------|-------------|
| `booking_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `slot_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_meeting_slots` |
| `student_id` | BIGINT(20) UNSIGNED | Student post ID |
| `parent_id` | BIGINT(20) UNSIGNED | Parent user ID |
| `teacher_id` | BIGINT(20) UNSIGNED | Staff ID |
| `parent_name` | VARCHAR(255) | Parent name at booking time |
| `parent_email` | VARCHAR(255) | Parent email |
| `parent_phone` | VARCHAR(50) | Parent phone |
| `topics` | TEXT | Selected discussion topics |
| `status` | VARCHAR(20) DEFAULT 'confirmed' | `confirmed`, `cancelled`, `completed` |
| `meeting_notes` | TEXT | Meeting notes |
| `teacher_feedback` | TEXT | Post-meeting feedback |
| `attended` | TINYINT(1) DEFAULT 0 | Attendance flag |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

### `institutionkit_meeting_topics`

Predefined discussion topics for meetings.

| Column | Type | Description |
|--------|------|-------------|
| `topic_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `topic_name` | VARCHAR(255) | Topic name |
| `description` | TEXT | Topic description |
| `is_active` | TINYINT(1) DEFAULT 1 | Active flag |
| `sort_order` | INT(11) DEFAULT 0 | Display order |

**Default topics:** Academic Performance, Behavior and Conduct, Attendance Issues, Homework Concerns, Special Needs Support, College/Career Planning, Extra-curricular Activities, Health Concerns, Social Development, General Check-in

---

## Homework

### `institutionkit_homework`

| Column | Type | Description |
|--------|------|-------------|
| `id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `class_id` | BIGINT(20) UNSIGNED | Class post ID |
| `section` | VARCHAR(50) | Optional section filter |
| `subject` | VARCHAR(255) | Subject name |
| `title` | VARCHAR(255) | Homework title |
| `description` | LONGTEXT | Full description |
| `attachment` | VARCHAR(500) | File URL |
| `due_date` | DATE | Submission deadline |
| `is_recurring` | TINYINT(1) DEFAULT 0 | Recurring homework flag |
| `recurring_pattern` | VARCHAR(50) | `daily`, `weekly`, `monthly` |
| `recurring_value` | INT(11) | Interval value |
| `assigned_by` | BIGINT(20) UNSIGNED | Staff ID |
| `assigned_date` | DATETIME DEFAULT CURRENT_TIMESTAMP | Assignment timestamp |
| `status` | VARCHAR(20) DEFAULT 'active' | `active`, `archived` |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

### `institutionkit_homework_submissions`

| Column | Type | Description |
|--------|------|-------------|
| `submission_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `homework_id` | BIGINT(20) UNSIGNED | FK to `institutionkit_homework` |
| `student_id` | BIGINT(20) UNSIGNED | Student post ID |
| `submission_date` | DATETIME DEFAULT CURRENT_TIMESTAMP | Submission timestamp |
| `submission_text` | TEXT | Student's submitted text |
| `attachment` | VARCHAR(500) | Student's file URL |
| `status` | VARCHAR(20) DEFAULT 'submitted' | `submitted`, `graded`, `returned` |
| `grade` | VARCHAR(10) | Assigned grade |
| `teacher_feedback` | TEXT | Feedback from teacher |
| `graded_by` | BIGINT(20) UNSIGNED | Staff ID |
| `graded_date` | DATETIME | Grading timestamp |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

---

## Other Tables

### `institutionkit_certificates`

| Column | Type | Description |
|--------|------|-------------|
| `certificate_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `recipient_id` | BIGINT(20) UNSIGNED | Student or staff ID |
| `recipient_type` | ENUM('student','teacher') | Recipient category |
| `certificate_type` | ENUM('leaving','character','achievement','employment') | Certificate type |
| `issue_date` | DATE | Issue date |
| `certificate_number` | VARCHAR(50) | Unique certificate number |

### `ik_student_promotions`

| Column | Type | Description |
|--------|------|-------------|
| `promotion_id` | BIGINT UNSIGNED AUTO_INCREMENT | Primary key |
| `student_id` | BIGINT UNSIGNED | Student post ID |
| `from_class_id` | BIGINT UNSIGNED | Previous class |
| `to_class_id` | BIGINT UNSIGNED | New class |
| `promotion_date` | DATE | Promotion date |
| `academic_year` | VARCHAR(9) | Academic year |
| `overall_percentage` | DECIMAL(5,2) | Performance percentage |
| `status` | VARCHAR(20) DEFAULT 'promoted' | `promoted`, `retained` |
| `promoted_by` | BIGINT UNSIGNED | User who promoted |

### `institutionkit_teacher_comments`

| Column | Type | Description |
|--------|------|-------------|
| `comment_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `student_id` | BIGINT(20) UNSIGNED | Student post ID |
| `teacher_id` | BIGINT(20) UNSIGNED | Staff ID (references `institutionkit_staff`) |
| `class_id` | BIGINT(20) UNSIGNED | Class post ID |
| `comment_type` | VARCHAR(50) DEFAULT 'general' | Comment category |
| `comment` | TEXT | Teacher's comment |
| `parent_response` | TEXT | Parent's response |
| `response_date` | DATETIME | Response timestamp |
| `is_read` | TINYINT(1) DEFAULT 0 | Read status |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

### `institutionkit_parent_child`

Links parent WordPress users to student posts.

| Column | Type | Description |
|--------|------|-------------|
| `relation_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `parent_id` | BIGINT(20) UNSIGNED | Parent user ID |
| `student_id` | BIGINT(20) UNSIGNED | Student post ID |
| `relationship_type` | VARCHAR(50) DEFAULT 'father' | `father`, `mother`, `guardian` |
| `is_primary` | TINYINT(1) DEFAULT 0 | Primary contact flag |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

**Unique Key:** `UNIQUE KEY parent_student (parent_id, student_id)`

### `institutionkit_ratings`

Student performance star ratings.

| Column | Type | Description |
|--------|------|-------------|
| `rating_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `student_id` | BIGINT(20) UNSIGNED | Student post ID |
| `term_id` | VARCHAR(50) | Term identifier |
| `overall_percentage` | DECIMAL(5,2) DEFAULT 0 | Overall percentage |
| `star_rating` | TINYINT(1) DEFAULT 0 | 1-5 star rating |
| `performance_label` | VARCHAR(50) | Performance category label |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

### `institutionkit_performance`

Student performance summaries per exam.

| Column | Type | Description |
|--------|------|-------------|
| `performance_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `student_id` | BIGINT(20) UNSIGNED | Student post ID |
| `exam_id` | BIGINT(20) UNSIGNED | Exam post ID |
| `total_marks` | DECIMAL(10,2) DEFAULT 0 | Total possible |
| `obtained_marks` | DECIMAL(10,2) DEFAULT 0 | Total scored |
| `percentage` | DECIMAL(5,2) DEFAULT 0 | Overall percentage |
| `grade` | VARCHAR(5) | Overall grade |
| `remarks` | TEXT | Teacher remarks |
| `teacher_remarks` | TEXT | Additional teacher comments |
| `campus_id` | INT DEFAULT 1 | Campus partition key |

### `institutionkit_campus_collections`

Revenue tracking per campus.

| Column | Type | Description |
|--------|------|-------------|
| `collection_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `campus_id` | BIGINT(20) UNSIGNED | Campus partition key |
| `collection_date` | DATE | Collection date |
| `source` | VARCHAR(30) | Revenue source |
| `amount` | DECIMAL(12,2) | Amount collected |
| `payment_method` | VARCHAR(20) | Payment method |
| `reference_number` | VARCHAR(255) | Reference/transaction ID |
| `student_id` | BIGINT(20) UNSIGNED | Optional student link |
| `invoice_id` | BIGINT(20) UNSIGNED | Optional invoice link |
| `payer_name` | VARCHAR(255) | Payer name |
| `recorded_by` | BIGINT(20) UNSIGNED | User who recorded |

### `institutionkit_email_log`

| Column | Type | Description |
|--------|------|-------------|
| `log_id` | BIGINT(20) UNSIGNED AUTO_INCREMENT | Primary key |
| `email_type` | VARCHAR(50) | Type of email |
| `recipient_email` | VARCHAR(255) | Recipient |
| `student_id` | BIGINT(20) UNSIGNED | Related student |
| `invoice_id` | BIGINT(20) UNSIGNED | Related invoice |
| `subject` | VARCHAR(255) | Email subject |
| `status` | VARCHAR(20) DEFAULT 'sent' | Delivery status |
| `sent_at` | DATETIME DEFAULT CURRENT_TIMESTAMP | Send timestamp |

---

## Table Listing (Complete)

All 43 custom tables:

| # | Table Name | Records |
|---|-----------|---------|
| 1 | `institutionkit_fee_types` | Fee categories |
| 2 | `institutionkit_fee_structures` | Fee structure groups |
| 3 | `institutionkit_fee_structure_items` | Structure line items |
| 4 | `institutionkit_student_fees` | Student fee assignments |
| 5 | `institutionkit_invoices` | Invoice records |
| 6 | `institutionkit_invoice_items` | Invoice line items |
| 7 | `institutionkit_transactions` | Payment records |
| 8 | `institutionkit_attendance` | Student attendance |
| 9 | `institutionkit_teacher_attendance` | Legacy teacher attendance |
| 10 | `institutionkit_staff_attendance` | Staff attendance |
| 11 | `ik_periods` | Grading periods |
| 12 | `ik_grades_v2` | Grade records (current) |
| 13 | `institutionkit_gradebook` | Grade records (legacy) |
| 14 | `ik_grade_scales` | Grade-to-letter mapping |
| 15 | `ik_exam_types` | Exam categories |
| 16 | `ik_exam_schedules` | Exam date sheets |
| 17 | `ik_exam_results` | Student results |
| 18 | `ik_report_cards` | Generated report cards |
| 19 | `institutionkit_staff` | Staff/employee records |
| 20 | `institutionkit_staff_salary_components` | Salary structure |
| 21 | `institutionkit_staff_loans` | Staff loans |
| 22 | `institutionkit_loan_installments` | Loan repayments |
| 23 | `institutionkit_payroll` | Payroll records |
| 24 | `institutionkit_expense_heads` | Expense categories |
| 25 | `institutionkit_expenses` | Expense ledger |
| 26 | `institutionkit_expense_approvals` | Approval workflow |
| 27 | `institutionkit_expense_budgets` | Budget planning |
| 28 | `institutionkit_campus_collections` | Revenue tracking |
| 29 | `institutionkit_campuses` | Campus definitions |
| 30 | `institutionkit_campus_users` | User-campus assignments |
| 31 | `institutionkit_campus_transfers` | Transfer audit trail |
| 32 | `institutionkit_announcements` | Announcements |
| 33 | `institutionkit_events` | Events calendar |
| 34 | `institutionkit_notifications` | Notification log |
| 35 | `institutionkit_meeting_slots` | Meeting availability |
| 36 | `institutionkit_meeting_bookings` | Meeting bookings |
| 37 | `institutionkit_meeting_topics` | Discussion topics |
| 38 | `institutionkit_homework` | Homework assignments |
| 39 | `institutionkit_homework_submissions` | Homework submissions |
| 40 | `institutionkit_certificates` | Certificates |
| 41 | `institutionkit_parent_child` | Parent-student links |
| 42 | `institutionkit_teacher_comments` | Teacher comments |
| 43 | `institutionkit_ratings` | Performance ratings |

---

## Indexes & Performance

### Key Indexes

| Table | Index | Type | Purpose |
|-------|-------|------|---------|
| Most tables | `campus_id` | KEY | Multi-campus data partitioning |
| `institutionkit_invoices` | `student_id`, `class_id` | KEY | Fast invoice lookup |
| `institutionkit_staff_attendance` | `staff_id`, `attendance_date` | UNIQUE | One record per staff per day |
| `ik_grades_v2` | `student_id`, `subject_id`, `period_id` | UNIQUE | One grade per student per subject per period |
| `ik_exam_results` | `schedule_id`, `student_id` | UNIQUE | One result per student per exam |
| `institutionkit_payroll` | `staff_id`, `campus_id`, `payroll_month` | UNIQUE | One payroll record per staff per month |
| `institutionkit_expense_budgets` | `campus_id`, `head_id`, `budget_month` | UNIQUE | One budget per head per month |
| `institutionkit_campus_users` | `user_id`, `campus_id` | UNIQUE | One assignment per user per campus |
| `institutionkit_parent_child` | `parent_id`, `student_id` | UNIQUE | One link per parent-student pair |

### Query Optimization Notes

- All campus-dependent queries use indexed `campus_id` columns
- Attendance queries are optimized with composite indexes on `(student_id, attendance_date)`
- Payroll queries use `(staff_id, payroll_month)` for fast monthly lookups
- Grade queries use the unique constraint `(student_id, subject_id, period_id)` for upsert operations
```
