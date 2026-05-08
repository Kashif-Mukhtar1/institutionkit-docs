```markdown
# Parent-Teacher Meetings

The Meetings module enables teachers to create availability slots and parents to book appointments. It includes topic management, booking tracking, and post-meeting feedback.

---

## Accessing Meetings

| Interface | Path | Who Uses It |
|-----------|------|-------------|
| **Admin Panel** | InstitutionKit → Teacher Management → Meeting Slots | Teachers, Admins |
| **Parent Portal** | Parent Portal → Upcoming Meetings | Parents |

---

## Meeting Architecture

```
Teacher creates Slot
    │
    ├── Date, Time, Duration, Location
    ├── Max Bookings limit
    └── Optional Class restriction
        │
Parent books Slot
    │
    ├── Selects discussion Topics
    ├── Provides contact information
    └── Booking confirmed
        │
Meeting occurs
    │
    ├── Teacher records Notes
    ├── Teacher provides Feedback
    └── Attendance marked
```

---

## Managing Meeting Slots (Teacher View)

### Creating a Slot

Navigate to **Teacher Management → Meeting Slots** and click **Add Slot**.

| Field | Description |
|-------|-------------|
| **Date** | Date of availability |
| **Start Time** | Meeting start time |
| **End Time** | Meeting end time |
| **Duration** | Default: 30 minutes |
| **Max Bookings** | How many parents can book this slot (default: 1) |
| **Location** | Where the meeting will be held |
| **Meeting Type** | In-Person or Online |
| **Class** | Optional — restrict to a specific class |
| **Notes** | Internal notes |

### Slot Status

| Status | Meaning |
|--------|---------|
| **Available** | Open for bookings |
| **Full** | All booking slots taken |
| **Cancelled** | Slot cancelled by teacher |

When `current_bookings` reaches `max_bookings`, the slot automatically changes to "Full".

### Slot List View

| Column | Description |
|--------|-------------|
| Date | Slot date |
| Time | Start - End time |
| Duration | Slot duration |
| Bookings | Current / Max |
| Location | Meeting location |
| Status | Availability status |
| Actions | Edit / Cancel |

---

## Booking a Meeting (Parent View)

### Parent Portal

Parents access meetings through the Parent Portal:

1. Navigate to the **Upcoming Meetings** section
2. Click **Book a Meeting**
3. Select a child (if multiple children linked)
4. Choose from available slots
5. Select discussion topics
6. Confirm booking

### Booking Form

| Field | Description |
|--------|-------------|
| **Student** | Auto-selected from parent's linked children |
| **Time Slot** | Visual card selection showing date, time, location |
| **Topics** | Checkboxes for discussion topics |
| **Parent Name** | Auto-filled from user profile |
| **Parent Email** | Auto-filled from user profile |
| **Parent Phone** | Auto-filled if available |

### Booking Confirmation

After booking:

- Parent sees confirmation message
- Slot's `current_bookings` increments
- If full, slot status changes to "Full"
- Booking appears in parent's "Upcoming Meetings" list

---

## Meeting Topics

### Default Topics

InstitutionKit includes 10 predefined topics:

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

### Managing Topics

Topics can be managed at **InstitutionKit → Teacher Management → Meeting Topics**:

- Add new topics
- Edit existing topics
- Reorder topics (sort order)
- Deactivate unused topics

---

## After the Meeting

### Teacher Actions

After the meeting, teachers can:

1. Mark attendance (Attended / No Show)
2. Add meeting notes
3. Provide feedback visible to parents
4. Update booking status

### Booking Status Flow

```
Confirmed → Meeting Held → Completed
                │
                └── No Show → Cancelled
