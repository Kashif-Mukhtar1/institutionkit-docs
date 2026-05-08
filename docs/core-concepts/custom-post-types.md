File name: `docs/core-concepts/custom-post-types.md`

```markdown
# Custom Post Types & Taxonomies

InstitutionKit registers **5 custom post types** and **2 custom taxonomies** to manage core educational entities within WordPress. This document covers each type, its metadata schema, and how they interact with the custom database tables.

---

## Architecture Overview

InstitutionKit uses a **hybrid storage model**:

| Data Type | Storage | Why |
|-----------|---------|-----|
| **Students, Classes, Exams, Certificates, Grading Periods** | WordPress CPTs + Post Meta | Native admin UI, revision support, media attachments |
| **Financial records, Attendance, Grades, Payroll** | Custom database tables | Performance, complex queries, aggregation |
| **Subjects, Sections** | WordPress Taxonomies | Native term management, assignment to posts |

---

## Custom Post Types

### 1. `ik_student` — Students

| Property | Value |
|----------|-------|
| **Post Type Key** | `ik_student` |
| **Supports** | `title`, `thumbnail` |
| **Public** | Yes |
| **Show in REST** | Yes |
| **Menu Position** | Under InstitutionKit dashboard |
| **Title Placeholder** | "Enter Student Name" |

#### Post Meta Keys

| Meta Key | Type | Description |
|----------|------|-------------|
| `_ik_student_class_id` | INT | Class post ID (links to `ik_class`) |
| `_ik_campus_id` | INT | Campus assignment |
| `_ik_roll_number` | VARCHAR | Student roll number |
| `_ik_date_of_birth` | DATE | Date of birth |
| `_ik_gender` | VARCHAR | Gender |
| `_ik_email` | VARCHAR | Contact email |
| `_ik_guardian_name` | VARCHAR | Guardian/parent name |
| `_ik_guardian_title` | VARCHAR | e.g., "Father", "Mother" |
| `_ik_father_name` | VARCHAR | Father's full name |
| `_ik_father_contact` | VARCHAR | Father's phone |
| `_ik_mother_contact` | VARCHAR | Mother's phone |
| `_ik_father_occupation` | VARCHAR | Father's occupation |
| `_ik_mother_occupation` | VARCHAR | Mother's occupation |
| `_ik_father_qualification` | VARCHAR | Father's qualification |
| `_ik_mother_qualification` | VARCHAR | Mother's qualification |
| `_ik_address` | TEXT | Physical address |
| `_ik_emergency_contact` | VARCHAR | Emergency phone number |
| `_ik_cnic_number` | VARCHAR | Guardian's ID card number |
| `_ik_section` | VARCHAR | Section assignment |

---

### 2. `ik_class` — Classes

| Property | Value |
|----------|-------|
| **Post Type Key** | `ik_class` |
| **Supports** | `title` only |
| **Public** | Yes |
| **Hierarchical** | Yes |
| **Show in REST** | Yes |
| **Title Placeholder** | "Enter Class Name (e.g., Grade 5)" |
| **Taxonomies** | `ik_subject`, `ik_section` |

#### Post Meta Keys

| Meta Key | Type | Description |
|----------|------|-------------|
| `_ik_campus_id` | INT | Campus assignment (if campus-specific) |

---

### 3. `ik_exam` — Exams (Legacy)

!!! warning "Legacy Post Type"
    `ik_exam` was the original exam storage before the Exam Pro module was introduced. It is maintained for backward compatibility. New exam data is stored in custom tables: `ik_exam_types`, `ik_exam_schedules`, `ik_exam_results`, `ik_report_cards`.

| Property | Value |
|----------|-------|
| **Post Type Key** | `ik_exam` |
| **Supports** | `title` only |
| **Public** | No |
| **Show in REST** | Yes |
| **Title Placeholder** | "Enter Exam Name (e.g., Midterm 2025)" |

---

### 4. `ik_certificate` — Certificate Templates

| Property | Value |
|----------|-------|
| **Post Type Key** | `ik_certificate` |
| **Supports** | `title` only |
| **Public** | No |
| **Show in REST** | Yes |
| **Purpose** | Template designs for certificate generation |

Certificates themselves are stored in the `institutionkit_certificates` custom table — this CPT stores only the design templates.

---

### 5. `ik_period` — Grading Periods

| Property | Value |
|----------|-------|
| **Post Type Key** | `ik_period` |
| **Supports** | `title` only |
| **Public** | No |
| **Show in REST** | No |
| **Title Placeholder** | "Enter Period Name (e.g., October 2025 Monthly Assessment)" |

#### Meta Boxes

The `ik_period` CPT has a custom meta box with these fields:

| Meta Key | Type | Options | Description |
|----------|------|---------|-------------|
| `_ik_period_type` | VARCHAR | `weekly`, `monthly`, `quarterly`, `yearly`, `exam` | Period classification |
| `_ik_start_date` | DATE | — | Period start date |
| `_ik_end_date` | DATE | — | Period end date |
| `_ik_academic_year` | VARCHAR | e.g., "2025-2026" | Academic year |
| `_ik_weight` | DECIMAL(5,2) | Default: 1.00 | Weight for grade averaging |
| `_ik_is_published` | VARCHAR | `1` or `0` | Visible to students/parents |

#### Gradebook Integration

Grading periods are the backbone of the multi-period gradebook system. When a teacher enters grades, they select a period, and the data is stored in `ik_grades_v2` with the `period_id` and `period_type` columns.

---

## Custom Taxonomies

### 1. `ik_subject` — Subjects

| Property | Value |
|----------|-------|
| **Taxonomy Key** | `ik_subject` |
| **Object Types** | `ik_class` |
| **Hierarchical** | Yes (supports parent-child subjects) |
| **Show in REST** | Yes |
| **Show Admin Column** | Yes |
| **Rewrite Slug** | `subject` |

Subjects are assigned to **classes** (not directly to students). When a student is enrolled in a class, they inherit that class's subjects.

### 2. `ik_section` — Sections

| Property | Value |
|----------|-------|
| **Taxonomy Key** | `ik_section` |
| **Object Types** | `ik_class` |
| **Hierarchical** | No (flat) |
| **Show in REST** | Yes |
| **Show Admin Column** | Yes |
| **Rewrite Slug** | `section` |
| **Show in Menu** | No (added manually via `admin_menu` hook) |

Sections allow splitting a class into groups (e.g., "A", "B", "C", "Morning", "Afternoon"). The Sections submenu is added manually to the InstitutionKit menu:

```php
add_submenu_page(
    'ik-dashboard',
    'Sections',
    'Sections',
    'manage_options',
    'edit-tags.php?taxonomy=ik_section&post_type=ik_class'
);
```

---

## Registration Code

### CPT Registration (simplified)

```php
// Students
register_post_type('ik_student', [
    'labels'        => ['name' => 'Students', 'singular_name' => 'Student'],
    'public'        => true,
    'show_ui'       => true,
    'show_in_menu'  => 'ik-dashboard',
    'supports'      => ['title', 'thumbnail'],
    'menu_icon'     => 'dashicons-id',
    'show_in_rest'  => true,
]);

