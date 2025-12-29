# 🔧 رفع مشکل ذخیره تنظیمات PageSpeed Admin

> **تاریخ:** ژانویه 2025  
> **نوع:** Bug Fix  
> **فایل‌های تغییر یافته:** 2 فایل

---

## 📋 مشکل

تنظیمات PageSpeed در پنل ادمین وردپرس ذخیره نمی‌شدند. با کلیک روی دکمه "ذخیره تغییرات"، تنظیمات به درستی ارسال نمی‌شدند و مقادیر چک‌باکس‌ها همیشه `false` ذخیره می‌شد.

### علائم:
- چک‌باکس‌ها فعال می‌شدند اما بعد از refresh صفحه غیرفعال بودند
- AJAX request موفق بود اما تنظیمات ذخیره نمی‌شد
- تمام boolean values به `false` تبدیل می‌شدند

---

## 🔍 علت اصلی

### مشکل 1: JavaScript - ارسال boolean به جای string

فایل: `assets/admin/pagespeed-admin.js`

**کد مشکل‌دار:**
```javascript
const data = {
    action: "save_pagespeed_settings",
    nonce: pagespeedAdmin.nonce,
    enable_lcp_optimization: $("#enable_lcp_optimization").is(":checked"),  // ⛔ boolean
    enable_asset_versioning: $("#enable_asset_versioning").is(":checked"),  // ⛔ boolean
    // ...
};
```

**مشکل:** jQuery AJAX وقتی `true/false` را ارسال می‌کند، ممکن است آن را به string `"true"/"false"` تبدیل کند یا اصلاً ارسال نکند (در حالت `false`).

### مشکل 2: PHP - عدم تبدیل صحیح به boolean

فایل: `app/Admin/PageSpeedAdmin.php`

**کد مشکل‌دار:**
```php
private function saveLCPSettings()
{
    update_option('xpay_lcp_optimization', $_POST['enable_lcp_optimization'] ?? false);
    // ...
}
```

**مشکل:** مقدار `"true"` یا `"1"` یا `"false"` به صورت string ذخیره می‌شد نه boolean.

---

## ✅ راه‌حل

### فیکس 1: JavaScript - ارسال "1"/"0"

فایل: `assets/admin/pagespeed-admin.js`

```javascript
jQuery(document).ready(function ($) {
    // Helper function برای تبدیل checkbox به "1" یا "0"
    const getCheckboxValue = (id) => $(`#${id}`).is(":checked") ? "1" : "0";

    $("#pagespeed-settings-form").on("submit", function (e) {
        e.preventDefault();

        const data = {
            action: "save_pagespeed_settings",
            nonce: pagespeedAdmin.nonce,
            
            // ✅ استفاده از "1"/"0" به جای boolean
            enable_lcp_optimization: getCheckboxValue("enable_lcp_optimization"),
            enable_asset_versioning: getCheckboxValue("enable_asset_versioning"),
            enable_preload: getCheckboxValue("enable_preload"),
            enable_deferred_loading: getCheckboxValue("enable_deferred_loading"),
            enable_critical_css_inline: getCheckboxValue("enable_critical_css_inline"),
            enable_performance_optimizer: getCheckboxValue("enable_performance_optimizer"),
            enable_forced_reflow_prevention: getCheckboxValue("enable_forced_reflow_prevention"),
            enable_inp_optimizer: getCheckboxValue("enable_inp_optimizer"),

            // ... سایر فیلدها
        };

        $.post(pagespeedAdmin.ajaxUrl, data, function (response) {
            // ...
        });
    });
});
```

### فیکس 2: PHP - استفاده از filter_var

فایل: `app/Admin/PageSpeedAdmin.php`

```php
public function saveSettings()
{
    // Security check
    if (!check_ajax_referer('pagespeed_nonce', 'nonce', false)) {
        wp_send_json_error(['message' => 'Security check failed']);
        return;
    }

    // ✅ Helper function برای parse کردن boolean
    $parseBool = function($key) {
        return filter_var(
            $_POST[$key] ?? false, 
            FILTER_VALIDATE_BOOLEAN
        );
    };

    // ✅ استفاده از parseBool
    $this->saveLCPSettings($parseBool);
    $this->saveAssetSettings($parseBool);
    $this->saveOptimizationSettings($parseBool);

    wp_send_json_success([
        'message' => 'Settings saved successfully',
        'debug' => [
            'lcp' => $parseBool('enable_lcp_optimization'),
            'versioning' => $parseBool('enable_asset_versioning'),
            // ...
        ]
    ]);
}

