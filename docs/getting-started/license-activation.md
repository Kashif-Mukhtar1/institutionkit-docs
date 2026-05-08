# License Activation

InstitutionKit requires a valid license to receive updates, access premium support, and unlock all features. Without activation, the system displays a license reminder banner on all InstitutionKit pages.

---

## How the License System Works

| License State | What You See |
|---------------|--------------|
| **Inactive** | Yellow banner on all IK pages: "⚠️ License Required: Please activate your InstitutionKit license..." |
| **Active** | No banner. Full access to updates and support. |
| **Expired** | Warning banner. Updates are paused but the system continues working. |

!!! info "No Feature Lockout"
    InstitutionKit **does not** disable features when the license is inactive. The banner is a reminder — all functionality remains available.

---

## Where to Get Your License Key

1. Purchase a license from [institutionkit.com](https://institutionkit.com)
2. Check your email for the license key
3. The key format is: `IK-XXXX-XXXX-XXXX-XXXX`

---

## Activating Your License

### Step 1: Open the License Page

1. Go to **InstitutionKit → Settings**
2. Look for the license section, or
3. Navigate directly to `wp-admin/admin.php?page=ik-license`

### Step 2: Enter Your License Key

1. Enter your license key in the input field
2. Click **Activate License**

### Step 3: Verify Activation

After successful activation:

- The warning banner disappears from all InstitutionKit pages
- The license page shows: **"✅ License Active"**
- You can now receive automatic updates

---

## Troubleshooting

### "Invalid License Key"

| Cause | Solution |
|-------|----------|
| Typo in the key | Copy-paste directly from your email |
| Wrong domain | License is tied to your registered domain. Contact support to update |
| Expired license | Renew your license at institutionkit.com |

### "Could Not Connect to License Server"

| Cause | Solution |
|-------|----------|
| Firewall blocking outbound connections | Ask your host to allow outbound connections to `institutionkit.com` |
| Server offline | Wait and try again. The system caches the license locally |
| cURL not installed | Contact your host to enable the PHP cURL extension |

### "License Already in Use"

Your license key is active on another domain. Contact support to transfer it.

---

## Checking License Status Programmatically

```php
// Get license status directly from the database
global $wpdb;
$status = $wpdb->get_var(
    $wpdb->prepare(
        "SELECT option_value FROM {$wpdb->options} WHERE option_name = %s",
        'ik_license_status'
    )
);

if (trim($status) === 'active') {
    echo "License is active";
}
