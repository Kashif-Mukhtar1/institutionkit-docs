
# Installation

## System Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| **WordPress** | 5.8+ | 6.4+ |
| **PHP** | 7.4 | 8.1+ |
| **MySQL** | 5.7 | 8.0+ (or MariaDB 10.4+) |
| **PHP Memory Limit** | 128MB | 256MB+ |
| **PHP Max Execution Time** | 60s | 300s |
| **PHP Extensions** | `mysqli`, `curl`, `dom`, `mbstring`, `gd` | `zip`, `intl` |

!!! warning "Memory Limit"
    Multi-campus environments with large datasets should allocate **256MB+** PHP memory. Payroll generation and report card PDF generation are memory-intensive operations.

---

## Installation Steps

### 1. Upload the Plugin

=== "WordPress Admin (Recommended)"

    1. Download the `institutionkit.zip` file from your account
    2. Go to **Plugins → Add New → Upload Plugin**
    3. Choose the ZIP file and click **Install Now**
    4. Click **Activate Plugin**

=== "FTP / cPanel"

    1. Extract `institutionkit.zip` on your computer
    2. Upload the `institutionkit` folder to `/wp-content/plugins/`
    3. Go to **Plugins** in your WordPress admin
    4. Find "InstitutionKit" and click **Activate**

---

### 2. Automatic Table Creation

Upon activation, InstitutionKit automatically:

- Creates **30+ custom database tables** for students, fees, invoices, attendance, payroll, exams, and more
- Registers **5 custom post types** (`ik_student`, `ik_class`, `ik_exam`, `ik_certificate`, `ik_period`)
- Registers **2 taxonomies** (`ik_subject`, `ik_section`)
- Creates **5 user roles** with their capabilities
- Inserts **default data** (grade scales, meeting topics, expense heads)
- Creates the **default "Main Campus"**

!!! success "What to Expect"
    After activation, you'll be automatically redirected to the InstitutionKit Dashboard. A success message confirms that all tables were created.

---

### 3. Verify Installation

1. Go to **InstitutionKit → Dashboard** in the admin menu
2. You should see the dashboard with "Main Campus" as the default campus
3. Check **InstitutionKit → Campuses** — "Main Campus" should exist
4. Check **InstitutionKit → Setup** — you can now create classes, subjects, and sections

---

## Upgrading from an Earlier Version

### Version 1.1.x → 1.2.x

The 1.2.0 update includes a **Teacher CPT → Staff Table Migration**. The system:

- Migrates all `ik_teacher` posts to the `institutionkit_staff` table
- Migrates teacher attendance records
- Maps teacher IDs in comments and meeting slots

This runs automatically. Check the migration status in **InstitutionKit → Settings → System Status**.

!!! note "Manual Verification"
    After migration, verify that all teachers appear in **Payroll & Expenses → Staff**. Teacher management no longer uses WordPress posts.

---

## Post-Installation Checklist

- [ ] Activate license at **InstitutionKit → Settings → License**
- [ ] Set up classes, subjects, and sections in **InstitutionKit → Setup**
- [ ] Configure grading periods at **InstitutionKit → Grading Periods**
- [ ] Set campus details at **InstitutionKit → Campuses**
- [ ] Add staff members at **Payroll & Expenses → Staff**
- [ ] Configure fee types and structures at **Fee Management**
- [ ] Set school operating hours in **InstitutionKit → Settings**

---

## Troubleshooting Installation

### Tables Not Created

If tables fail to create automatically, run the manual table creation:

1. Go to **InstitutionKit → Settings → System Status**
2. Click **Recreate Tables**
3. If this fails, check your database user has `CREATE TABLE` privileges

### Memory Exhausted Errors

Add to `wp-config.php`:

```php
define('WP_MEMORY_LIMIT', '256M');
define('WP_MAX_MEMORY_LIMIT', '512M');
