# 🔒 راهنمای Cross-Origin Security Policies

## 📋 فهرست مطالب
- [COOP/COEP/CORP چیست؟](#coopcoep-corp-چیست)
- [چرا مهم است؟](#چرا-مهم-است)
- [پیکربندی محیط Local/Docker](#پیکربندی-محیط-localdocker)
- [پیکربندی روی سرور Production](#-پیکربندی-روی-سرور-production)
- [توضیحات هر Policy](#توضیحات-هر-policy)
- [تست و بررسی](#تست-و-بررسی)
- [Troubleshooting](#troubleshooting)

---

## 🔐 COOP/COEP/CORP چیست؟

این سه header امنیتی برای **جداسازی و محافظت از Origin** طراحی شده‌اند:

### 1. COOP (Cross-Origin-Opener-Policy)
**جداسازی پنجره اصلی از اسناد cross-origin**

```
COOP → محافظت از صفحه اصلی در برابر pop-ups مخرب
```

### 2. COEP (Cross-Origin-Embedder-Policy)  
**کنترل بارگذاری منابع cross-origin**

```
COEP → اطمینان از دریافت صریح مجوز برای منابع external
```

### 3. CORP (Cross-Origin-Resource-Policy)
**کنترل این‌که چه origin هایی می‌توانند این منبع را لود کنند**

```
CORP → تعیین می‌کند کدام سایت‌ها می‌توانند از منابع شما استفاده کنند
```

---

## ⚠️ چرا مهم است؟

### بدون COOP:
```javascript
// صفحه مخرب
const win = window.open('https://your-site.com');
// ⚠️ می‌تواند به your-site دسترسی داشته باشد!
win.location = 'https://malicious-site.com/phishing';
```

### با COOP:
```javascript
// صفحه مخرب
const win = window.open('https://your-site.com');
// ✅ دسترسی مسدود شده!
// win.location = null
```

### تأثیر در PageSpeed Insights:

**قبل:**
```
❌ Ensure proper origin isolation with COOP
   Severity: High
   No COOP header found
```

**بعد:**
```
✅ COOP is properly configured
   (این warning دیگر نمایش داده نمی‌شود)
```

---

## 🛠️ پیکربندی محیط Local/Docker

> **⚠️ توجه:** این پیکربندی برای محیط **Local/Docker** است.  
> برای **Production Server**، به بخش [پیکربندی روی سرور](#-پیکربندی-روی-سرور-production) مراجعه کنید.

### تغییرات در nginx/default.conf:

```nginx
# nginx/default.conf (خط 47-64)
server {
    listen 443 ssl;
    server_name localhost;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # CSP
    add_header Content-Security-Policy "..." always;

    # 🔒 Cross-Origin-Opener-Policy (COOP)
    # Isolates browsing context from cross-origin documents
    add_header Cross-Origin-Opener-Policy "same-origin-allow-popups" always;

    # 🔒 Cross-Origin-Embedder-Policy (COEP)
    # Prevents loading of cross-origin resources without explicit permission
    add_header Cross-Origin-Embedder-Policy "unsafe-none" always;

    # 🔒 Cross-Origin-Resource-Policy (CORP)
    # Controls which origins can load this resource
    add_header Cross-Origin-Resource-Policy "cross-origin" always;

    # ... بقیه تنظیمات
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

### 📌 قبل از شروع: بررسی سازگاری

- [ ] ✅ آیا از pop-ups استفاده می‌کنید؟
- [ ] ✅ آیا از OAuth window.open استفاده می‌کنید؟
- [ ] ✅ آیا از iframes cross-origin استفاده می‌کنید؟
- [ ] ✅ آیا منابع external (CDN, fonts, images) دارید؟

⚠️ **مهم**: تنظیمات نادرست می‌تواند pop-ups یا OAuth را خراب کند!

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

    # CSP
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;" always;

    # 🔒 Cross-Origin Policies
    # COOP: مرحله 1 - تست (اجازه pop-ups)
    add_header Cross-Origin-Opener-Policy "same-origin-allow-popups" always;
    
    # COOP: مرحله 2 - نهایی (بعد از تست، اگر pop-up ندارید)
    # add_header Cross-Origin-Opener-Policy "same-origin" always;
    
    # COEP: اجازه منابع بدون CORS
    add_header Cross-Origin-Embedder-Policy "unsafe-none" always;
    
    # CORP: اجازه استفاده cross-origin
    add_header Cross-Origin-Resource-Policy "cross-origin" always;

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

    # HSTS
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"

    # CSP
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;"

    # 🔒 Cross-Origin Policies
    Header always set Cross-Origin-Opener-Policy "same-origin-allow-popups"
    Header always set Cross-Origin-Embedder-Policy "unsafe-none"
    Header always set Cross-Origin-Resource-Policy "cross-origin"
</VirtualHost>
```

**روش 2: در .htaccess**
```apache
<IfModule mod_headers.c>
    <If "%{HTTPS} == 'on'">
        # HSTS
        Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"

        # CSP
        Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;"

        # 🔒 Cross-Origin Policies
        Header always set Cross-Origin-Opener-Policy "same-origin-allow-popups"
        Header always set Cross-Origin-Embedder-Policy "unsafe-none"
        Header always set Cross-Origin-Resource-Policy "cross-origin"
    </If>
</IfModule>
```

**اعمال:**
```bash
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
        # HSTS
        Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"

        # CSP
        Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;"
        
        # Cross-Origin Policies
        Header always set Cross-Origin-Opener-Policy "same-origin-allow-popups"
        Header always set Cross-Origin-Embedder-Policy "unsafe-none"
        Header always set Cross-Origin-Resource-Policy "cross-origin"
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
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;"
    Header always set Cross-Origin-Opener-Policy "same-origin-allow-popups"
    Header always set Cross-Origin-Embedder-Policy "unsafe-none"
    Header always set Cross-Origin-Resource-Policy "cross-origin"
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
4. در **Additional directives for HTTPS**:

```apache
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.google.com https://www.gstatic.com https://cdn.jsdelivr.net https://unpkg.com https://code.highcharts.com https://www.googletagmanager.com https://www.google-analytics.com https://static.cloudflareinsights.com https://challenges.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.jsdelivr.net; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https: http:; connect-src 'self' https://www.google-analytics.com https://region1.google-analytics.com https://cloudflareinsights.com https://xpay.co; frame-src 'self' https://www.google.com https://challenges.cloudflare.com; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests;"
Header always set Cross-Origin-Opener-Policy "same-origin-allow-popups"
Header always set Cross-Origin-Embedder-Policy "unsafe-none"
Header always set Cross-Origin-Resource-Policy "cross-origin"
```

5. **OK** → Apply

---

#### 5️⃣ Cloudflare

1. وارد **Cloudflare Dashboard** شوید
2. **Security** → **Settings** → **HTTP Headers**
3. **Add Headers**:

```
Cross-Origin-Opener-Policy: same-origin-allow-popups
Cross-Origin-Embedder-Policy: unsafe-none
Cross-Origin-Resource-Policy: cross-origin
```

یا از **Transform Rules**:
1. **Rules** → **Transform Rules** → **Modify Response Header**
2. **Create rule** برای هر header

---

## 📊 توضیحات هر Policy

### 1️⃣ COOP (Cross-Origin-Opener-Policy)

```nginx
Cross-Origin-Opener-Policy: same-origin-allow-popups
```

#### مقادیر ممکن:

| مقدار | توضیح | استفاده |
|-------|-------|---------|
| **`unsafe-none`** | بدون محدودیت (پیش‌فرض) | ⚠️ ناامن |
| **`same-origin-allow-popups`** | اجازه pop-ups از same-origin | ✅ توصیه شده |
| **`same-origin`** | فقط same-origin، بدون pop-up | 🔒 امن‌ترین |

#### انتخاب صحیح:

**استفاده از `same-origin-allow-popups` اگر:**
- ✅ دارید: OAuth با window.open (Google, Facebook login)
- ✅ دارید: Payment gateways با pop-up
- ✅ دارید: Social media share با pop-up

**استفاده از `same-origin` اگر:**
- ✅ هیچ pop-up ندارید
- ✅ همه فرم‌ها در همان صفحه هستند
- ✅ OAuth با redirect کار می‌کنید

#### مثال:

```javascript
// با same-origin-allow-popups
const popup = window.open('https://your-site.com/oauth');
// ✅ کار می‌کند

// با same-origin
const popup = window.open('https://your-site.com/oauth');
// ✅ کار می‌کند

const popup2 = window.open('https://other-site.com');
// ❌ مسدود می‌شود
```

---

### 2️⃣ COEP (Cross-Origin-Embedder-Policy)

```nginx
Cross-Origin-Embedder-Policy: unsafe-none
```

#### مقادیر ممکن:

| مقدار | توضیح | استفاده |
|-------|-------|---------|
| **`unsafe-none`** | بدون محدودیت (پیش‌فرض) | ✅ توصیه شده برای شروع |
| **`require-corp`** | نیاز به CORP برای همه منابع | 🔒 امن‌تر اما پیچیده |

#### استفاده از `unsafe-none` اگر:
- ✅ از CDN های external استفاده می‌کنید
- ✅ Google Fonts، jsdelivr، unpkg دارید
- ✅ Images از sources مختلف دارید

#### استفاده از `require-corp` اگر:
- ✅ تمام منابع را کنترل می‌کنید
- ✅ همه منابع external دارای CORS header هستند
- ✅ می‌خواهید از SharedArrayBuffer استفاده کنید

---

### 3️⃣ CORP (Cross-Origin-Resource-Policy)

```nginx
Cross-Origin-Resource-Policy: cross-origin
```

#### مقادیر ممکن:

| مقدار | توضیح | استفاده |
|-------|-------|---------|
| **`same-site`** | فقط same-site | برای منابع داخلی |
| **`same-origin`** | فقط same-origin | برای منابع محرمانه |
| **`cross-origin`** | همه origin ها | ✅ برای CDN و منابع عمومی |

#### انتخاب صحیح:

**`cross-origin` برای:**
- ✅ Assets عمومی (CSS, JS, Images)
- ✅ CDN files
- ✅ Public API responses

**`same-origin` برای:**
- 🔒 User data
- 🔒 API tokens
- 🔒 محتوای محرمانه

---

## ✅ تست و بررسی

### 1. تست با cURL

```bash
# تست COOP header
curl -I https://your-domain.com | grep -i "cross-origin"

# خروجی مورد انتظار:
cross-origin-opener-policy: same-origin-allow-popups
cross-origin-embedder-policy: unsafe-none
cross-origin-resource-policy: cross-origin
```

### 2. تست در مرورگر

**Chrome DevTools:**
1. `F12` → **Network** tab
2. Reload صفحه
3. کلیک روی اولین request
4. **Response Headers** → بررسی Cross-Origin headers

**Console Tab:**
```javascript
// تست COOP در Console
console.log(window.opener);
// اگر COOP فعال باشد: null یا محدود شده
```

### 3. تست Pop-ups

```javascript
// تست pop-up بعد از اعمال COOP
const popup = window.open('https://your-domain.com/test');
console.log(popup); // باید باز شود

const popup2 = window.open('https://google.com');
console.log(popup2); // باید محدود باشد
```

### 4. تست با Security Headers

```
https://securityheaders.com/?q=https://your-domain.com
```

**امتیاز مورد انتظار:**
- 🅰️ **Grade A** یا **A+**
- ✅ COOP header detected
- ✅ COEP header detected
- ✅ CORP header detected

### 5. تست PageSpeed Insights

```
https://pagespeed.web.dev/analysis?url=https://your-domain.com
```

**قبل:**
```
❌ Ensure proper origin isolation with COOP
   No COOP header found (High)
```

**بعد:**
```
✅ COOP is properly configured
```

---

## 🚨 Troubleshooting

### مشکل 1: Pop-ups باز نمی‌شوند

**Console Error:**
```
Blocked opening 'https://...' in a new window because the request was made 
in a sandboxed frame whose 'allow-popups' permission is not set.
```

**راه‌حل:**
```nginx
# از same-origin-allow-popups استفاده کنید
Cross-Origin-Opener-Policy: same-origin-allow-popups

# نه same-origin
```

### مشکل 2: OAuth نمی‌کند

**علت**: COOP با `same-origin` OAuth window.open را مسدود می‌کند

**راه‌حل:**
```nginx
# تغییر به same-origin-allow-popups
Cross-Origin-Opener-Policy: same-origin-allow-popups

# یا غیرفعال کردن موقت
# Cross-Origin-Opener-Policy: unsafe-none
```

### مشکل 3: Images/Fonts لود نمی‌شوند

**Console Error:**
```
net::ERR_BLOCKED_BY_RESPONSE.NotSameOriginAfterDefaultedToSameOriginByCoep
```

**راه‌حل:**
```nginx
# استفاده از unsafe-none برای COEP
Cross-Origin-Embedder-Policy: unsafe-none

# یا اضافه کردن CORS به CDN/images
```

### مشکل 4: Iframe خارجی کار نمی‌کند

**راه‌حل:**
```nginx
# CORP را cross-origin نگه دارید
Cross-Origin-Resource-Policy: cross-origin

# و CSP را چک کنید
frame-src 'self' https://trusted-domain.com;
```

---

## 📋 استراتژی استقرار تدریجی

### مرحله 1️⃣: تست اولیه (هفته 1)

```nginx
# تست با کمترین محدودیت
Cross-Origin-Opener-Policy: same-origin-allow-popups
Cross-Origin-Embedder-Policy: unsafe-none
Cross-Origin-Resource-Policy: cross-origin
```

**بررسی:**
- ✅ Pop-ups کار می‌کنند؟
- ✅ OAuth/Login کار می‌کنند؟
- ✅ Payment gateways کار می‌کنند?
- ✅ هیچ error در Console نیست؟

### مرحله 2️⃣: سخت‌تر کردن (بعد از تست موفق)

اگر هیچ pop-up ندارید:

```nginx
# تغییر به same-origin
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: unsafe-none
Cross-Origin-Resource-Policy: cross-origin
```

### مرحله 3️⃣: Maximum Security (اختیاری)

اگر تمام منابع را کنترل می‌کنید:

```nginx
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Resource-Policy: same-origin
```

⚠️ **نیاز به:** تمام CDN ها و external resources باید CORS headers داشته باشند!

---

## 📋 خلاصه برای تیم

### برای DevOps:

**مرحله 1: اضافه کردن Headers**
```nginx
# Nginx
add_header Cross-Origin-Opener-Policy "same-origin-allow-popups" always;
add_header Cross-Origin-Embedder-Policy "unsafe-none" always;
add_header Cross-Origin-Resource-Policy "cross-origin" always;
```

**مرحله 2: تست**
```bash
curl -I https://your-domain.com | grep -i "cross-origin"
```

**مرحله 3: Monitor**
- بررسی Console errors
- تست pop-ups و OAuth
- بررسی PageSpeed Insights

### برای SEO:

1. ✅ تست popup sharing (Twitter, Facebook, LinkedIn)
2. ✅ تست login با OAuth providers
3. ✅ بررسی payment gateways
4. ✅ تست Google Analytics events
5. ✅ بررسی PageSpeed score

### فایل‌های مهم:
- **Local**: `nginx/default.conf` (خط 54-64)
- **Production Nginx**: `/etc/nginx/sites-available/your-domain.conf`
- **Production Apache**: `/etc/apache2/sites-available/your-domain.conf` یا `.htaccess`
- **Docs**: `docs/CROSS-ORIGIN-POLICIES.md` (این فایل)

---

## 📚 منابع بیشتر

### Documentation:
- [MDN: Cross-Origin-Opener-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Opener-Policy)
- [MDN: Cross-Origin-Embedder-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy)
- [MDN: Cross-Origin-Resource-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Resource-Policy)
- [WHATWG: COOP and COEP](https://html.spec.whatwg.org/multipage/origin.html#cross-origin-opener-policies)

### Test Tools:
- [Security Headers](https://securityheaders.com/)
- [COOP/COEP Test](https://web.dev/coop-coep/)

### Related Docs:
- [HSTS Security](HSTS-SECURITY.md)
- [CSP Security](CSP_SECURITY.md)
- [HTTP Headers Optimization](HTTP-HEADERS-OPTIMIZATION.md)

---

## 📝 خلاصه

✅ **Headers اضافه شده:**
```nginx
Cross-Origin-Opener-Policy: same-origin-allow-popups
Cross-Origin-Embedder-Policy: unsafe-none
Cross-Origin-Resource-Policy: cross-origin
```

✅ **مزایا:**
- 🛡️ محافظت از Spectre attacks
- 🚫 جلوگیری از cross-origin info leaks
- ⚡ بهبود امتیاز امنیتی
- ✅ رفع warning در PageSpeed Insights

✅ **تست:**
```bash
curl -I https://your-domain.com | grep -i "cross-origin"
```

⚠️ **نکات مهم:**
- با `same-origin-allow-popups` شروع کنید
- Pop-ups و OAuth را تست کنید
- Console errors را بررسی کنید
- بعد از تست می‌توانید سخت‌تر کنید

---

**آخرین بروزرسانی:** 23 دسامبر 2025  
**وضعیت:** ✅ فعال و تست شده  
**PageSpeed Status:** ✅ رفع شده
