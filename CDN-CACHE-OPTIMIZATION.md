# 🚀 راهنمای بهینه‌سازی Cache در CDN

## 📋 مقدمه

این مستند راهنمای کامل برای تنظیم cache lifetime برای فایل‌های استاتیک روی CDN را ارائه می‌دهد. با پیاده‌سازی صحیح این تنظیمات، PageSpeed Insights دیگر هشدار "Use efficient cache lifetimes" نمی‌دهد.

---

## 🎯 مشکل

### گزارش PageSpeed Insights:
```
⚠️ Use efficient cache lifetimes
Est savings of 38 KiB

Request                          Cache TTL    Transfer Size
cdn.xpay.co/UUSD.webp                 7d          11 KiB
cdn.xpay.co/TRX.webp                  7d           5 KiB
cdn.xpay.co/BTC.webp                  7d           4 KiB
cdn.xpay.co/USDT.webp                 7d           4 KiB
```

**مشکل اصلی:** فایل‌های تصویری روی CDN فقط ۷ روز (7 days) cache می‌شوند.  
**استاندارد Google:** حداقل ۱ سال (1 year) برای فایل‌های استاتیک.

---

## ✅ راه‌حل

### فایل `.htaccess` برای CDN

یک فایل `.htaccess` در روت CDN (مثلاً `cdn.xpay.co`) با تنظیمات زیر ایجاد کنید:

```apache
# XPay CDN Cache Configuration
# این فایل باید در روت CDN به عنوان .htaccess قرار گیرد

<IfModule mod_headers.c>
    # حذف ETags (ما از strong cache با max-age استفاده می‌کنیم)
    Header unset ETag
    FileETag None
    
    # تصاویر - Cache 1 ساله (31536000 ثانیه)
    <FilesMatch "\.(jpg|jpeg|png|gif|ico|svg|webp|avif)$">
        Header set Cache-Control "public, max-age=31536000, immutable"
        Header set Expires "Thu, 31 Dec 2026 23:59:59 GMT"
    </FilesMatch>
    
    # CSS/JS - Cache 1 ساله
    <FilesMatch "\.(css|js)$">
        Header set Cache-Control "public, max-age=31536000, immutable"
        Header set Expires "Thu, 31 Dec 2026 23:59:59 GMT"
        Header append Vary "Accept-Encoding"
    </FilesMatch>
    
    # فونت‌ها - Cache 1 ساله با CORS
    <FilesMatch "\.(woff|woff2|ttf|eot|otf)$">
        Header set Cache-Control "public, max-age=31536000, immutable"
        Header set Expires "Thu, 31 Dec 2026 23:59:59 GMT"
        Header set Access-Control-Allow-Origin "*"
    </FilesMatch>
    
    # فایل‌های مدیا - Cache 1 ساله
    <FilesMatch "\.(mp4|webm|ogg|mp3|wav|pdf)$">
        Header set Cache-Control "public, max-age=31536000"
        Header set Expires "Thu, 31 Dec 2026 23:59:59 GMT"
    </FilesMatch>
    
    # CORS برای تمام فایل‌ها
    Header set Access-Control-Allow-Origin "*"
    Header set Access-Control-Allow-Methods "GET, OPTIONS"
    Header set Access-Control-Allow-Headers "Content-Type"
</IfModule>

# Expires Headers (fallback برای Apache قدیمی)
<IfModule mod_expires.c>
    ExpiresActive On
    
    # تصاویر - 1 سال
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/jpg "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType image/gif "access plus 1 year"
    ExpiresByType image/svg+xml "access plus 1 year"
    ExpiresByType image/webp "access plus 1 year"
    ExpiresByType image/avif "access plus 1 year"
    ExpiresByType image/x-icon "access plus 1 year"
    
    # CSS/JS - 1 سال
    ExpiresByType text/css "access plus 1 year"
    ExpiresByType text/javascript "access plus 1 year"
    ExpiresByType application/javascript "access plus 1 year"
    ExpiresByType application/x-javascript "access plus 1 year"
    
    # فونت‌ها - 1 سال
    ExpiresByType font/woff "access plus 1 year"
    ExpiresByType font/woff2 "access plus 1 year"
    ExpiresByType font/ttf "access plus 1 year"
    ExpiresByType font/otf "access plus 1 year"
    ExpiresByType application/font-woff "access plus 1 year"
    ExpiresByType application/font-woff2 "access plus 1 year"
    
    # مدیا - 1 سال
    ExpiresByType video/mp4 "access plus 1 year"
    ExpiresByType video/webm "access plus 1 year"
    ExpiresByType audio/mp3 "access plus 1 year"
    ExpiresByType application/pdf "access plus 1 year"
</IfModule>

# فشرده‌سازی Gzip
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/css text/javascript text/xml text/plain
    AddOutputFilterByType DEFLATE application/javascript application/x-javascript application/json
    AddOutputFilterByType DEFLATE application/xml application/xhtml+xml
    AddOutputFilterByType DEFLATE image/svg+xml
</IfModule>

# Security Headers برای CDN
<IfModule mod_headers.c>
    Header set X-Content-Type-Options "nosniff"
    Header set Referrer-Policy "strict-origin-when-cross-origin"
</IfModule>
```

