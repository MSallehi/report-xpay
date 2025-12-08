# راهنمای Deploy روی cPanel (هاست اشتراکی)

## 📋 فهرست مطالب
1. [فایل‌های مورد نیاز](#فایل‌های-مورد-نیاز)
2. [تنظیمات .htaccess](#تنظیمات-htaccess)
3. [تست Cache Headers](#تست-cache-headers)
4. [مشکلات احتمالی](#مشکلات-احتمالی)

---

## 🔧 فایل‌های مورد نیاز

### 1. فایل .htaccess (ROOT سایت)
این فایل در root directory وردپرس (همان جایی که `wp-config.php` است) قرار دارد.

**مسیر:** `/public_html/.htaccess`

**بخش‌های اضافه شده:**
```apache
# BEGIN XPay Performance Optimization - Cache Headers
<IfModule mod_headers.c>
    # Cache headers for static assets (1 year)
    ...
</IfModule>

<IfModule mod_expires.c>
    # Expires headers (fallback)
    ...
</IfModule>

<IfModule mod_deflate.c>
    # Gzip compression
    ...
</IfModule>
# END XPay Performance Optimization
```

---

## 📝 تنظیمات .htaccess

### Cache Headers (1 Year)

**JS/CSS:**
```apache
<FilesMatch "\.(js|css)$">
    Header set Cache-Control "public, max-age=31536000, immutable"
    Header append Vary "Accept-Encoding"
</FilesMatch>
```

**Images:**
```apache
<FilesMatch "\.(jpg|jpeg|png|gif|ico|svg|webp)$">
    Header set Cache-Control "public, max-age=31536000, immutable"
</FilesMatch>
```

**Fonts (با CORS):**
```apache
<FilesMatch "\.(woff|woff2|ttf|eot|otf)$">
    Header set Cache-Control "public, max-age=31536000, immutable"
    Header set Access-Control-Allow-Origin "*"
</FilesMatch>
```

### چرا 1 سال؟
- ✅ Assets ما versioned هستند (مثل `app-vendor.v4.9.7.js`)
- ✅ وقتی فایل تغییر می‌کند، نام فایل عوض می‌شود
- ✅ مرورگر نیازی به revalidate ندارد → `immutable`
- ✅ صرفه‌جویی bandwidth: **165 KiB per visit**

---

## 🧪 تست Cache Headers

### روش 1: Chrome DevTools
```
1. باز کردن DevTools (F12)
2. رفتن به تب Network
3. Reload صفحه (Ctrl+R)
4. کلیک روی یک asset (مثل .js یا .css)
5. بررسی Headers:
   
   ✅ باید ببینید:
   Cache-Control: public, max-age=31536000, immutable
   Expires: Fri, 15 Nov 2026 [time] GMT
```

### روش 2: cURL Command
```bash
curl -I https://xpay.co/wp-content/themes/xpay_main_theme/assets/js/app-vendor.v4.9.7.js

# باید ببینید:
HTTP/2 200
cache-control: public, max-age=31536000, immutable
expires: Fri, 15 Nov 2026 15:45:00 GMT
vary: Accept-Encoding
```

### روش 3: آنلاین
```
رفتن به: https://redbot.org/
URL asset را وارد کنید
بررسی Cache Headers
```

---

## ⚠️ مشکلات احتمالی

### مشکل 1: Cache Headers اعمال نمی‌شوند

**علت:**
- `mod_headers` یا `mod_expires` غیرفعال است در Apache

**راه‌حل:**
1. تماس با پشتیبانی هاست برای فعال‌سازی:
   - `mod_headers`
   - `mod_expires`
   - `mod_deflate` (برای Gzip)

2. یا اضافه کردن به `.htaccess`:
```apache
<IfModule !mod_headers.c>
    LoadModule headers_module modules/mod_headers.so
</IfModule>
```

### مشکل 2: Cache Headers فقط 7 روز است

**علت:**
- Plugin کش (مثل LiteSpeed Cache) override می‌کند

**راه‌حل:**
1. رفتن به تنظیمات Plugin کش
2. افزایش Browser Cache TTL به 1 year (31536000)
3. یا غیرفعال کردن Browser Cache در plugin

**LiteSpeed Cache:**
```
Cache → Browser → Browser Cache TTL
تغییر به: 31536000 (1 year)
```

### مشکل 3: CORS Error برای Fonts

**علت:**
- Fonts از subdomain load می‌شوند (مثل `cdn.xpay.co`)

**راه‌حل:**
فایل `.htaccess` قبلاً شامل CORS است:
```apache
<FilesMatch "\.(woff|woff2|ttf|eot|otf)$">
    Header set Access-Control-Allow-Origin "*"
</FilesMatch>
```

اگر همچنان مشکل دارید:
```apache
# اضافه کردن برای همه assets
<IfModule mod_headers.c>
    SetEnvIf Origin "^http(s)?://(.+\.)?(xpay\.co)$" AccessControlAllowOrigin=$0
    Header set Access-Control-Allow-Origin %{AccessControlAllowOrigin}e env=AccessControlAllowOrigin
</IfModule>
```

### مشکل 4: صفحه 500 Error بعد از اضافه کردن .htaccess

**علت:**
- Syntax error در `.htaccess`
- Module مورد نیاز غیرفعال است

**راه‌حل:**
1. Backup کردن `.htaccess` قدیمی
2. تست کردن تدریجی:
   ```apache
   # فقط Cache-Control
   <FilesMatch "\.(js|css)$">
       Header set Cache-Control "public, max-age=31536000"
   </FilesMatch>
   ```
3. افزودن بقیه بخش‌ها یکی یکی
4. بررسی Error Log در cPanel

### مشکل 5: Assets Cache نمی‌شوند برای کاربران Login شده

**علت:**
- WordPress به طور پیش‌فرض cache را برای logged-in users غیرفعال می‌کند

**راه‌حل:**
این رفتار صحیح است! کد ما در `PageSpeedController.php` این را handle می‌کند:
```php
public function addCacheHeaders() {
    if (is_admin() || is_user_logged_in()) {
        return; // No cache for admin/logged-in users
    }
    // Cache headers...
}
```

---

## 📊 نتایج مورد انتظار

### قبل از بهینه‌سازی:
```
Cache-Control: max-age=604800 (7 days)
Transfer Size: 1,141 KiB
```

### بعد از بهینه‌سازی:
```
Cache-Control: public, max-age=31536000, immutable (1 year)
Transfer Size (Repeat Visit): 0 KiB (from cache)
```

### صرفه‌جویی:
- **Per Visitor:** 1,141 KiB
- **10,000 Visitors/Month:** ~11 GB bandwidth
- **PageSpeed Score:** +5-10 points
- **Load Time (Repeat Visitors):** -1.5s

---

## ✅ Checklist برای Deploy

- [ ] Backup کردن `.htaccess` فعلی
- [ ] اضافه کردن بخش "XPay Performance Optimization" به `.htaccess`
- [ ] Test کردن یک asset با cURL یا DevTools
- [ ] بررسی Cache Headers در PageSpeed Insights
- [ ] Clear کردن Cache Browser (Ctrl+Shift+Delete)
- [ ] Test کردن Repeat Visit (باید 0 KiB transfer باشد)
- [ ] بررسی عدم مشکل برای Logged-in Users

---

## 🆘 نیاز به کمک؟

اگر مشکلی پیش آمد:
1. Error Log در cPanel → Errors
2. بررسی Apache version: `php -i | grep Apache`
3. بررسی Loaded Modules: `php -i | grep mod_`
4. تماس با پشتیبانی هاست با این اطلاعات:
   - "لطفاً mod_headers و mod_expires را فعال کنید"
   - "برای بهینه‌سازی Browser Cache نیاز است"

---

**تاریخ:** 15 نوامبر 2025  
**نسخه:** 4.9.9  
**نویسنده:** XPay Development Team