```

---

## Meeting Database

### Slots Table

```sql
CREATE TABLE wp_institutionkit_meeting_slots (
    slot_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    teacher_id BIGINT(20) UNSIGNED NOT NULL,
    class_id BIGINT(20) UNSIGNED,
    slot_date DATE NOT NULL,
    start_time TIME NOT NULL,
    end_time TIME NOT NULL,
    duration INT(11) DEFAULT 30,
    max_bookings INT(11) DEFAULT 1,
    current_bookings INT(11) DEFAULT 0,
    location VARCHAR(255) DEFAULT 'School Office',
    meeting_type VARCHAR(50) DEFAULT 'in_person',
    status VARCHAR(20) DEFAULT 'available',
    notes TEXT,
    campus_id INT DEFAULT 1
);
```

### Bookings Table

```sql
CREATE TABLE wp_institutionkit_meeting_bookings (
    booking_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    slot_id BIGINT(20) UNSIGNED NOT NULL,
    student_id BIGINT(20) UNSIGNED NOT NULL,
    parent_id BIGINT(20) UNSIGNED NOT NULL,
    teacher_id BIGINT(20) UNSIGNED NOT NULL,
    parent_name VARCHAR(255) NOT NULL,
    parent_email VARCHAR(255) NOT NULL,
    parent_phone VARCHAR(50),
    topics TEXT,
    status VARCHAR(20) DEFAULT 'confirmed',
    meeting_notes TEXT,
    teacher_feedback TEXT,
    attended TINYINT(1) DEFAULT 0,
    campus_id INT DEFAULT 1
);
```

### Topics Table

```sql
CREATE TABLE wp_institutionkit_meeting_topics (
    topic_id BIGINT(20) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    topic_name VARCHAR(255) NOT NULL,
    description TEXT,
    is_active TINYINT(1) DEFAULT 1,
    sort_order INT(11) DEFAULT 0
);
```

---

## Capabilities

| Capability | Who Has It | Purpose |
|-----------|------------|---------|
| `ik_manage_meeting_slots` | Admin, Campus Admin, Teacher | Create and manage availability slots |
| `ik_view_meeting_bookings` | Admin, Campus Admin, Teacher | View bookings |
| `ik_manage_meeting_bookings` | Admin, Campus Admin | Manage all bookings |
| `ik_book_meeting_slots` | Parent | Book available slots |
| `ik_view_own_meetings` | Parent | View own bookings |

---

## AJAX Endpoints (Parent Portal)

| Action | Purpose |
|--------|---------|
| `ik_parent_get_upcoming_meetings` | Load upcoming meetings for a student |
| `ik_parent_get_available_slots` | Load available slots for booking |
| `ik_parent_book_meeting` | Submit a booking |

---

## Programmatic Usage

### Create a Meeting Slot

```php
global $wpdb;
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_meeting_slots",
    [
        'teacher_id'    => $staff_id,
        'slot_date'     => '2026-06-20',
        'start_time'    => '14:00',
        'end_time'      => '14:30',
        'duration'      => 30,
        'max_bookings'  => 1,
        'location'      => 'Room 101',
        'meeting_type'  => 'in_person',
        'campus_id'     => $campus_id,
    ]
);
```

### Get Available Slots for a Date

```php
global $wpdb;
$slots = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_meeting_slots 
     WHERE slot_date = %s 
       AND status = 'available' 
       AND current_bookings < max_bookings
       AND campus_id = %d
     ORDER BY start_time ASC",
    $date, $campus_id
));
```

### Book a Slot

```php
global $wpdb;

// Insert booking
$wpdb->insert(
    "{$wpdb->prefix}institutionkit_meeting_bookings",
    [
        'slot_id'      => $slot_id,
        'student_id'   => $student_id,
        'parent_id'    => $parent_user_id,
        'teacher_id'   => $teacher_id,
        'parent_name'  => $parent_name,
        'parent_email' => $parent_email,
        'topics'       => 'Academic Performance, Behavior',
    ]
);

// Update slot booking count
$wpdb->query($wpdb->prepare(
    "UPDATE {$wpdb->prefix}institutionkit_meeting_slots 
     SET current_bookings = current_bookings + 1 
     WHERE slot_id = %d",
    $slot_id
));

// Check if slot is now full
$slot = $wpdb->get_row($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}institutionkit_meeting_slots WHERE slot_id = %d",
    $slot_id
));

if ($slot->current_bookings >= $slot->max_bookings) {
    $wpdb->update(
        "{$wpdb->prefix}institutionkit_meeting_slots",
        ['status' => 'full'],
        ['slot_id' => $slot_id]
    );
}
```
