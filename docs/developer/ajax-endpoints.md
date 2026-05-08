```markdown
# AJAX Endpoints Reference

InstitutionKit registers AJAX endpoints across multiple modules for dynamic data loading, form submissions, and real-time updates. This reference documents every endpoint, its purpose, required parameters, and expected responses.

---

## Exams Module

**AJAX action:** `ik_exam_ajax_handler`

All exam AJAX operations flow through a single handler with a `method` parameter.

### Exam Types

| Method | Description | Parameters |
|--------|-------------|------------|
| `save_exam_type` | Create or update exam type | `exam_type_id` (optional), `exam_name`, `exam_category`, `max_marks`, `weight_percentage`, `is_active`, `campus_id` |
| `get_exam_type` | Get single exam type | `exam_type_id` |
| `delete_exam_type` | Delete or deactivate exam type | `exam_type_id` |
| `list_exam_types` | List exam types | `campus_id`, `category`, `active_only` |

### Exam Schedule

| Method | Description | Parameters |
|--------|-------------|------------|
| `save_schedule` | Create or update schedule | `schedule_id` (optional), `exam_type_id`, `class_id`, `subject_id`, `exam_date`, `start_time`, `end_time`, `max_marks`, etc. |
| `get_schedule` | Get single schedule | `schedule_id` |
| `delete_schedule` | Delete schedule | `schedule_id` |
| `list_schedules` | List schedules | `campus_id`, `exam_type_id`, `class_id` |
| `publish_schedule` | Publish schedule | `schedule_id` |

### Result Entry

| Method | Description | Parameters |
|--------|-------------|------------|
| `save_results` | Save student results | `schedule_id`, `results` (array of student data) |
| `get_results` | Get results for schedule | `schedule_id` |
| `verify_results` | Verify submitted results | `schedule_id` |
| `publish_results` | Publish results | `schedule_id` |
| `get_students_for_result` | Get students for entry | `class_id`, `subject_id` |

### Report Cards

| Method | Description | Parameters |
|--------|-------------|------------|
| `generate_report_card` | Generate single card | `student_id`, `exam_type_id`, `academic_year` |
| `bulk_generate_report_cards` | Generate cards for class | `class_id`, `exam_type_id`, `academic_year` |
| `get_report_card_list` | List report cards | `campus_id`, `exam_type_id`, `academic_year` |
| `publish_report_card` | Publish report card | `card_id` |
| `download_report_card` | Download card PDF | `card_id` |

### Analytics

| Method | Description | Parameters |
|--------|-------------|------------|
| `get_class_performance` | Class performance data | `exam_type_id`, `class_id`, `campus_id` |
| `get_student_performance` | Student performance data | `student_id`, `academic_year` |
| `get_subject_analysis` | Subject analysis data | `subject_id`, `exam_type_id`, `campus_id` |
| `get_comparative_analysis` | Cross-exam comparison | `exam_type_ids`, `campus_ids`, `academic_year` |

**Example request:**

```javascript
$.ajax({
    url: ikExam.ajax_url,
    type: 'POST',
    data: {
        action: 'ik_exam_ajax_handler',
        method: 'save_exam_type',
        nonce: ikExam.nonce,
        data: {
            exam_name: 'First Term Exam 2026',
            exam_category: 'term',
            max_marks: 100,
            weight_percentage: 70,
            is_active: 1,
            campus_id: 3
        }
    },
    success: function(response) {
        console.log(response);
    }
});
```

---

## Payroll & Expenses Module

Each endpoint is a separate WordPress AJAX action.

| Action | Purpose | Key Parameters | Response |
|--------|---------|---------------|----------|
| `ik_add_expense` | Add new expense | `campus_id`, `head_id`, `amount`, `description`, `expense_date`, `paid_by` | `{ id: expense_id }` |
| `ik_get_expenses` | Get expenses with filters | `campus_id`, `status`, `date_from`, `date_to`, `limit` | Array of expense objects |
| `ik_approve_expense` | Approve pending expense | `expense_id` | `{ status: 'fully_approved' }` |
| `ik_reject_expense` | Reject expense | `expense_id`, `reason` | `true` |
| `ik_add_staff` | Add staff member | `full_name`, `email`, `role`, `primary_campus_id`, `contract_type` | `{ id: staff_id }` |
| `ik_get_staff` | Get all staff | — | Array of staff objects |
| `ik_generate_payroll` | Generate monthly payroll | `campus_id`, `month` | `{ count: records_generated }` |
| `ik_get_payroll` | Get payroll records | `campus_id`, `payroll_month`, `staff_id`, `status` | Array of payroll objects |
| `ik_get_financial_summary` | Get financial summary | `campus_id`, `start_date`, `end_date` | `{ revenue, expenses, profit, metrics }` |
| `ik_get_comparison_data` | Get campus comparison | `start_date`, `end_date` | Array of campus comparison objects |
| `ik_get_dashboard_data` | Get dashboard data | `campus_id` | `{ summary, trend, breakdown, comparison, alerts, pending }` |
| `ik_add_expense_head` | Add custom expense head | `head_name`, `head_type`, `campus_id`, `icon_class`, `color_code` | `true` |
| `ik_record_attendance` | Record staff attendance | `staff_id`, `campus_id`, `attendance_date`, `status`, `check_in`, `check_out` | `true` |
| `ik_add_collection` | Record collection | `campus_id`, `collection_date`, `source`, `amount`, `payment_method` | `true` |
| `ik_add_salary_component` | Add salary component | `staff_id`, `type`, `label`, `amount`, `taxable` | `{ id: component_id }` |
| `ik_remove_salary_component` | Remove salary component | `id` | `true` |
| `ik_add_loan` | Add staff loan | `staff_id`, `principal`, `tenure`, `interest_rate`, `loan_date` | `{ loan_id, monthly, total }` |
| `ik_get_payslip` | Get payslip data | `staff_id`, `month` | Full payslip object |

**Nonce required:** `ik_payroll_nonce` for all endpoints.

**Example request:**

```javascript
$.ajax({
    url: ik_ajax.ajax_url,
    type: 'POST',
    data: {
        action: 'ik_add_expense',
        nonce: ik_ajax.nonce,
        campus_id: 3,
        head_id: 2,
        amount: 15000,
        description: 'Monthly electricity bill',
        expense_date: '2026-06-15',
        paid_by: 'campus_petty'
    },
    success: function(response) {
        if (response.success) {
            console.log('Expense #' + response.data.id + ' created');
        }
    }
});
```

---

## Frontend Gradebook

| Action | Purpose | Parameters | Response |
|--------|---------|------------|----------|
| `ik_frontend_get_subjects` | Get subjects for a class | `class_id` | Array of subject objects |
| `ik_frontend_save_grades` | Save grades (upsert) | `period_id`, `class_id`, `subject_id`, `period_type`, `grades` (array) | `true` |

**Nonce required:** `ik_frontend_gradebook` nonce from `ik_grade_nonce` field.

---

## Frontend Invoices

| Action | Purpose | Parameters | Response |
|--------|---------|------------|----------|
| `ik_add_invoice_payment` | Record payment via AJAX | `form_data` (serialized form) | `{ new_total_paid_formatted, new_outstanding_balance_formatted, is_fully_paid, new_payment_row_html }` |

**Nonce required:** `ik_invoices_nonce`.

---

## Parent Portal

| Action | Purpose | Parameters | Response |
|--------|---------|------------|----------|
| `ik_parent_get_upcoming_meetings` | Load meetings | `student_id` | `{ html: meeting_cards_html }` |
| `ik_parent_get_available_slots` | Load booking slots | `student_id` | `{ html: slot_selection_html }` |
| `ik_parent_book_meeting` | Book a slot | `slot_id`, `student_id`, `topics` | `{ message: 'Meeting booked!' }` |
| `ik_parent_respond_comment` | Save parent response | `comment_id`, `response` | `{ message: 'Response saved.' }` |

**Nonces required:** `ik_parent_meeting_nonce`, `ik_parent_comment_nonce`.

---

## Dashboard

| Action | Purpose | Parameters | Response |
|--------|---------|------------|----------|
| `ik_export_dashboard_data` | Export dashboard as JSON | — | JSON file download |
| `ik_send_fee_reminders` | Send overdue reminders | — | `{ message: 'X reminders sent' }` |
| `ik_delete_announcement` | Delete announcement | `announcement_id` | `true` |
| `ik_delete_event` | Delete event | `event_id` | `true` |

---

## Nonce Reference

| Module | Nonce Action | Field Name |
|--------|-------------|------------|
| Exams | `ik_exam_nonce` | `nonce` |
| Payroll & Expenses | `ik_payroll_nonce` | `nonce` |
| Frontend Gradebook | `ik_frontend_gradebook` | `ik_grade_nonce` |
| Frontend Invoices | `ik_invoices_nonce` | `nonce` |
| Parent Meetings | `ik_parent_meeting_nonce` | `nonce` |
| Parent Comments | `ik_parent_comment_nonce` | `nonce` |
| Dashboard Export | `ik_export_nonce` | `nonce` |
| Dashboard Reminders | `ik_reminders_nonce` | `nonce` |
| Delete Announcement | `ik_delete_announcement_nonce` | `nonce` |
| Delete Event | `ik_delete_event_nonce` | `nonce` |

---

## Response Format

All endpoints return JSON in standard WordPress format:

```json
{
    "success": true,
    "data": {
        // Response-specific data
    }
}
```

Error responses:

```json
{
    "success": false,
    "data": {
        "message": "Error description"
    }
}
```