// Classes
register_post_type('ik_class', [
    'labels'        => ['name' => 'Classes', 'singular_name' => 'Class'],
    'public'        => true,
    'hierarchical'  => true,
    'show_in_menu'  => 'ik-dashboard',
    'supports'      => ['title'],
    'menu_icon'     => 'dashicons-groups',
    'show_in_rest'  => true,
]);

// Grading Periods
register_post_type('ik_period', [
    'labels'        => ['name' => 'Grading Periods', 'singular_name' => 'Grading Period'],
    'public'        => false,
    'show_ui'       => true,
    'show_in_menu'  => 'ik-dashboard',
    'capability_type' => 'post',
    'supports'      => ['title'],
    'has_archive'   => false,
]);
```

### Taxonomy Registration (simplified)

```php
// Subjects
register_taxonomy('ik_subject', ['ik_class'], [
    'hierarchical'      => true,
    'labels'            => ['name' => 'Subjects', 'singular_name' => 'Subject'],
    'show_admin_column' => true,
    'show_in_rest'      => true,
    'rewrite'           => ['slug' => 'subject'],
]);

// Sections
register_taxonomy('ik_section', ['ik_class'], [
    'hierarchical'      => false,
    'labels'            => ['name' => 'Sections', 'singular_name' => 'Section'],
    'show_admin_column' => true,
    'show_in_rest'      => true,
    'rewrite'           => ['slug' => 'section'],
    'show_in_menu'      => false, // Manually added to IK menu
]);
```

---

## Menu Integration

All CPTs and taxonomies are displayed under the **InstitutionKit** dashboard menu. The `IK_Admin_Menu` class handles menu highlighting so that when editing any IK post type or taxonomy, the IK parent menu stays highlighted:

```php
// From IK_Admin_Menu::fix_menu_highlight()
$ik_post_types  = ['ik_student', 'ik_class', 'ik_exam', 'ik_certificate', 'ik_period'];
$ik_taxonomies  = ['ik_subject', 'ik_section'];