private function saveLCPSettings($parseBool)
{
    // ✅ تبدیل صحیح به boolean
    update_option('xpay_lcp_optimization', $parseBool('enable_lcp_optimization'));
    update_option('xpay_preload_enabled', $parseBool('enable_preload'));
    update_option('xpay_deferred_loading', $parseBool('enable_deferred_loading'));
    update_option('xpay_critical_css_inline', $parseBool('enable_critical_css_inline'));
}

private function saveAssetSettings($parseBool)
{
    update_option('xpay_asset_versioning', $parseBool('enable_asset_versioning'));
}

private function saveOptimizationSettings($parseBool)
{
    update_option('xpay_performance_optimizer', $parseBool('enable_performance_optimizer'));
    update_option('xpay_forced_reflow_prevention', $parseBool('enable_forced_reflow_prevention'));
    update_option('xpay_inp_optimizer', $parseBool('enable_inp_optimizer'));
}
```

---

## 🔬 چرا `filter_var` با `FILTER_VALIDATE_BOOLEAN`؟

این function مقادیر مختلف را به boolean تبدیل می‌کند:

| ورودی | خروجی |
|-------|-------|
| `"1"` | `true` |
| `"0"` | `false` |
| `"true"` | `true` |
| `"false"` | `false` |
| `"yes"` | `true` |
| `"no"` | `false` |
| `"on"` | `true` |
| `"off"` | `false` |
| `1` | `true` |
| `0` | `false` |
| `true` | `true` |
| `false` | `false` |
| `null` | `false` |
| `""` | `false` |

این روش بهترین راه برای handle کردن انواع مختلف ورودی‌های boolean است.

---

## 📝 درس‌های آموخته

### 1. عدم اعتماد به jQuery AJAX برای boolean

jQuery AJAX رفتار متفاوتی برای boolean دارد:
- در برخی نسخه‌ها `false` اصلاً ارسال نمی‌شود
- در برخی نسخه‌ها به string تبدیل می‌شود

**بهترین روش:** همیشه از `"1"/"0"` برای checkbox values استفاده کنید.

### 2. استفاده از filter_var در PHP

به جای:
```php
$value = $_POST['field'] === 'true';
$value = (bool)$_POST['field'];
```

از این استفاده کنید:
```php
$value = filter_var($_POST['field'], FILTER_VALIDATE_BOOLEAN);
```

### 3. Debug Response

همیشه در پاسخ AJAX، مقادیر دریافتی را برگردانید:
```php
wp_send_json_success([
    'message' => 'Success',
    'debug' => [
        'received' => $_POST['field'],
        'parsed' => $parsedValue
    ]
]);
```

---

## 🧪 تست

### تست دستی:

1. به `wp-admin` بروید
2. منوی "PageSpeed Settings" را باز کنید
3. یک چک‌باکس را فعال کنید
4. روی "ذخیره تغییرات" کلیک کنید
5. صفحه را refresh کنید
6. چک‌باکس باید همچنان فعال باشد

### تست با Console:

```javascript
// بررسی مقدار ارسالی
jQuery("#pagespeed-settings-form").on("submit", function(e) {
    console.log("Sending:", jQuery("#enable_lcp_optimization").is(":checked") ? "1" : "0");
});
```

### تست PHP:

```php
// در saveSettings() اضافه کنید
error_log('Received: ' . print_r($_POST, true));
error_log('Parsed LCP: ' . (filter_var($_POST['enable_lcp_optimization'], FILTER_VALIDATE_BOOLEAN) ? 'true' : 'false'));
```

---

## 📁 فایل‌های تغییر یافته

| فایل | تغییرات |
|------|---------|
| `assets/admin/pagespeed-admin.js` | استفاده از helper function برای checkbox values |
| `app/Admin/PageSpeedAdmin.php` | استفاده از `filter_var` با `FILTER_VALIDATE_BOOLEAN` |

---

## 🔗 منابع

- [PHP filter_var Documentation](https://www.php.net/manual/en/function.filter-var.php)
- [FILTER_VALIDATE_BOOLEAN](https://www.php.net/manual/en/filter.filters.validate.php)
- [jQuery.ajax() Data Types](https://api.jquery.com/jquery.ajax/)
