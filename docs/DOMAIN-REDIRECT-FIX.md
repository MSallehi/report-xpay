# راهنمای رفع مشکل Redirect از Staging به Production

## مشکل
بعد از deploy کردن روی production، دیگر نمی‌توان به staging دسترسی پیدا کرد و همه چیز به xpay.co redirect می‌شود.

## راه‌حل‌های پیاده‌شده

### 1. تنظیم Dynamic Site URL در `wp-config.php`

```php
// wp-config.php - Lines 98-116
if (isset($_SERVER['HTTP_HOST'])) {
    $protocol = (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off') ? 'https://' : 'http://';
    $current_domain = $_SERVER['HTTP_HOST'];
    
    define('WP_HOME', $protocol . $current_domain);
    define('WP_SITEURL', $protocol . $current_domain);
    
    if (strpos($current_domain, 'staging') !== false) {
        define('WP_ENV', 'staging');
    } else {
        define('WP_ENV', 'production');
    }
}
```

**نتیجه:** WordPress همیشه از domain جاری استفاده می‌کند، نه از database.

### 2. فیلترهای URL در `functions.php`

```php
// functions.php - Lines 31-98

// 1. Force correct siteurl/home options
add_filter('option_siteurl', ...);
add_filter('option_home', ...);

// 2. Fix canonical redirects
add_filter('redirect_canonical', ...);

// 3. Intercept all wp_redirect calls
add_filter('wp_redirect', ...);

// 4. Clear cache on domain change
add_action('init', ...);
```

**نتیجه:** تمام redirect ها به domain صحیح هدایت می‌شوند.

### 3. استفاده از فایل `fix-domain-urls.php`

اگر مشکل همچنان وجود دارد، URLs در database باید update شوند:

#### روش 1: از طریق Browser
```
https://staging.xpay.co/wp-content/themes/xpay_main_theme/fix-domain-urls.php?run=1&key=xpay2024
```

#### روش 2: از طریق WP-CLI
```bash
cd /path/to/wordpress
wp eval-file wp-content/themes/xpay_main_theme/fix-domain-urls.php
```

**⚠️ مهم:** این فایل را بعد از استفاده حذف کنید!

### 4. Manual Database Update (Alternative)

اگر نمی‌خواهید از فایل PHP استفاده کنید:

```sql
-- Update siteurl and home
UPDATE wp_options 
SET option_value = 'https://staging.xpay.co' 
WHERE option_name IN ('siteurl', 'home');

-- Update post content
UPDATE wp_posts 
SET post_content = REPLACE(post_content, 'https://xpay.co', 'https://staging.xpay.co'),
    guid = REPLACE(guid, 'https://xpay.co', 'https://staging.xpay.co');

-- Update post meta
UPDATE wp_postmeta 
SET meta_value = REPLACE(meta_value, 'https://xpay.co', 'https://staging.xpay.co');

-- Clear transients
DELETE FROM wp_options WHERE option_name LIKE '_transient_%';
```

### 5. پاک کردن Cache

بعد از هر تغییر:

```bash
# Redis Cache
wp cache flush

# WP Optimize
wp optimize clear-cache

# Rewrite Rules
wp rewrite flush
```

یا از WordPress Admin:
- Settings > Permalinks > Save Changes (بدون تغییر)
- Redis Cache > Flush Cache
- WP Optimize > Clear Cache

## تست کردن

بعد از اعمال تغییرات:

1. **Clear Browser Cache**: Ctrl+Shift+Del
2. **Test URLs**:
   - ✅ https://staging.xpay.co
   - ✅ https://staging.xpay.co/coin/tron/
   - ✅ https://xpay.co
   - ✅ https://xpay.co/coin/tron/

3. **Check Logs**: `/wp-content/debug.log`
   ```
   [XPay Redirect Fix] Prevented redirect from staging.xpay.co to xpay.co
   [XPay Cache] Domain changed from xpay.co to staging.xpay.co - Cache flushed
   ```

## Debug Mode

برای troubleshooting بیشتر، در `wp-config.php`:

```php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);
```

سپس logs را در `/wp-content/debug.log` بررسی کنید.

## پیشگیری از مشکل در آینده

### هنگام Deploy به Production:

1. **Better Search Replace** plugin را استفاده کنید:
   - Search: `https://staging.xpay.co`
   - Replace: `https://xpay.co`
   - Tables: Select All
   - ✅ Run as dry run first

2. **یا از WP-CLI**:
   ```bash
   wp search-replace 'https://staging.xpay.co' 'https://xpay.co' --all-tables
   ```

3. **Clear all caches** بعد از deploy.

### هنگام بازگشت به Staging:

1. **Restore database** از backup
2. **یا run** `fix-domain-urls.php`
3. **Clear caches**

## Files Changed

- ✅ `/wp-config.php` - Dynamic WP_HOME/WP_SITEURL
- ✅ `/wp-content/themes/xpay_main_theme/functions.php` - URL filters
- ✅ `/wp-content/themes/xpay_main_theme/fix-domain-urls.php` - Helper script (⚠️ delete after use)

## Support

اگر مشکل همچنان وجود دارد:
1. Check `/wp-content/debug.log`
2. Check nginx/apache error logs
3. Check browser Network tab for redirect chains
4. Verify database wp_options: `siteurl` and `home`

---

**نسخه مستندات:** 1.0  
**تاریخ:** نوامبر 16, 2025  
**تست شده روی:** WordPress 6.4+, PHP 8.x
