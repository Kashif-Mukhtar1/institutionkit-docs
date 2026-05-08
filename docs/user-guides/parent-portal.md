```markdown
# Parent Portal

The Parent Portal is a standalone web application that provides parents with a comprehensive view of their children's academic performance, attendance, homework, fee invoices, and teacher communications.

---

## Accessing the Parent Portal

The Parent Portal is accessed at:

```
https://your-school.edu/parent-portal.php
```

It is a **standalone PHP application** that loads WordPress core but renders its own complete HTML document — bypassing the WordPress theme entirely for optimal performance and control.

---

## Authentication

### Login Requirement

Parents must log in with their WordPress account. If not logged in, a styled login prompt appears:

```
┌──────────────────────────────────┐
│           🔐                      │
│    Parent Portal Login            │
│                                    │
│  Please login to access your      │
│  child's information.             │
│                                    │
│        [ Login Here ]             │
└──────────────────────────────────┘
```

### Role Restriction

Only users with the `parent` WordPress role can access the portal:

```php
if (!in_array('parent', $current_user->roles)) {
    wp_die('Only parents can access the parent portal.');
}
```

---

## Child Selection

Parents linked to multiple students can switch between children:

```
👧 Select Child: [Ahmed Khan - Grade 5 (Roll #42)  ▼]
                  [Fatima Khan - Grade 3 (Roll #15) ]
                  [Usman Khan - Grade 8 (Roll #07)  ]
```

The selected student's data loads dynamically across all sections.

### Parent-Child Linking

Links are stored in `institutionkit_parent_child`:

```php
$student_ids = $wpdb->get_col($wpdb->prepare(
    "SELECT student_id FROM {$parent_child_table} WHERE parent_id = %d",
    $current_user->ID
));
```

If no children are linked:

```
┌──────────────────────────────────┐
│              👶                   │
│       No Children Found           │
│                                    │
│  You are not linked to any        │
│  student yet. Please contact      │
│  the school administrator.        │
└──────────────────────────────────┘
```

---

## Portal Sections

### 1. Student Profile

Displays basic student information:

| Field | Source |
|-------|--------|
| Photo | Featured image or initial avatar |
| Name | Post title |
| Class | `_ik_student_class_id` → class name |
| Section | `_ik_section` |
| Roll Number | `_ik_roll_number` |
| Guardian Name | `_ik_guardian_name` or `_ik_father_name` |
| Contact | `_ik_father_contact` |

Plus two stat cards:

| Stat | Value |
|------|-------|
| Roll Number | Display value |
| Attendance | Overall percentage with progress bar |

---

### 2. Academic Performance

Dynamically displays performance data based on the current academic calendar:

#### Weekly View (Default)

Shows the current week's grading period grades:

```
📊 Weekly Assessment - Jun 8 to Jun 14

| Subject      | Marks   | Grade |
|-------------|---------|-------|
| Mathematics | 85/100  | A     |
| English     | 78/100  | B     |
| Science     | 92/100  | A+    |
```

#### Monthly View (4th week or month-end)

Shows monthly averages plus attendance:

```
📊 Monthly Progress (including Attendance)

| Subject      | Avg     | Grade |
|-------------|---------|-------|
| Mathematics | 82.5/100| —     |
| English     | 76.0/100| —     |
| 📊 Attendance| 91%     |       |
```

#### Yearly View (Year-end)

Shows final exam results:

```
📊 Yearly Final Exam Results

| Subject      | Marks   | Grade |
|-------------|---------|-------|
| Mathematics | 88/100  | A     |
| English     | 82/100  | B     |
| Science     | 95/100  | A+    |
```

#### Fallback

If no current period matches, shows the 5 most recent grades:

```
📊 Recent Assessments

| Subject (Period)     | Marks   | Grade |
|---------------------|---------|-------|
| Mathematics (Monthly)| 85/100  | A     |
| English (Weekly)    | 78/100  | B     |
```

---

### 3. Homework Assignments

Shows active homework for the student's class:

```
📚 Homework Assignments

┌──────────────────────────────────────────┐
│ 📖 Mathematics                 🔄 Every 1 week │
│ Worksheet Chapter 5                      │
│ 📅 Due: Friday, June 20, 2026            │
│                                           │
│ Complete all exercises from pages 42-45. │
│ 📎 Download Attachment                    │
└──────────────────────────────────────────┘
```

Empty state if no homework:

```
📓 No homework assigned at the moment.
```

---

### 4. Teacher Comments

Two-way communication with teachers:

```
💬 Teacher Comments

┌──────────────────────────────────────────┐
│ 👩‍🏫 Sarah Ahmed                  Jun 15, 2026 │
│                                           │
│ Your child has been doing well in         │
│ mathematics but needs to improve          │
│ homework completion.                      │
│                                           │
│ ┌─ Your Response ──────────────────────┐ │
│ │ Thank you for the feedback. We will  │ │
│ │ work on homework completion.         │ │
│ └──────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

Parents can respond to comments if no response has been sent yet:

```php
// AJAX response submission
$.ajax({
    url: ajaxurl,
    type: 'POST',
    data: {
        action: 'ik_parent_respond_comment',
        comment_id: commentId,
        response: responseText,
        nonce: commentNonce
    }
});
```

---

### 5. Fee Invoices

Shows the student's invoice history:

```
💰 Fee Invoices

| Invoice # | Title          | Total    | Paid   | Due Date  | Status    |
|-----------|---------------|----------|--------|-----------|-----------|
| #127      | June 2026 Fees | 4,700.00 | 0.00   | Jun 5, 26 | ❌ Unpaid |
| #115      | May 2026 Fees  | 4,700.00 | 4,700  | May 5, 26 | ✅ Paid   |
```

Status colors:
- **Paid**: Green
- **Unpaid**: Red
- **Partial**: Yellow

---

### 6. Upcoming Meetings

Parent-Teacher meeting management:

```
📅 Upcoming Parent-Teacher Meetings

┌──────────────────────────────────────────┐
│ [ + Book a Meeting ]                     │
└──────────────────────────────────────────┘

┌──────────────────────────────────────────┐
│ 📅 June 20, 2026                         │
│ ⏰ 2:00 PM - 2:30 PM                     │
│ 📍 Room 101                               │
│ 👩‍🏫 Sarah Ahmed                            │
│ 📋 Topics: Academic Performance           │
└──────────────────────────────────────────┘
```

#### Booking a Meeting

Click **Book a Meeting** to:

1. View available time slots for all teachers
2. Select a slot (visual card with date, time, location)
3. Choose discussion topics from predefined list
4. Confirm booking

```javascript
$('#ik-confirm-booking').on('click', function() {
    $.ajax({
        url: ajaxurl,
        type: 'POST',
        data: {
            action: 'ik_parent_book_meeting',
            slot_id: selectedSlotId,
            student_id: studentId,
            topics: selectedTopics,
            nonce: meetingNonce
        }
    });
});
```

#### Available Topics

| # | Topic |
|---|-------|
| 1 | Academic Performance |
| 2 | Behavior and Conduct |
| 3 | Attendance Issues |
| 4 | Homework Concerns |
| 5 | Special Needs Support |
| 6 | College/Career Planning |
| 7 | Extra-curricular Activities |
| 8 | Health Concerns |
| 9 | Social Development |
| 10 | General Check-in |

---

## AJAX Endpoints

| Action | Purpose |
|--------|---------|
| `ik_parent_get_upcoming_meetings` | Load meeting cards |
| `ik_parent_get_available_slots` | Load slot selection UI |
| `ik_parent_book_meeting` | Submit booking |
| `ik_parent_respond_comment` | Save parent response |

---

## Performance Optimization

The Parent Portal:

- **Bypasses the WordPress theme** — No theme assets loaded
- **Loads only required WordPress core** — `wp-load.php` only
- **No caching** — Headers prevent browser caching for fresh data
- **Direct database queries** — No WordPress post loops for performance

```php
header('Cache-Control: no-cache, no-store, must-revalidate');
header('Pragma: no-cache');
header('Expires: 0');
```

---

## No Children State

If a parent account has no linked children:

```
┌──────────────────────────────────┐
│              👶                   │
│       No Children Found           │
│                                    │
│  You are not linked to any        │
│  student yet. Please contact      │
│  the school administrator.        │
└──────────────────────────────────┘
```

---

## Responsive Design

The Parent Portal is fully responsive:

- Desktop: Two-column layout
- Tablet: Single column, adjusted spacing
- Mobile: Stacked cards, full-width tables, horizontal scroll for data tables

---

## Programmatic Usage

### Link Parent to Student

```php
global $wpdb;
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_parent_child",
    [
        'parent_id'         => $parent_user_id,
        'student_id'        => $student_id,
        'relationship_type' => 'father',
        'is_primary'        => 1,
        'campus_id'         => $campus_id,
    ]
);
```

### Get Parent's Children

```php
global $wpdb;
$student_ids = $wpdb->get_col($wpdb->prepare(
    "SELECT student_id FROM {$wpdb->prefix}institutionkit_parent_child 
     WHERE parent_id = %d",
    $parent_user_id
));
```

### Get Student Performance Data

```php
global $wpdb;
$grades = $wpdb->get_results($wpdb->prepare(
    "SELECT g.*, p.post_title as period_title 
     FROM {$wpdb->prefix}ik_grades_v2 g
     JOIN {$wpdb->posts} p ON g.period_id = p.ID
     WHERE g.student_id = %d 
     ORDER BY g.created_at DESC 
     LIMIT 10",
    $student_id
));
```
