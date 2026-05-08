File name: `docs/user-guides/dashboard.md`

```markdown
# Dashboard

The InstitutionKit Dashboard is the command center for your entire institution. It provides real-time statistics, attendance tracking, financial summaries, and quick access to all modules — all filtered by campus context.

---

## Dashboard Layout

The dashboard is organized into distinct sections:

```
┌──────────────────────────────────────────────┐
│  HEADER: Welcome + Campus Badge + Switcher   │
├──────────────────────────────────────────────┤
│  CAMPUS INFO BAR (single campus view only)   │
├──────────────────────────────────────────────┤
│  CAMPUS SUMMARY CARDS (All Campuses view)    │
├─────────────┬─────────────┬──────────────────┤
│  STUDENTS   │  TEACHERS   │  CLASSES   │ SUBJECTS │
├─────────────┴─────────────┴──────────────────┤
│  TODAY'S ATTENDANCE (Students + Staff)       │
├──────────────────────────────────────────────┤
│  FEE MANAGEMENT SUMMARY                      │
├──────────────────────┬───────────────────────┤
│  LEFT COLUMN         │  RIGHT COLUMN         │
│  • Announcements     │  • Outstanding Fees   │
│  • Upcoming Events   │  • Recent Payments    │
│  • Performance Charts│  • Parent Stats       │
│                      │  • Notifications      │
├──────────────────────┴───────────────────────┤
│  QUICK ACTIONS                               │
└──────────────────────────────────────────────┘
```

---

## Header Section

### Welcome Message

Displays a personalized greeting with the current date:

> "Welcome back, John! Here's your school management overview for today."
> 📅 Friday, June 12, 2026

### Campus Badge

| View | Badge Display |
|------|--------------|
| Single Campus | 🏛️ Downtown Campus (DT) — with principal name, phone, email, address |
| All Campuses | 🏛️ All Campuses |

### Campus Switcher (Super Admins Only)

The dropdown next to the campus badge allows Super Admins to switch between campuses or view aggregated data. Campus Admins do not see this control.

---

## Campus Info Bar

When viewing a **single campus**, a horizontal info bar displays:

| Field | Icon | Example |
|-------|------|---------|
| Campus Name | 📍 | Downtown Campus (DT) |
| Principal | 👤 | Dr. Sarah Johnson |
| Phone | 📞 | +1 555-0100 |
| Email | ✉️ | downtown@school.edu |
| Address | 🏠 | 123 Main Street, Springfield |

---

## Campus Summary Cards

When viewing **All Campuses**, each campus gets a summary card:

| Metric | Description |
|--------|-------------|
| Students | Total enrolled |
| Teachers | Active teaching staff |
| Outstanding | Total unpaid fees |
| Attendance | Today's attendance rate |

Clicking a card switches to that campus's detailed dashboard.

---

## Stats Grid

Four key metrics displayed as cards:

| Card | Description | Source |
|------|-------------|--------|
| 👥 **Total Students** | Count of published `ik_student` posts | Filtered by campus |
| 👨‍🏫 **Total Teachers** | Count of active staff with teacher roles | `institutionkit_staff` |
| 🏫 **Total Classes** | Count of published `ik_class` posts | Filtered by campus |
| 📚 **Total Subjects** | Count of `ik_subject` terms | System-wide |

---

## Today's Attendance

### Student Attendance Card

| Stat | Values |
|------|--------|
| Present | Count + green styling |
| Absent | Count + red styling — **clickable** to see absent student list |
| Late | Count + yellow styling — **clickable** to see late student list |
| On Leave | Count + blue styling — **clickable** to see leave list |

Includes a **progress bar** and **percentage**:

```
[████████████░░░░] 82% Attendance Rate
```

### Teacher Attendance Card

Identical format but sourced from `institutionkit_staff_attendance` for staff with teacher roles.

### Weekly Trend Chart

A Chart.js line graph showing the last 7 days of student attendance percentages.

---

## Fee Management Summary

| Card | Formula | Description |
|------|---------|-------------|
| 💰 **Total Outstanding** | SUM(total_amount - amount_paid) WHERE status != 'paid' | Across all students |
| ✅ **Total Collected** | SUM(transactions) for current month | Month-to-date collections |
| ⚠️ **Overdue Invoices** | COUNT WHERE due_date < today AND status != 'paid' | Past-due invoices with amount |
| 📊 **Collection Rate** | (Collected ÷ Total Billed) × 100 | Month-to-date efficiency |

---

## Two-Column Content Area

### Left Column

#### Announcements Widget

Displays the 10 most recent active announcements filtered by audience:

- Title, content preview, post date, expiry date
- Delete button for admins
- Empty state: "No announcements at this time"

#### Upcoming Events Widget

Shows the next 10 upcoming events:

- Title with type badge (Exam, Holiday, Meeting, School Event, Fee Due Date)
- Description, location, date/time
- Delete button for admins

#### Performance Charts (Admin & Teacher only)

Two Chart.js bar charts:

| Chart | Data |
|-------|------|
| **Class Performance** | Top 5 classes by average percentage |
| **Subject Performance** | Top 5 subjects by average percentage |

Both sourced from `institutionkit_gradebook`.

### Right Column

#### Outstanding Fees by Class (Admin & Accountant only)

Table sorted by highest outstanding balance:

| Column | Description |
|--------|-------------|
| Class | Class name |
| Total Due | SUM(total_amount - amount_paid) |
| Students | COUNT of students with outstanding balances |

#### Parent Portal Stats (Admin only)

| Stat | Value |
|------|-------|
| Total Parents | Count of users with `parent` role |
| Linked to Students | Count with active parent-child links |
| Link Rate | Percentage displayed as progress bar |

#### Recent Notifications (Admin only)

Last 10 notifications from `institutionkit_notifications`:

| Column | Description |
|--------|-------------|
| Date | Timestamp |
| Type | Notification type |
| Channel | Email/SMS |
| Status | Sent/Failed |

#### Recent Payments (Admin & Accountant only)

Last 10 transactions:

| Column | Description |
|--------|-------------|
| Date | Payment date |
| Student | Student name |
| Amount | Payment amount (green) |
| Method | Cash, Bank, Mobile |

---

## Quick Actions

Nine action buttons providing one-click access to common tasks:

| Button | Destination |
|--------|------------|
| ➕ New Admission | Student Management → New Admission |
| 📊 Compare Campuses | Campus Comparison |
| 🆔 Add Student | Add New Student form |
| 👨‍🏫 Add Teacher/Staff | Staff Management |
| 🏫 Add Class | Add New Class form |
| 📋 Mark Attendance | Student Attendance page |
| 🧾 Manage Invoices | Invoices page |
| 📢 Post Announcement | Opens announcement modal |
| 📅 Add Event | Opens event modal |

---

## Modals

### Post Announcement Modal

Opened via Quick Actions button. Fields:

| Field | Type | Required |
|-------|------|----------|
| Title | Text | Yes |
| Content | Textarea | Yes |
| Target Audience | Select: Everyone, Teachers Only, Parents Only, Students Only | Yes |
| Expires On | Date | No |

### Add Event Modal

| Field | Type | Required |
|-------|------|----------|
| Event Title | Text | Yes |
| Description | Textarea | No |
| Event Type | Select: Exam, Holiday, Meeting, School Event, Fee Due Date | Yes |
| Start Date & Time | Datetime-local | Yes |
| End Date & Time | Datetime-local | No |
| Location | Text | No |

---

## Export & Actions

### Export Report

The **Export Report** button generates a JSON file containing:

- All students
- All staff
- All invoices
- Last 30 days of attendance

Downloaded as `institutionkit-export-YYYY-MM-DD.json`.

### Send Fee Reminders

The **Send Fee Reminders** button triggers an AJAX process that:

1. Finds all overdue invoices (`due_date < today AND status != 'paid'`)
2. Looks up parent email addresses from student meta and parent-child links
3. Sends reminder emails with invoice details and outstanding amounts
4. Logs each notification in `institutionkit_notifications`
5. Returns a summary: "X reminder(s) sent! (Y students have no email)"

---

## Role-Based Visibility

Not all dashboard sections appear for all users:

| Section | Admin | Campus Admin | Accountant | Teacher | Parent |
|---------|:---:|:---:|:---:|:---:|:---:|
| Stats Grid | ✅ | ✅ | ✅ | ✅ | ✅ |
| Student Attendance | ✅ | ✅ | ✅ | ✅ | — |
| Teacher Attendance | ✅ | ✅ | — | — | — |
| Fee Summary | ✅ | ✅ | ✅ | — | — |
| Announcements | ✅ | ✅ | ✅ | ✅ | — |
| Events | ✅ | ✅ | ✅ | ✅ | — |
| Performance Charts | ✅ | ✅ | — | ✅ | — |
| Outstanding Fees | ✅ | ✅ | ✅ | — | — |
| Parent Stats | ✅ | — | — | — | — |
| Notifications | ✅ | — | — | — | — |
| Recent Payments | ✅ | ✅ | ✅ | — | — |
| Quick Actions | ✅ | ✅ | — | — | — |
| Campus Switcher | ✅ | — | — | — | — |

---

## Performance Notes

- Dashboard data is cached in WordPress transients with a 5-minute TTL
- The "All Campuses" view executes aggregation queries — may be slower with 10+ campuses
- Campus-specific views use indexed `campus_id` columns for consistent performance
- The attendance trend chart loads data via localized JSON — no additional AJAX calls

---

## Customizing the Dashboard

### Adding a Custom Widget

```php
add_action('ik_dashboard_widgets', function() {
    ?>
    <div class="ik-stat-card">
        <h3>📊 My Custom Widget</h3>
        <div class="ik-stat-number"><?php echo my_custom_data(); ?></div>
    </div>
    <?php
});
```

### Removing a Default Widget

```php
// Disable parent portal stats for non-admins
add_filter('ik_show_parent_stats', '__return_false');
```
