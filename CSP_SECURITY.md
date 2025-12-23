# 🛡️ Content Security Policy (CSP) Implementation

## 📋 فهرست مطالب
- [CSP چیست؟](#csp-چیست)
- [پیکربندی محیط Local/Docker](#پیکربندی-محیط-localdocker)
- [پیکربندی روی سرور Production](#-پیکربندی-روی-سرور-production)
- [توضیحات Directives](#توضیحات-directives)
- [تست و بررسی](#تست-و-بررسی)
- [Troubleshooting](#troubleshooting)

---

## 🔐 CSP چیست؟

**Content Security Policy (CSP)** یک لایه امنیتی است که از حملات زیر جلوگیری می‌کند:
- ✅ **XSS (Cross-Site Scripting)** - حملات تزریق اسکریپت
- ✅ **Data Injection** - تزریق داده‌های مخرب
- ✅ **Clickjacking** - حملات کلیک‌جکینگ

### چرا HTTP Header؟
```
❌ Bad:  <meta http-equiv="Content-Security-Policy" content="...">
✅ Good: HTTP Response Header
```

**دلایل استفاده از HTTP Header:**
- 🔒 **امن‌تر**: قبل از پردازش HTML اعمال می‌شود
- ⚡ **سریع‌تر**: بدون نیاز به parse کردن HTML
- 💪 **قدرتمندتر**: تمام directive های CSP قابل استفاده
- ✅ **توصیه PageSpeed**: Google از header استفاده را توصیه می‌کند

---

## 🛠️ پیکربندی محیط Local/Docker

> **⚠️ توجه:** این پیکربندی برای محیط **Local/Docker** است.  
> برای **Production Server**، به بخش [پیکربندی روی سرور](#-پیکربندی-روی-سرور-production) مراجعه کنید.

### تغییرات انجام شده:

#### 1. حذف Meta Tag از header.php ❌

**قبل:**
```php
<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">
```

**بعد:**
```php
<!-- Meta tag CSP حذف شد - از HTTP Header استفاده می‌شود -->
```

#### 2. اضافه شدن CSP Header در nginx/default.conf ✅

```nginx
# nginx/default.conf (خط 41-49)
server {
    listen 443 ssl;
    server_name localhost;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # 🛡️ Content Security Policy (CSP)
    # Prevents XSS attacks and code injection
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;" always;
}
```

### Restart Docker:
```bash
cd C:\Docker\xpay
docker-compose restart nginx
```

---

## 🚀 پیکربندی روی سرور Production

> **مخاطبان:** تیم DevOps و SEO  
> **محیط:** Production Server (Apache/Nginx/cPanel/Plesk)

### 📌 قبل از شروع

- [ ] ✅ SSL Certificate معتبر نصب است
- [ ] ✅ تمام external scripts شناسایی شده‌اند
- [ ] ✅ Analytics و tracking codes لیست شده‌اند
- [ ] ✅ CDN domains مشخص شده‌اند

---

### 🔧 پیکربندی بر اساس Web Server

#### 1️⃣ Nginx (توصیه شده)

**فایل:** `/etc/nginx/sites-available/your-domain.conf`

```nginx
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # SSL configs...
    ssl_certificate /path/to/ssl/fullchain.pem;
    ssl_certificate_key /path/to/ssl/privkey.pem;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # 🛡️ Content Security Policy
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;" always;

    # بقیه تنظیمات...
}
```

**اعمال تغییرات:**
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

#### 2️⃣ Apache

**فایل:** `/etc/apache2/sites-available/your-domain.conf` یا `.htaccess`

**روش 1: در VirtualHost**
```apache
<VirtualHost *:443>
    ServerName example.com
    
    SSLEngine on
    # SSL configs...

    # 🛡️ Content Security Policy
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;"
</VirtualHost>
```

**روش 2: در .htaccess**
```apache
<IfModule mod_headers.c>
    # فقط برای HTTPS
    <If "%{HTTPS} == 'on'">
        # 🛡️ Content Security Policy
        Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;"
    </If>
</IfModule>
```

**اعمال:**
```bash
# فعال‌سازی mod_headers
sudo a2enmod headers
sudo apache2ctl configtest
sudo systemctl reload apache2
```

---

#### 3️⃣ cPanel / WHM

**روش 1: از طریق .htaccess**

1. وارد **cPanel** → **File Manager** شوید
2. فایل `.htaccess` در root directory را باز کنید
3. کد زیر را اضافه کنید:

```apache
<IfModule mod_headers.c>
    <If "%{HTTPS} == 'on'">
        Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;"
    </If>
</IfModule>
```

**روش 2: از طریق WHM (دسترسی root)**

1. وارد **WHM** شوید
2. **Service Configuration** → **Apache Configuration** → **Include Editor**
3. **Pre VirtualHost Include** → **All Versions**
4. کد:

```apache
<IfModule mod_headers.c>
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;"
</IfModule>
```

5. Save و Rebuild
6. Restart:
```bash
/scripts/restartsrv_httpd
```

---

#### 4️⃣ Plesk

1. وارد **Plesk Panel** شوید
2. **Domains** → انتخاب domain
3. **Apache & nginx Settings**
4. در **Additional directives for HTTP** و **Additional directives for HTTPS**:

```apache
# فقط در بخش HTTPS
Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;"
```

5. **OK** → Apply

---

#### 5️⃣ Cloudflare (اگر استفاده می‌کنید)

1. وارد **Cloudflare Dashboard** شوید
2. **Security** → **Settings**
3. **HTTP Headers**
4. **Add Header**:
   - Name: `Content-Security-Policy`
   - Value: (کد CSP کامل از بالا)

یا از **Transform Rules**:
1. **Rules** → **Transform Rules** → **Modify Response Header**
2. **Create rule**
3. Set header: `Content-Security-Policy`
4. Value: (کد CSP کامل)

---

## 📊 توضیحات Directives

### CSP کامل اعمال شده:

## 📊 توضیحات Directives

### CSP کامل اعمال شده:

```
Content-Security-Policy: 
  default-src 'self'; 
  script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; 
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; 
  font-src 'self' https://fonts.gstatic.com data:; 
  img-src 'self' data: https: http:; 
  connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; 
  frame-src 'self' https://www.google.com https://challenges.cloudflare.com; 
  object-src 'none'; 
  base-uri 'self'; 
  form-action 'self'; 
  upgrade-insecure-requests;
```

---

### 1️⃣ `default-src 'self'`
**پیش‌فرض برای همه منابع**

```nginx
default-src 'self'
```

- 🔒 فقط از خود دامنه (same-origin) مجاز است
- ✅ امن‌ترین حالت پیش‌فرض
- 📝 سایر directive ها این را override می‌کنند

---

### 2️⃣ `script-src` ⚠️ **HIGH Priority**
**کنترل اجرای JavaScript**

```nginx
script-src 'self' 'unsafe-inline' 'unsafe-eval' 
  https://www.google.com 
  https://www.gstatic.com 
  https://cdn.jsdelivr.net 
  https://unpkg.com 
  https://code.highcharts.com 
  https://www.googletagmanager.com 
  https://www.google-analytics.com 
  https://static.cloudflareinsights.com 
  https://challenges.cloudflare.com
```

**توضیحات:**
- `'self'` - اسکریپت‌های خود سایت
- `'unsafe-inline'` - inline scripts (مورد نیاز برای Google Tag Manager)
- `'unsafe-eval'` - eval() و Function() (مورد نیاز برای analytics)

**Domains اضافه شده:**
- **Google**: reCAPTCHA و Analytics
- **jsdelivr/unpkg**: CDN libraries
- **Highcharts**: نمودارها
- **Cloudflare**: Challenge pages و analytics

⚠️ **نکته امنیتی**: `'unsafe-inline'` و `'unsafe-eval'` خطرناک هستند! در نسخه بعدی از **nonces** استفاده کنید.

---

### 3️⃣ `object-src 'none'` ✅ **HIGH Priority**
**مسدود کردن Plugins**

```nginx
object-src 'none'
```

- 🚫 مسدود کردن `<object>`, `<embed>`, `<applet>`
- ✅ **حیاتی برای امنیت** - جلوگیری از Flash/Java injection
- 🛡️ یکی از مهم‌ترین directive ها

**این directive مشکل PageSpeed را برطرف می‌کند!**

---

### 4️⃣ `style-src`
**کنترل CSS**

```nginx
style-src 'self' 'unsafe-inline' 
  https://fonts.googleapis.com 
  https://cdn.jsdelivr.net
```

- `'self'` - CSS های خود سایت
- `'unsafe-inline'` - inline styles (مورد نیاز برای dynamic styles)
- **Google Fonts** - برای فونت‌های فارسی
- **jsdelivr** - CDN styles

---

### 5️⃣ `font-src`
**کنترل فونت‌ها**

```nginx
font-src 'self' https://fonts.gstatic.com data:
```

- `'self'` - فونت‌های لوکال
- **fonts.gstatic.com** - Google Fonts
- `data:` - فونت‌های base64 encoded

---

### 6️⃣ `img-src`
**کنترل تصاویر**

```nginx
img-src 'self' data: https: http:
```

- `'self'` - تصاویر خود سایت
- `data:` - base64 images
- `https:` - همه تصاویر HTTPS (برای user-generated content)
- `http:` - backward compatibility (می‌توان حذف کرد)

---

### 7️⃣ `connect-src`
**کنترل AJAX و WebSocket**

```nginx
connect-src 'self' 
  https://www.google-analytics.com 
  https://region1.google-analytics.com 
  https://cloudflareinsights.com 
  https://xpay.co
```

**استفاده:**
- API calls
- Analytics tracking
- WebSocket connections
- fetch() و XMLHttpRequest

**Domains:**
- **Google Analytics** - tracking
- **Cloudflare** - insights
- **xpay.co** - API calls برای نرخ‌ها

---

### 8️⃣ `frame-src`
**کنترل iframes**

```nginx
frame-src 'self' 
  https://www.google.com 
  https://challenges.cloudflare.com
```

- `'self'` - iframes از خود سایت
- **Google** - reCAPTCHA
- **Cloudflare** - challenge pages

---

### 9️⃣ `base-uri 'self'`
**محدود کردن `<base>` tag**

```nginx
base-uri 'self'
```

- 🔒 جلوگیری از حمله base tag hijacking
- ✅ فقط از خود دامنه

---

### 🔟 `form-action 'self'`
**محدود کردن مقصد فرم‌ها**

```nginx
form-action 'self'
```

- 🔒 فرم‌ها فقط به خود دامنه ارسال می‌شوند
- ✅ جلوگیری از phishing attacks

---

### 1️⃣1️⃣ `upgrade-insecure-requests`
**ارتقا خودکار HTTP به HTTPS**

```nginx
upgrade-insecure-requests
```

- ⚡ همه درخواست‌های HTTP به HTTPS تبدیل می‌شوند
- ✅ جایگزین meta tag قبلی
- 🔒 هماهنگ با HSTS

---

## ✅ تست و بررسی

### 1. تست با cURL

```bash
# تست CSP header
curl -I https://your-domain.com | grep -i "content-security-policy"

# خروجی مورد انتظار:
content-security-policy: default-src 'self'; script-src 'self' 'unsafe-inline'...
```

### 2. تست در مرورگر

**Chrome DevTools:**
1. `F12` → **Network** tab
2. Reload صفحه
3. کلیک روی اولین request
4. **Response Headers** → بررسی `content-security-policy`

**Console Tab:**
- بررسی CSP Violations (اگر هست)
- باید هیچ error قرمزی نباشد

### 3. تست با Security Headers

```
https://securityheaders.com/?q=https://your-domain.com
```

**امتیاز مورد انتظار:**
- 🅰️ **Grade A** یا **A+**
- ✅ CSP header detected
- ✅ CSP includes script-src
- ✅ CSP includes object-src

### 4. تست PageSpeed Insights

```
https://pagespeed.web.dev/analysis?url=https://your-domain.com
```

**قبل از CSP:**
```
❌ Ensure CSP is effective against XSS attacks
   - script-src directive is missing (High)
   - object-src missing (High)
   - CSP in meta tag (Medium)
```

**بعد از CSP:**
```
✅ CSP is properly configured
   (این warning دیگر نمایش داده نمی‌شود)
```

### 5. تست CSP Evaluator

```
https://csp-evaluator.withgoogle.com/
```

1. CSP خود را paste کنید
2. **Evaluate** کلیک کنید
3. بررسی warnings و suggestions

---

## 🚨 Troubleshooting

### مشکل 1: Scripts لود نمی‌شوند

**Console Error:**
```
Refused to load the script 'https://example.com/script.js' 
because it violates the following Content Security Policy directive: "script-src..."
```

**راه‌حل:**
```nginx
# Domain را به script-src اضافه کنید
script-src 'self' ... https://example.com;
```

### مشکل 2: Inline Styles کار نمی‌کنند

**Console Error:**
```
Refused to apply inline style because it violates CSP directive: "style-src..."
```

**راه‌حل:**
```nginx
# اگر unsafe-inline ندارید، اضافه کنید
style-src 'self' 'unsafe-inline';

# یا بهتر: از external stylesheet استفاده کنید
```

### مشکل 3: Google Analytics کار نمی‌کند

**Console Error:**
```
Refused to connect to 'https://www.google-analytics.com' 
because it violates CSP directive: "connect-src..."
```

**راه‌حل:**
```nginx
# Google Analytics را اضافه کنید
script-src 'self' ... https://www.googletagmanager.com https://www.google-analytics.com;
connect-src 'self' ... https://www.google-analytics.com https://region1.google-analytics.com;
```

### مشکل 4: reCAPTCHA نمایش داده نمی‌شود

**راه‌حل:**
```nginx
script-src 'self' ... https://www.google.com https://www.gstatic.com;
frame-src 'self' ... https://www.google.com;
```

### مشکل 5: Images از CDN لود نمی‌شوند

**راه‌حل:**
```nginx
# برای همه CDN ها
img-src 'self' data: https: http:;

# یا فقط CDN خاص
img-src 'self' data: https://cdn.example.com;
```

---

## 🔧 سفارشی‌سازی برای سایت شما

### اضافه کردن Domain جدید

**مثال: اضافه کردن Tawk.to (Live Chat)**

```nginx
# قبل
script-src 'self' 'unsafe-inline' ...;

# بعد
script-src 'self' 'unsafe-inline' ... https://embed.tawk.to;
connect-src 'self' ... https://va.tawk.to;
```

### اضافه کردن CDN جدید

```nginx
script-src 'self' ... https://your-cdn.com;
style-src 'self' ... https://your-cdn.com;
font-src 'self' ... https://your-cdn.com;
img-src 'self' ... https://your-cdn.com;
```

### حذف `'unsafe-inline'` (پیشرفته)

برای امنیت بیشتر، استفاده از **nonces**:

```nginx
script-src 'self' 'nonce-{RANDOM}' https://...;
```

در PHP:
```php
<?php
$nonce = base64_encode(random_bytes(16));
header("Content-Security-Policy: script-src 'self' 'nonce-$nonce';");
?>

<script nonce="<?php echo $nonce; ?>">
  // inline script
</script>
```

---

## 📋 خلاصه برای تیم

### برای DevOps:
1. ✅ حذف meta tag از header.php
2. ✅ اضافه کردن CSP header در nginx/apache
3. ✅ شامل: `script-src`, `object-src 'none'`, `upgrade-insecure-requests`
4. ✅ Restart web server
5. ✅ تست با curl و securityheaders.com

### برای SEO:
1. ✅ بررسی Console در Chrome DevTools
2. ✅ تست PageSpeed Insights
3. ✅ بررسی Analytics و Tracking codes کار می‌کنند
4. ✅ تست reCAPTCHA در contact forms
5. ✅ بررسی error logs

### فایل‌های مهم:
- **Local**: `nginx/default.conf` (خط 47)
- **Production Nginx**: `/etc/nginx/sites-available/your-domain.conf`
- **Production Apache**: `/etc/apache2/sites-available/your-domain.conf` یا `.htaccess`
- **Docs**: `docs/CSP_SECURITY.md` (این فایل)

---

## 📚 منابع بیشتر

### Documentation:
- [MDN: Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [Google: CSP Guide](https://developers.google.com/web/fundamentals/security/csp)
- [OWASP: CSP Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)

### Test Tools:
- [Security Headers](https://securityheaders.com/)
- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [CSP Scanner](https://cspscanner.com/)

### Related Docs:
- [HSTS Security](HSTS-SECURITY.md)
- [HTTP Headers Optimization](HTTP-HEADERS-OPTIMIZATION.md)
- [PageSpeed Optimizations](PAGESPEED-OPTIMIZATIONS.md)

---

## 📝 خلاصه

✅ **تغییرات انجام شده:**
- ❌ Meta tag CSP حذف شد از header.php
- ✅ CSP header کامل در nginx اضافه شد
- ✅ شامل `script-src` (High priority)
- ✅ شامل `object-src 'none'` (High priority)  
- ✅ شامل `upgrade-insecure-requests`

✅ **مزایا:**
- 🛡️ محافظت از XSS attacks
- 🚫 جلوگیری از code injection
- ⚡ بهبود امتیاز امنیتی
- ✅ رفع warning در PageSpeed Insights

✅ **تست:**
```bash
curl -I https://your-domain.com | grep -i "content-security-policy"
```

⚠️ **نکات مهم:**
- همیشه در محیط staging ابتدا تست کنید
- Console errors را بررسی کنید
- Analytics و tracking codes را تست کنید
- برای production از استراتژی تدریجی استفاده کنید

---

**آخرین بروزرسانی:** 23 دسامبر 2025  
**وضعیت:** ✅ فعال و تست شده  
**PageSpeed Status:** ✅ رفع شده
```
script-src 'self' 'unsafe-inline' 'unsafe-eval' 
  https://www.googletagmanager.com 
  https://van.najva.com 
  https://cdn.goftino.com
  https://s1.mediaad.org
```

**Why `'unsafe-inline'` and `'unsafe-eval'`?**
- Google Tag Manager requires `'unsafe-eval'`
- Some analytics tools use inline scripts
- Future improvement: Use nonces for inline scripts

**Added Domains:**
- `s1.mediaad.org` - Retargeting and analytics script

### 2. object-src
Blocks plugins like Flash, Java applets:
```
object-src 'none'
```
This is **critical** for security - blocks injection of malicious plugins.

### 3. style-src
Controls stylesheet sources:
```
style-src 'self' 'unsafe-inline' https://fonts.googleapis.com
```
Allows Google Fonts and inline styles.

### 4. img-src
Controls image sources:
```
img-src 'self' data: https: http:
```
Allows all images (data URIs, HTTPS, HTTP for compatibility).

### 5. font-src
Controls font sources:
```
font-src 'self' data: https://fonts.gstatic.com
```
Allows Google Fonts and data URIs.

### 6. connect-src
Controls AJAX, WebSocket, EventSource connections:
```
connect-src 'self' 
  https://www.google-analytics.com 
  https://van.najva.com 
  https://cdn.goftino.com
  https://api.xpay.co
```

**Added Domains:**
- `api.xpay.co` - XPay API for chart data and fiat conversions

### 7. frame-src
Controls iframe sources:
```
frame-src 'self' https://www.googletagmanager.com
```
Only allows Google Tag Manager iframes.

### 8. Other Security Directives

```
base-uri 'self'               # Prevents <base> tag hijacking
form-action 'self'            # Forms can only submit to same origin
frame-ancestors 'self'        # Prevents clickjacking
upgrade-insecure-requests     # Upgrades HTTP to HTTPS
```

## Testing CSP

### Browser Console
Open browser DevTools (F12) → Console tab. If CSP blocks something, you'll see:
```
Refused to load the script 'https://evil.com/script.js' because it violates 
the following Content Security Policy directive: "script-src 'self'..."
```

### Online Tools
- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [Mozilla Observatory](https://observatory.mozilla.org/)

### PageSpeed Insights
Test at: https://pagespeed.web.dev/
Should show ✅ for "Ensure CSP is effective against XSS attacks"

## Common Issues & Solutions

### Issue: "Script blocked by CSP"
**Solution**: Add the domain to `script-src`:
```php
"script-src 'self' 'unsafe-inline' https://new-domain.com",
```

### Issue: "Stylesheet blocked by CSP"
**Solution**: Add the domain to `style-src`:
```php
"style-src 'self' 'unsafe-inline' https://new-domain.com",
```

### Issue: "Font blocked by CSP"
**Solution**: Add the domain to `font-src`:
```php
"font-src 'self' data: https://new-domain.com",
```

## Future Improvements

### 1. Use Nonces for Inline Scripts
Instead of `'unsafe-inline'`, use nonces:
```php
$nonce = base64_encode(random_bytes(16));
header("Content-Security-Policy: script-src 'self' 'nonce-$nonce'");
```

Then in HTML:
```html
<script nonce="<?php echo $nonce; ?>">
  // Your inline script
</script>
```

### 2. Report-Only Mode
Test CSP without blocking:
```php
header("Content-Security-Policy-Report-Only: ...");
```

### 3. CSP Reporting
Log violations to server:
```php
"report-uri /csp-violation-report"
```

## File Structure
```
functions.php                     # CSP implementation
docs/CSP_SECURITY.md             # This documentation
docs/changelog/CHANGELOG-FA.md   # Persian changelog
docs/changelog/CHANGELOG-EN.md   # English changelog
```

## References
- [MDN CSP Guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [Google CSP Guide](https://developers.google.com/web/fundamentals/security/csp)
- [Content-Security-Policy.com](https://content-security-policy.com/)

## Related Files
- `functions.php` - CSP implementation
- `header.php` - Removed old meta tag CSP
- `docs/NAJVA_OPTIMIZATION.md` - Najva integration docs
- `docs/SOURCE_MAPS_README.md` - Source maps docs

---

**Last Updated**: December 2025  
**Version**: 2.1.0
