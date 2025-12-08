# 🌍 راهنمای GeoLocation

سیستم تشخیص موقعیت جغرافیایی کاربران در قالب XPay.

---

## 📋 فهرست مطالب

1. [معرفی](#معرفی)
2. [نصب و راه‌اندازی](#نصب-و-راهاندازی)
3. [استفاده](#استفاده)
4. [پیکربندی](#پیکربندی)
5. [Helper Functions](#helper-functions)
6. [Airdrop و IP ایران](#airdrop-و-ip-ایران)
7. [Troubleshooting](#troubleshooting)
8. [Changelog](#changelog)

---

## معرفی

سیستم GeoLocation از پکیج [`msallehi/php-geolocation`](https://packagist.org/packages/msallehi/php-geolocation) استفاده می‌کند و قابلیت‌های زیر را فراهم می‌کند:

- ✅ تشخیص کشور کاربر از روی IP
- ✅ محدود کردن دسترسی به کشورهای خاص (مثل ایران)
- ✅ پشتیبانی از چندین API Provider
- ✅ Fallback خودکار در صورت خطای API
- ✅ کش WordPress Transient برای کاهش درخواست‌ها
- ✅ IP Ranges داخلی برای ایران (بدون نیاز به API)

---

## نصب و راه‌اندازی

### نصب پکیج

```bash
cd wp-content/themes/xpay_main_theme
composer require msallehi/php-geolocation
```

### آپدیت به آخرین ورژن

```bash
composer update msallehi/php-geolocation
```

---

## استفاده

### استفاده مستقیم از کلاس

```php
use XPayMain\Support\GeoLocation;

// چک کردن IP کاربر فعلی
$isIran = GeoLocation::isIranIp();  // true, false, or null

// چک کردن IP خاص
$isIran = GeoLocation::isIranIp('5.160.139.15');

// گرفتن کد کشور
$country = GeoLocation::getCountryCode();  // 'IR', 'US', etc.

// گرفتن جزئیات کامل
$details = GeoLocation::getLocationDetails();
// [
//     'ip' => '5.160.139.15',
//     'country_code' => 'IR',
//     'country_name' => 'Iran',
//     'city' => 'Tehran',
//     'region' => 'Tehran',
// ]
```

### استفاده از Helper Functions

```php
// چک کردن IP ایران
if (xpay_is_iran_ip()) {
    echo 'کاربر از ایران است';
}

// نسخه امن (هیچوقت exception نمی‌زنه)
if (xpay_is_allowed_safe()) {
    echo 'کاربر مجاز است';
}

// اطلاعات دیباگ
$debug = xpay_geo_debug_info();
print_r($debug);

// پاک کردن کش
xpay_geo_clear_cache();  // همه
xpay_geo_clear_cache('5.160.139.15');  // IP خاص
```

---

## پیکربندی

### تنظیمات پیش‌فرض

کلاس `GeoLocation` با تنظیمات زیر کار می‌کند:

```php
$defaultConfig = [
    'allowed_countries' => ['IR'],
    'api_provider' => 'ip-api',
    'timeout' => 5,
    'connect_timeout' => 3,      // v1.1.0
    'retry_count' => 2,          // v1.1.0
    'fallback_allow' => true,    // v1.1.0 - مهم!
    'allow_local' => true,
    'local_country' => 'IR',     // IP لوکال = ایران
];
```

### `fallback_allow` (مهم!)

این تنظیم در ورژن 1.1.0 اضافه شده:

- `true` (پیش‌فرض): اگر API خطا بده، کاربر رو **اجازه** بده
- `false`: اگر API خطا بده، کاربر رو **بلاک** کن

> ⚠️ **توصیه:** در production حتماً `fallback_allow: true` باشد تا کاربران ایرانی به خاطر مشکلات API بلاک نشوند.

### API Providers

پکیج از چندین provider پشتیبانی می‌کند:

1. **ip-api.com** (پیش‌فرض - رایگان، 45 درخواست/دقیقه)
2. **ipinfo.io** (نیاز به Token)
3. **ipdata.co** (نیاز به API Key)

---

## Helper Functions

| تابع | توضیحات |
|------|---------|
| `xpay_is_iran_ip($ip, $useCache)` | چک کردن آیا IP از ایران است |
| `xpay_is_allowed_safe($ip)` | نسخه امن - هیچوقت exception نمی‌زنه |
| `xpay_geo_debug_info()` | اطلاعات دیباگ برای troubleshooting |
| `xpay_geo_clear_cache($ip)` | پاک کردن کش GeoLocation |

---

## Airdrop و IP ایران

### نحوه کار

1. **PHP**: موقع لود صفحه، `xpay_is_iran_ip()` صدا زده می‌شه
2. **JavaScript**: مقدار به `window.giftBox.isIranIp` ارسال می‌شه
3. **Validation**: در `gift-box.js` چک می‌شه

### لاجیک بررسی

```javascript
// فقط اگر صریحاً false باشد بلاک کن
if (isIranIp === false) {
  // بلاک
}

// true یا null = اجازه
// (null یعنی API خطا داده، benefit of the doubt)
```

### مشکل VPN و راه‌حل

**مشکل:** کاربر VPN داره، ارور می‌گیره، VPN رو خاموش می‌کنه، ولی هنوز ارور می‌گیره.

**علت:** مقدار `isIranIp` موقع لود صفحه cache شده.

**راه‌حل:** وقتی کاربر روی دکمه "دریافت جایزه" کلیک می‌کنه و IP خارج تشخیص داده شده، اتوماتیک با AJAX دوباره چک می‌شه:

```javascript
// در gift-box.js - تابع recheckIranIpAndContinue
// وقتی isIranIp === false هست، AJAX درخواست می‌زنه
// اگر IP جدید ایران بود، ادامه flow رو اجرا می‌کنه
```

### AJAX Endpoint

در `functions.php`:

```php
add_action('wp_ajax_recheck_iran_ip', 'handle_recheck_iran_ip');
add_action('wp_ajax_nopriv_recheck_iran_ip', 'handle_recheck_iran_ip');

function handle_recheck_iran_ip() {
    check_ajax_referer('airdrop_nonce', 'nonce');
    
    // پاک کردن کش
    xpay_geo_clear_cache();
    
    // چک جدید بدون کش
    $isIranIp = xpay_is_iran_ip(null, false);
    
    wp_send_json_success([
        'isIranIp' => $isIranIp,
        'message' => $isIranIp ? 'موقعیت تأیید شد' : 'خارج از ایران'
    ]);
}
```

---

## Troubleshooting

### مشکل: کاربران ایران بلاک می‌شن

**علت احتمالی:**
1. Rate limit API (45 req/min برای ip-api.com)
2. API timeout
3. مشکل شبکه

**راه‌حل:**
- مطمئن شوید `fallback_allow: true` است
- از کش WordPress استفاده کنید
- IP ranges داخلی برای fallback وجود دارد

### مشکل: کاربران خارج بلاک نمی‌شن

**علت احتمالی:**
1. API fail کرده و `fallback_allow: true` است
2. کش قدیمی

**راه‌حل:**
```php
// پاک کردن کش
xpay_geo_clear_cache();

// چک بدون کش
$result = xpay_is_iran_ip(null, false);
```

### دیباگ کردن

```php
// گرفتن اطلاعات کامل
$debug = xpay_geo_debug_info();
error_log(print_r($debug, true));

// خروجی:
// [
//     'ip' => '5.160.139.15',
//     'is_public_ip' => true,
//     'cached_result' => 'iran',
//     'country' => 'IR',
//     'is_allowed' => true,
//     'in_iran_ip_ranges' => true,
//     'package_version' => '1.1.0',
// ]
```

---

## Changelog

### v1.1.0 (2024-12-02)

#### ✨ فیچرهای جدید
- **Fallback Allow**: گزینه `fallback_allow` - اگر API fail کرد، کاربر رو allow کن
- **Multiple API Fallback**: اگر API اول fail کرد، API بعدی امتحان می‌شه
- **Retry Mechanism**: تنظیم `retry_count` برای retry خودکار
- **Connect Timeout**: تایم‌اوت جداگانه برای connection سریع‌تر
- **In-Memory Cache**: کش داخلی برای کاهش API calls
- **geo_debug_info()**: تابع دیباگ برای troubleshooting
- **geo_is_allowed_safe()**: تابع امن که هیچوقت exception نمی‌زنه

#### 🐛 باگ‌های فیکس شده
- **Critical Bug**: کاربران ایران وقتی API timeout می‌شد بلاک می‌شدن
- **WordPress Integration**: wp_die() بدون توضیح مناسب

#### 🔄 تغییرات
- رفتار پیش‌فرض: اگر API fail کرد، اجازه بده (`fallback_allow: true`)
- WordPress functions با `fallback_allow: true` پیش‌فرض

### v1.0.0 (2024-12-01)

- انتشار اولیه
- تشخیص کشور از IP
- پشتیبانی از ip-api, ipinfo, ipdata
- WordPress و Laravel integration

---

## 📚 منابع

- [پکیج msallehi/php-geolocation](https://packagist.org/packages/msallehi/php-geolocation)
- [مستندات GitHub](https://github.com/MSallehi/php-geolocation)
- [لیست IP های ایران](https://github.com/AlirezaF80/iran-ip-ranges)

---

## 👨‍💻 نویسنده

- **Mohammad Salehi** - [GitHub](https://github.com/MSallehi)