if (in_array($current_screen->post_type, $ik_post_types) ||
    in_array($current_screen->taxonomy, $ik_taxonomies)) {
    return 'ik-dashboard'; // Highlight the IK menu
}
```

---

## Teacher CPT (Removed)

!!! danger "Historical Note"
    Earlier versions of InstitutionKit used an `ik_teacher` custom post type. This has been **completely removed** in version 1.2.0. All teacher/staff data now resides in the `institutionkit_staff` custom table managed by the Payroll & Expenses module.

The migration process:

1. All `ik_teacher` posts are migrated to `institutionkit_staff` rows
2. Teacher attendance records migrate to `institutionkit_staff_attendance`
3. Teacher comments and meeting slots update their `teacher_id` references from post IDs to staff IDs
4. Each migrated teacher post gets a `_ik_migrated_to_staff` post meta flag

---

## Retrieving Data

### Get Students by Class

```php
$args = [
    'post_type'      => 'ik_student',
    'posts_per_page' => -1,
    'meta_key'       => '_ik_student_class_id',
    'meta_value'     => $class_id,
    'orderby'        => 'title',
    'order'          => 'ASC',
];
$students = get_posts($args);
```

### Get Classes by Campus

```php
$args = [
    'post_type'      => 'ik_class',
    'posts_per_page' => -1,
    'meta_query'     => [
        'relation' => 'OR',
        ['key' => '_ik_campus_id', 'value' => $campus_id, 'compare' => '='],
        ['key' => '_ik_campus_id', 'compare' => 'NOT EXISTS'],
    ],
];
$classes = get_posts($args);
```

### Get Subjects for a Class

```php
$subjects = wp_get_post_terms($class_id, 'ik_subject');
```

### Get Sections for a Class

```php
$sections = wp_get_post_terms($class_id, 'ik_section');
```

### Get Grading Periods by Type

```php
$args = [
    'post_type'      => 'ik_period',
    'posts_per_page' => -1,
    'meta_key'       => '_ik_period_type',
    'meta_value'     => 'monthly',
];
$periods = get_posts($args);
```

---

## Adding Custom Meta to Students

To add extra fields to the student profile:

```php
// Add meta box
add_action('add_meta_boxes', function() {
    add_meta_box(
        'custom_student_fields',
        'Custom Information',
        'render_custom_student_fields',
        'ik_student',
        'normal',
        'default'
    );
});

// Save meta
add_action('save_post_ik_student', function($post_id) {
    if (isset($_POST['custom_blood_group'])) {
        update_post_meta($post_id, '_ik_blood_group', sanitize_text_field($_POST['custom_blood_group']));
    }
});
```
```
