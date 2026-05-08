# Quick Setup Guide

Follow this guide to configure InstitutionKit for your school in under 30 minutes.

---

## Overview

After installation, InstitutionKit creates a default "Main Campus" and inserts default data (grade scales, meeting topics, expense heads). This guide walks you through the essential first-time configuration.

---

## Step 1: Configure Your Campus

Navigate to **InstitutionKit → Campuses**.

1. Click **Edit** on "Main Campus"
2. Update these fields:

| Field | Example |
|-------|---------|
| Campus Name | "Springfield Elementary" |
| Campus Code | "SE" |
| Address | Full campus address |
| Phone | +1 555-0123 |
| Email | admin@springfield.edu |
| Principal Name | "Dr. Robert Langdon" |

3. Click **Save**

??? tip "Multiple Campuses"
    If you operate multiple campuses, click **Add New Campus** and repeat. Campus codes must be unique and are used throughout the system for data partitioning.

---

## Step 2: Set Up Classes

Navigate to **InstitutionKit → Setup → Classes**.

1. Click **Add Class**
2. Enter the class name (e.g., "Grade 1", "Grade 2", "Year 10")
3. Repeat for all grade levels in your institution

!!! example "Class Naming Convention"
    Use consistent naming: "Grade 1", "Grade 2" rather than "First Grade", "2nd Grade". This ensures clean sorting in dropdowns and reports.

---

## Step 3: Define Subjects

Navigate to **InstitutionKit → Setup → Subjects**.

1. Click **Add New Subject**
2. Enter subject names: Mathematics, English, Science, History, etc.
3. Assign subjects to the relevant classes:

=== "Bulk Assignment"
    1. Go to **Edit Class**
    2. Check the subjects taught in that class
    3. Save

=== "Per-Subject Assignment"
    1. Edit each subject
    2. Select the classes where this subject is taught
    3. Save

---

## Step 4: Create Sections

Navigate to **InstitutionKit → Setup → Sections**.

1. Click **Add New Section**
2. Enter section names: "A", "B", "C" (or "Morning", "Afternoon", etc.)
3. Sections allow you to split a class into multiple groups

!!! info "Sections Are Optional"
    If your institution doesn't use sections, you can skip this step. The system works without them.

---

## Step 5: Configure Grading Periods

Navigate to **InstitutionKit → Grading Periods**.

Grading periods define when grades are entered and how they're weighted.

1. Click **Add Period**
2. Configure each period:

=== "Monthly Period"
    | Field | Value |
    |-------|-------|
    | Title | "October 2025 Monthly Assessment" |
    | Period Type | Monthly |
    | Start Date | 2025-10-01 |
    | End Date | 2025-10-31 |
    | Academic Year | 2025-2026 |
    | Weight | 1.00 |

=== "Term Exam"
    | Field | Value |
    |-------|-------|
    | Title | "First Term Exam 2025" |
    | Period Type | Exam |
    | Start Date | 2025-09-15 |
    | End Date | 2025-09-25 |
    | Academic Year | 2025-2026 |
    | Weight | 3.00 |

!!! tip "Weight Explained"
    Weight determines how much this period contributes to final grades. An exam with weight 3.00 counts three times as much as a monthly assessment with weight 1.00.

---

## Step 6: Configure Grade Scales

Navigate to **InstitutionKit → Settings → Grade Scales** (or use the default scales).

InstitutionKit comes with this default scale:

| Percentage | Grade | GPA |
|------------|-------|-----|
| 90-100% | A+ | 4.00 |
| 85-89.99% | A | 4.00 |
| 80-84.99% | A- | 3.70 |
| 77-79.99% | B+ | 3.30 |
| 73-76.99% | B | 3.00 |
| 70-72.99% | B- | 2.70 |
| 67-69.99% | C+ | 2.30 |
| 63-66.99% | C | 2.00 |
| 60-62.99% | C- | 1.70 |
| 50-59.99% | D | 1.00 |
| 0-49.99% | F | 0.00 |

You can customize this scale at **Settings → Gradebook Settings** or add campus-specific scales.

---

## Step 7: Set Up Fee Types & Structures

Navigate to **Fee Management**.

### 7a. Create Fee Types

1. Click **Fee Types**
2. Add common fee categories:

| Fee Type | Example |
|----------|---------|
| Tuition Fee | Monthly tuition |
| Admission Fee | One-time admission |
| Exam Fee | Per-term examination |
| Transport Fee | Monthly transport |
| Library Fee | Annual library access |

### 7b. Create Fee Structures

1. Click **Fee Structures**
2. Create a structure (e.g., "Primary Monthly Fees")
3. Add fee types with amounts:

| Fee Type | Amount |
|----------|--------|
| Tuition Fee | 3,500 |
| Transport Fee | 1,000 |
| Library Fee | 200 |

3. Repeat for each class level

---

## Step 8: Add Staff Members

Navigate to **Payroll & Expenses → Staff**.

1. Click **Add Staff Member**
2. Fill in the form:

    | Field | Required | Notes |
    |-------|----------|-------|
    | Full Name | Yes | Legal name |
    | Email | Yes | Used for login credentials |
    | Phone | No | Contact number |
    | Role | Yes | Select from 8 predefined roles |
    | Primary Campus | Yes | The campus where they primarily work |
    | Contract Type | Yes | Monthly Fixed, Hourly, or Per Lecture |
    | Join Date | Yes | Employment start date |

3. Click **Add Staff Member**

!!! success "Auto-Account Creation"
    When you add a staff member with a role of **Permanent Teacher**, **Visiting Teacher**, or **Campus Head**, InstitutionKit automatically:

    - Creates a WordPress user account
    - Sends welcome email with login credentials
    - Assigns the appropriate WordPress role (teacher or campus_admin)

---

## Step 9: Configure School Settings

Navigate to **InstitutionKit → Settings**.

| Setting | Description |
|---------|-------------|
| Currency Symbol | Default: `$`. Change to `₨`, `£`, `€`, etc. |
| School Open Time | Default: `09:00 AM`. Used for staff attendance defaults |
| School Close Time | Default: `05:00 PM`. Used for staff attendance defaults |
| Academic Year | Current academic year (e.g., `2025-2026`) |
| Default Campus | The campus shown by default for super admins |

---

## Post-Setup Checklist

- [ ] Campus details configured
- [ ] All classes created
- [ ] Subjects defined and assigned to classes
- [ ] Sections created (if applicable)
- [ ] Grading periods configured for the academic year
- [ ] Grade scales reviewed
- [ ] Fee types and structures created
- [ ] At least one staff member added
- [ ] School operating hours set
- [ ] License activated

---

## Next Steps

Ready to start using InstitutionKit? Here's what to do next:

1. **Add students** via **Student Management → New Admission**
2. **Assign fees** to students via **Fee Management → Assign Fees**
3. **Mark attendance** via **Student Management → Student Attendance**
4. **Set up exams** via **Exams → Exam Types**

[License Activation →](license-activation.md){ .md-button }