---

## 📁 مراحل نصب

### 1️⃣ آپلود به سرور CDN

**روش A: از طریق cPanel:**
```bash
1. وارد cPanel شوید
2. به File Manager بروید
3. به مسیر public_html/cdn.xpay.co بروید (یا هر مسیری که CDN روی آن است)
4. فایل جدیدی با نام .htaccess بسازید
5. محتوای فایل .htaccess-cdn را در آن paste کنید
6. ذخیره کنید
```

**روش B: از طریق FTP:**
```bash
1. به سرور با FileZilla یا WinSCP متصل شوید
2. به مسیر cdn.xpay.co بروید
3. فایل .htaccess را آپلود کنید
```

### 2️⃣ تست Cache Headers

بعد از آپلود، با curl تست کنید:

```bash
curl -I https://cdn.xpay.co/coins/images/webp/UUSD.webp
```

**خروجی مورد انتظار:**
```http
HTTP/2 200
cache-control: public, max-age=31536000, immutable
expires: Thu, 31 Dec 2026 23:59:59 GMT
x-content-type-options: nosniff
access-control-allow-origin: *
```

✅ **باید `max-age=31536000` (1 سال) را ببینید به جای `max-age=604800` (7 روز)**

---

## 🔧 تنظیمات Cloudflare (اگر استفاده می‌کنید)

اگر `cdn.xpay.co` از **Cloudflare** استفاده می‌کند:

### 1. وارد Dashboard Cloudflare شوید
### 2. به بخش **Caching → Configuration** بروید
### 3. تنظیمات زیر را اعمال کنید:

```yaml
Browser Cache TTL: 1 year
Edge Cache TTL: 1 month
```

### 4. Cache Rules ایجاد کنید:

```javascript
// Rule برای تصاویر
If: (http.request.uri.path matches ".*\.(jpg|jpeg|png|gif|webp|svg|ico)$")
Then: Cache TTL = 1 year, Browser TTL = 1 year

// Rule برای فونت‌ها
If: (http.request.uri.path matches ".*\.(woff|woff2|ttf|eot|otf)$")
Then: Cache TTL = 1 year, Browser TTL = 1 year

// Rule برای CSS/JS
If: (http.request.uri.path matches ".*\.(css|js)$")
Then: Cache TTL = 1 year, Browser TTL = 1 year
```

---

## 📊 توضیح تنظیمات

### `Cache-Control: public, max-age=31536000, immutable`

| پارامتر | توضیح |
|---------|-------|
| `public` | فایل می‌تواند توسط CDN و browser cache شود |
| `max-age=31536000` | مدت زمان cache: 31536000 ثانیه = 1 سال |
| `immutable` | فایل **هرگز** تغییر نمی‌کند (نیازی به revalidate ندارد) |

### چرا 1 سال؟

- **استاندارد Google:** فایل‌های استاتیک باید حداقل 1 سال cache شوند
- **بهترین عملکرد:** مرورگر فایل را **هرگز** دوباره دانلود نمی‌کند
- **صرفه‌جویی bandwidth:** کاهش بار سرور و CDN

### چگونه فایل را بروزرسانی کنیم؟

چون فایل‌ها 1 سال cache می‌شوند، برای بروزرسانی باید **نام فایل را تغییر دهید**:

```
❌ قبل: cdn.xpay.co/UUSD.webp
✅ بعد: cdn.xpay.co/UUSD.webp?v=2
```

یا از versioning استفاده کنید:

```
✅ cdn.xpay.co/UUSD-v2.webp
```

---

## 🧪 تست و بررسی

### 1. PageSpeed Insights
```
https://pagespeed.web.dev/analysis?url=https://staging.xpay.co/coin/uvoucher/
```

**نتیجه مورد انتظار:**
```
✅ Use efficient cache lifetimes
    All static assets are cached for at least 1 year
```

### 2. Chrome DevTools
1. صفحه را باز کنید
2. F12 را بفشارید
3. به تب **Network** بروید
4. صفحه را Refresh کنید (Ctrl+Shift+R)
5. روی یک تصویر از CDN کلیک کنید
6. به بخش **Headers → Response Headers** بروید
7. باید `cache-control: public, max-age=31536000, immutable` را ببینید

### 3. بررسی با Online Tools

**WebPageTest:**
```
https://www.webpagetest.org/
```

**GTmetrix:**
```
https://gtmetrix.com/
```

هر دو ابزار باید **A grade** برای Browser Caching بدهند.

---

## 🎯 نتایج مورد انتظار

### قبل از بهینه‌سازی:
```
⚠️ Use efficient cache lifetimes
Est savings of 38 KiB

cdn.xpay.co/UUSD.webp     7d    11 KiB
cdn.xpay.co/TRX.webp      7d     5 KiB
cdn.xpay.co/BTC.webp      7d     4 KiB
cdn.xpay.co/USDT.webp     7d     4 KiB
```

### بعد از بهینه‌سازی:
```
✅ Efficient cache policy
All static assets use optimal cache lifetime (1 year)
```

**PageSpeed Score:**
- Mobile: بهبود +3 تا +5 امتیاز
- Desktop: بهبود +2 تا +4 امتیاز

---

## 🛠️ عیب‌یابی (Troubleshooting)

### مشکل: Cache هنوز 7 روز است

**راه‌حل 1:** `.htaccess` در مسیر صحیح قرار دارد؟
```bash
# باید دقیقاً در روت CDN باشد
/home/username/public_html/cdn.xpay.co/.htaccess
```

**راه‌حل 2:** Apache modules فعال هستند؟
```bash
# از طریق cPanel یا SSH بررسی کنید:
php -m | grep headers
php -m | grep expires
```

**راه‌حل 3:** Cloudflare Cache را پاک کنید
```
Cloudflare Dashboard → Caching → Purge Everything
```

### مشکل: CORS Errors در Console

**راه‌حل:** اضافه کردن CORS headers:
```apache
Header set Access-Control-Allow-Origin "*"
Header set Access-Control-Allow-Methods "GET, OPTIONS"
```

### مشکل: Gzip کار نمی‌کند

**راه‌حل:** بررسی فعال بودن `mod_deflate`:
```bash
# در cPanel → Software → Select PHP Version
# یا از طریق .htaccess فعال کنید (قبلاً در فایل وجود دارد)
```

---

## 📚 منابع مرتبط

- [مستندات Apache mod_headers](https://httpd.apache.org/docs/current/mod/mod_headers.html)
- [راهنمای Cache-Control در MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
- [بهینه‌سازی Cache در Google Developers](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)
- [Cloudflare Cache Documentation](https://developers.cloudflare.com/cache/)

---

## 📝 یادداشت‌های مهم

### ⚠️ نکات امنیتی:
- فقط برای فایل‌های **استاتیک** از cache طولانی استفاده کنید
- فایل‌های **HTML/PHP** نباید cache شوند
- برای فایل‌های حساس (API responses) از `Cache-Control: no-store` استفاده کنید

### 🔄 بروزرسانی فایل‌ها:
- همیشه از **versioning** استفاده کنید: `file.v2.webp`
- یا از **query strings** استفاده کنید: `file.webp?v=2`
- هرگز فایل را با همان نام جایگزین نکنید (1 سال cache دارد!)

### 📊 مانیتورینگ:
- هر ماه یکبار Cache headers را بررسی کنید
- PageSpeed را مرتباً چک کنید
- از Google Search Console برای بررسی Core Web Vitals استفاده کنید

---

## 🎉 نتیجه‌گیری

با پیاده‌سازی این تنظیمات:
- ✅ PageSpeed warning "Use efficient cache lifetimes" برطرف می‌شود
- ✅ سرعت بارگذاری صفحات برای بازدیدکنندگان تکراری **بسیار** بهبود می‌یابد
- ✅ مصرف bandwidth سرور و CDN کاهش می‌یابد
- ✅ امتیاز PageSpeed Insights بهبود می‌یابد (+3 تا +5 امتیاز)

---

**📅 آخرین بروزرسانی:** 8 دسامبر 2025  
**✍️ نویسنده:** XPay Development Team  
**📧 پشتیبانی:** برای سوالات به تیم توسعه مراجعه کنید
