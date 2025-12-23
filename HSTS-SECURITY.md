# 🔒 راهنمای HSTS Security (HTTP Strict Transport Security)

## 📋 فهرست مطالب
- [HSTS چیست؟](#hsts-چیست)
- [چرا HSTS مهم است؟](#چرا-hsts-مهم-است)
- [پیکربندی HSTS](#پیکربندی-hsts)
- [پارامترهای HSTS](#پارامترهای-hsts)
- [تست و بررسی](#تست-و-بررسی)
- [بهترین روش‌های امنیتی](#بهترین-روش‌های-امنیتی)
- [Troubleshooting](#troubleshooting)

---

## 🔐 HSTS چیست؟

**HSTS** مخفف **HTTP Strict Transport Security** است - یک مکانیزم امنیتی که به مرورگر می‌گوید **همیشه** از اتصال HTTPS (امن) استفاده کند، حتی اگر کاربر سعی کند از HTTP استفاده کند.

### ویژگی‌های کلیدی:
- ✅ **جلوگیری از Downgrade Attacks**: حمله‌هایی که سعی می‌کنند اتصال را از HTTPS به HTTP تنزل دهند
- ✅ **محافظت از Man-in-the-Middle**: جلوگیری از حملات شنود اطلاعات
- ✅ **اجباری کردن HTTPS**: مرورگر به طور خودکار تمام درخواست‌ها را به HTTPS تغییر می‌دهد
- ✅ **بهبود امتیاز امنیتی**: افزایش Security Score در ابزارهایی مثل PageSpeed Insights

---

## ⚠️ چرا HSTS مهم است؟

### قبل از HSTS:
```
1. کاربر تایپ می‌کند: http://example.com
2. Server redirect می‌کند: 301 → https://example.com
3. ⚠️ خطر: در این لحظه‌ی کوتاه، حمله MITM ممکن است!
```

### بعد از HSTS:
```
1. کاربر تایپ می‌کند: http://example.com
2. مرورگر خودش تبدیل می‌کند: https://example.com
3. ✅ امن: هیچ درخواست HTTP ارسال نمی‌شود!
```

### تأثیر در PageSpeed Insights:
- **قبل**: ⚠️ Warning: "Use a strong HSTS policy"
- **بعد**: ✅ Pass: "HSTS is properly configured"
- **توجه**: این توصیه "Unscored" است (روی امتیاز تأثیر ندارد) اما برای امنیت **خیلی مهم** است

---

## 🛠️ پیکربندی HSTS

> **⚠️ توجه:** پیکربندی زیر برای محیط **Local/Docker** است.  
> برای **Production Server**، به بخش [پیکربندی روی سرور Production](#-پیکربندی-روی-سرور-production) مراجعه کنید.

### پیکربندی محیط Local/Docker

HSTS header در فایل **nginx/default.conf** پیکربندی شده است:

```nginx
# HSTS Server Block (خط 33-38)
server {
    listen 443 ssl;
    server_name localhost;
    
    # ... SSL configs ...
    
    # HSTS (HTTP Strict Transport Security)
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
}
```

### کد کامل:

```nginx
# nginx/default.conf

# Redirect HTTP to HTTPS (خط 15-19)
server {
    listen 80;
    server_name localhost;
    return 301 https://$host$request_uri;
}

# HTTPS Server با HSTS
server {
    listen 443 ssl;
    server_name localhost;

    # SSL Certificates
    ssl_certificate /etc/nginx/certs/localhost.crt;
    ssl_certificate_key /etc/nginx/certs/localhost.key;

    # SSL Protocols
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # 🔒 HSTS Header (اضافه شده)
    # Force HTTPS for 1 year (31536000 seconds)
    # includeSubDomains: Apply to all subdomains
    # preload: Allow inclusion in browser HSTS preload lists
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # ... rest of config ...
}
```

---

## 📊 پارامترهای HSTS

### 1. `max-age`
```nginx
max-age=31536000
```
- **مدت زمان**: 31536000 ثانیه = 1 سال
- **معنی**: مرورگر باید برای 1 سال همیشه از HTTPS استفاده کند
- **توصیه**: برای production از 1 سال استفاده کنید

### 2. `includeSubDomains`
```nginx
includeSubDomains
```
- **معنی**: HSTS به تمام subdomain‌ها نیز اعمال شود
- **مثال**: اگر `example.com` دارای HSTS باشد، `api.example.com` و `blog.example.com` هم HTTPS اجباری دارند
- **توجه**: فقط اگر همه subdomain‌ها SSL دارند فعال کنید

### 3. `preload`
```nginx
preload
```
- **معنی**: اجازه اضافه شدن به لیست پیش‌بارگذاری HSTS مرورگرها
- **مزیت**: حتی در اولین بازدید هم HTTPS اجباری است
- **نحوه ثبت**: باید در [hstspreload.org](https://hstspreload.org/) ثبت شود

### 4. `always`
```nginx
add_header ... always;
```
- **معنی**: Header حتی در پاسخ‌های error (404, 500, ...) نیز اضافه شود
- **اهمیت**: بدون `always`، در صفحات خطا HSTS اعمال نمی‌شود

---

## ✅ تست و بررسی

### 1. تست Header با cURL

```bash
# Windows PowerShell
curl -I https://your-domain.com

# یا با سوییچ verbose
curl -v https://your-domain.com
```

**خروجی مورد انتظار:**
```
HTTP/2 200
strict-transport-security: max-age=31536000; includeSubDomains; preload
```

### 2. تست در مرورگر

**Chrome DevTools:**
1. باز کنید: `F12` → `Network` تب
2. صفحه را reload کنید
3. روی اولین request کلیک کنید
4. در بخش `Response Headers` باید ببینید:
   ```
   strict-transport-security: max-age=31536000; includeSubDomains; preload
   ```

**Firefox DevTools:**
1. باز کنید: `F12` → `Network` تب
2. صفحه را reload کنید
3. روی request کلیک کنید → `Headers` تب
4. بررسی کنید: `Strict-Transport-Security` header

### 3. تست با ابزار آنلاین

**Security Headers:**
```
https://securityheaders.com/?q=https://your-domain.com&followRedirects=on
```

**SSL Labs:**
```
https://www.ssllabs.com/ssltest/analyze.html?d=your-domain.com
```

### 4. تست PageSpeed Insights

```
https://pagespeed.web.dev/analysis?url=https://your-domain.com
```

**قبل از HSTS:**
```
❌ Use a strong HSTS policy
   Severity: High
   No HSTS header found
```

**بعد از HSTS:**
```
✅ HSTS is properly configured
   (این warning دیگر نمایش داده نمی‌شود)
```

---

## 🛡️ بهترین روش‌های امنیتی

### 1. استقرار تدریجی (Gradual Rollout)

برای production جدید، از `max-age` کم شروع کنید:

```nginx
# مرحله 1: تست (1 هفته)
add_header Strict-Transport-Security "max-age=604800" always;

# مرحله 2: تأیید (1 ماه)
add_header Strict-Transport-Security "max-age=2592000" always;

# مرحله 3: نهایی (1 سال)
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
```

### 2. قبل از فعال‌سازی `includeSubDomains`

✅ اطمینان حاصل کنید:
- همه subdomain‌ها SSL certificate معتبر دارند
- هیچ سرویسی روی HTTP نیست
- API endpoints همه HTTPS هستند

⚠️ خطر: اگر حتی یک subdomain بدون SSL باشد، کاربران نمی‌توانند به آن دسترسی داشته باشند!

### 3. Preload List

برای اضافه شدن به HSTS Preload List:

1. **شرایط:**
   - `max-age` حداقل 1 سال (31536000)
   - `includeSubDomains` فعال باشد
   - `preload` directive داشته باشد
   - تمام subdomain‌ها HTTPS داشته باشند

2. **ثبت:**
   - https://hstspreload.org/
   - دامنه خود را وارد کنید
   - فرم را پر کنید

3. **نتیجه:**
   - حتی در اولین بازدید، مرورگر HTTPS اجباری دارد
   - بدون نیاز به یک بار دریافت header

⚠️ **توجه**: حذف از preload list خیلی زمان‌بر است (چند ماه)، پس قبل از ثبت مطمئن شوید!

---

## 🔧 Troubleshooting

### مشکل 1: Header نمایش داده نمی‌شود

**علل احتمالی:**
1. Nginx restart نشده:
   ```bash
   docker-compose restart nginx
   ```

2. کش مرورگر:
   - `Ctrl + Shift + Delete` → Clear cache
   - یا از Incognito/Private mode استفاده کنید

3. CDN/Proxy caching:
   - CDN را پاک کنید (Purge cache)
   - یا مستقیماً به IP سرور متصل شوید

### مشکل 2: Warning هنوز وجود دارد

**بررسی کنید:**
1. آیا از HTTPS استفاده می‌کنید؟ (نه HTTP)
2. آیا header در تمام صفحات ارسال می‌شود؟
3. آیا `always` directive را اضافه کرده‌اید؟

**تست:**
```bash
# تست صفحه اصلی
curl -I https://your-domain.com

# تست صفحه 404
curl -I https://your-domain.com/nonexistent-page

# تست API endpoint
curl -I https://your-domain.com/wp-json/
```

همه باید HSTS header داشته باشند!

### مشکل 3: Mixed Content Errors

بعد از فعال‌سازی HSTS، ممکن است این خطا را ببینید:

```
Mixed Content: The page at 'https://...' was loaded over HTTPS,
but requested an insecure resource 'http://...'. This request has been blocked.
```

**راه‌حل:**
1. تمام لینک‌های `http://` را به `https://` تغییر دهید
2. یا از protocol-relative URLs استفاده کنید:
   ```html
   <!-- قبل -->
   <img src="http://example.com/image.jpg">
   
   <!-- بعد -->
   <img src="//example.com/image.jpg">
   ```

3. در WordPress:
   ```bash
   # Update URLs in database
   wp search-replace 'http://your-domain.com' 'https://your-domain.com' --all-tables
   ```

---

## � پیکربندی روی سرور Production

> **مخاطبان:** تیم DevOps و SEO  
> **محیط:** Production Server (Apache/Nginx/cPanel/Plesk)

این بخش راهنمای پیکربندی HSTS روی سرور واقعی است.

---

### 📌 قبل از شروع: چک‌لیست امنیتی

قبل از فعال‌سازی HSTS در production، **حتماً** موارد زیر را بررسی کنید:

- [ ] ✅ SSL Certificate معتبر نصب است (نه self-signed)
- [ ] ✅ تمام صفحات سایت با HTTPS قابل دسترسی هستند
- [ ] ✅ هیچ Mixed Content Error وجود ندارد
- [ ] ✅ تمام subdomain‌های فعال SSL دارند (اگر `includeSubDomains` می‌خواهید)
- [ ] ✅ API endpoints همه HTTPS هستند
- [ ] ✅ Redirect از HTTP به HTTPS فعال است
- [ ] ✅ تست در مرورگرهای مختلف انجام شده

⚠️ **اخطار مهم**: اگر این موارد تأیید نشده باشند، فعال‌سازی HSTS می‌تواند سایت را غیرقابل دسترسی کند!

---

### 🔧 پیکربندی بر اساس Web Server

#### 1️⃣ Nginx (توصیه شده)

**فایل:** `/etc/nginx/sites-available/your-domain.conf`

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    
    # Redirect همه ترافیک HTTP به HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # SSL Certificate
    ssl_certificate /path/to/ssl/fullchain.pem;
    ssl_certificate_key /path/to/ssl/privkey.pem;

    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 🔒 HSTS Header
    # مرحله 1: تست (1 هفته - توصیه برای شروع)
    add_header Strict-Transport-Security "max-age=604800" always;
    
    # مرحله 2: پس از تست موفق (1 ماه)
    # add_header Strict-Transport-Security "max-age=2592000; includeSubDomains" always;
    
    # مرحله 3: نهایی (1 سال)
    # add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # بقیه تنظیمات...
}
```

**دستورات اعمال تغییرات:**
```bash
# بررسی syntax
sudo nginx -t

# اعمال تغییرات
sudo systemctl reload nginx

# یا restart کامل
sudo systemctl restart nginx
```

---

#### 2️⃣ Apache

**فایل:** `/etc/apache2/sites-available/your-domain.conf` یا `.htaccess`

**روش 1: در VirtualHost Configuration**
```apache
<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com

    SSLEngine on
    SSLCertificateFile /path/to/ssl/cert.pem
    SSLCertificateKeyFile /path/to/ssl/privkey.pem
    SSLCertificateChainFile /path/to/ssl/chain.pem

    # 🔒 HSTS Header
    # مرحله 1: تست (1 هفته)
    Header always set Strict-Transport-Security "max-age=604800"
    
    # مرحله 2: بعد از تست (1 ماه)
    # Header always set Strict-Transport-Security "max-age=2592000; includeSubDomains"
    
    # مرحله 3: نهایی (1 سال)
    # Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"

    # بقیه تنظیمات...
</VirtualHost>

# Redirect HTTP به HTTPS
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    Redirect permanent / https://example.com/
</VirtualHost>
```

**روش 2: در .htaccess (root directory)**
```apache
<IfModule mod_headers.c>
    # فقط برای HTTPS
    <If "%{HTTPS} == 'on'">
        # مرحله 1: تست
        Header always set Strict-Transport-Security "max-age=604800"
        
        # مرحله 2: بعد از تست
        # Header always set Strict-Transport-Security "max-age=2592000; includeSubDomains"
        
        # مرحله 3: نهایی
        # Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    </If>
</IfModule>

# Redirect HTTP به HTTPS
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

**دستورات اعمال:**
```bash
# فعال‌سازی mod_headers (اگر فعال نیست)
sudo a2enmod headers
sudo a2enmod ssl
sudo a2enmod rewrite

# بررسی syntax
sudo apache2ctl configtest

# اعمال تغییرات
sudo systemctl reload apache2

# یا restart
sudo systemctl restart apache2
```

---

#### 3️⃣ cPanel / WHM

**روش 1: از طریق cPanel File Manager**

1. وارد **cPanel** شوید
2. بروید به **File Manager**
3. فایل `.htaccess` در root directory را باز کنید
4. کد زیر را اضافه کنید:

```apache
# در بالای فایل .htaccess
<IfModule mod_headers.c>
    <If "%{HTTPS} == 'on'">
        # مرحله 1: تست (1 هفته)
        Header always set Strict-Transport-Security "max-age=604800"
    </If>
</IfModule>

# اطمینان از redirect HTTP به HTTPS
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

**روش 2: از طریق WHM (دسترسی root)**

1. وارد **WHM** شوید
2. بروید به **Service Configuration** → **Apache Configuration** → **Include Editor**
3. انتخاب کنید: **Pre VirtualHost Include** → **All Versions**
4. کد زیر را اضافه کنید:

```apache
<IfModule mod_headers.c>
    Header always set Strict-Transport-Security "max-age=604800"
</IfModule>
```

5. Save و Rebuild Configuration
6. Restart Apache:
```bash
/scripts/restartsrv_httpd
```

---

#### 4️⃣ Plesk

1. وارد **Plesk Panel** شوید
2. بروید به **Domains** → انتخاب domain
3. بروید به **Apache & nginx Settings**
4. در بخش **Additional nginx directives** کد زیر را اضافه کنید:

```nginx
add_header Strict-Transport-Security "max-age=604800" always;
```

5. برای Apache، در **Additional directives for HTTP** و **Additional directives for HTTPS**:

```apache
# فقط در بخش HTTPS
Header always set Strict-Transport-Security "max-age=604800"
```

6. کلیک **OK** و Apply changes

---

#### 5️⃣ Cloudflare (اگر استفاده می‌کنید)

Cloudflare به طور پیش‌فرض HSTS را پشتیبانی می‌کند:

1. وارد **Cloudflare Dashboard** شوید
2. انتخاب domain
3. بروید به **SSL/TLS** → **Edge Certificates**
4. پیدا کنید: **HTTP Strict Transport Security (HSTS)**
5. **Enable HSTS** کلیک کنید
6. تنظیمات:
   - **Max Age**: 12 months (31536000)
   - **Include subdomains**: فعال (اگر همه subdomain‌ها SSL دارند)
   - **Preload**: فعال (اختیاری)
   - **No-Sniff header**: فعال (توصیه می‌شود)

7. **Next** و تأیید

⚠️ **نکته**: Cloudflare header را برای شما اضافه می‌کند، نیازی به تغییر در سرور نیست.

---

### 📋 استراتژی استقرار تدریجی (Deployment Strategy)

برای production، **حتماً** از این مراحل پیروی کنید:

#### مرحله 1️⃣: تست اولیه (هفته 1)
```nginx
add_header Strict-Transport-Security "max-age=604800" always;  # 1 week
```
**مدت**: 1 هفته  
**هدف**: تست و اطمینان از عدم وجود مشکل  
**بررسی**: Monitor error logs و user reports

#### مرحله 2️⃣: تأیید (هفته 2-5)
```nginx
add_header Strict-Transport-Security "max-age=2592000; includeSubDomains" always;  # 1 month
```
**مدت**: 1 ماه  
**هدف**: اعمال به subdomain‌ها و تست گسترده  
**بررسی**: تست تمام subdomain‌ها

#### مرحله 3️⃣: نهایی (بعد از تأیید)
```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;  # 1 year
```
**مدت**: دائمی (1 سال)  
**هدف**: امنیت کامل و آمادگی برای Preload  
**بررسی**: ثبت در hstspreload.org (اختیاری)

---

### ✅ بررسی و تست در Production

#### 1. تست با cURL
```bash
# تست HTTPS
curl -I https://your-domain.com

# بررسی redirect از HTTP
curl -I http://your-domain.com

# تست subdomain
curl -I https://api.your-domain.com
```

**خروجی مورد انتظار:**
```
HTTP/2 200
strict-transport-security: max-age=604800
```

#### 2. تست در مرورگر
1. باز کنید: DevTools (`F12`) → Network
2. Reload صفحه
3. بررسی Response Headers
4. تست در حالت Incognito (بدون cache)

#### 3. تست با ابزارهای آنلاین
```bash
# Security Headers
https://securityheaders.com/?q=https://your-domain.com

# SSL Labs
https://www.ssllabs.com/ssltest/analyze.html?d=your-domain.com

# PageSpeed Insights
https://pagespeed.web.dev/analysis?url=https://your-domain.com
```

#### 4. بررسی Logs
```bash
# Nginx Error Log
sudo tail -f /var/log/nginx/error.log

# Apache Error Log
sudo tail -f /var/log/apache2/error.log

# Check for SSL errors
sudo grep -i "ssl" /var/log/nginx/error.log
```

---

### 🎯 برای تیم SEO

#### چک‌لیست SEO بعد از فعال‌سازی HSTS:

- [ ] ✅ تمام URL‌ها در Google Search Console به HTTPS تغییر کرده‌اند
- [ ] ✅ Sitemap با URL‌های HTTPS به‌روز شده
- [ ] ✅ Canonical URLs همه HTTPS هستند
- [ ] ✅ Open Graph و Twitter Cards با HTTPS
- [ ] ✅ Schema.org JSON-LD URLs با HTTPS
- [ ] ✅ Robots.txt و ads.txt قابل دسترسی هستند
- [ ] ✅ هیچ Mixed Content Warning در Console وجود ندارد
- [ ] ✅ Crawl Errors در GSC چک شود

#### دستورات WP-CLI برای تیم SEO:
```bash
# تغییر تمام URL‌ها به HTTPS
wp search-replace 'http://your-domain.com' 'https://your-domain.com' --all-tables --dry-run
wp search-replace 'http://your-domain.com' 'https://your-domain.com' --all-tables

# بررسی URLs در database
wp db query "SELECT * FROM wp_options WHERE option_value LIKE '%http://%' LIMIT 10"

# Flush cache
wp cache flush

# Regenerate sitemap (Rank Math)
wp rank-math sitemap generate
```

---

### 🚨 Troubleshooting در Production

#### مشکل 1: سایت قابل دسترسی نیست بعد از HSTS

**علت**: SSL certificate مشکل دارد یا Mixed Content وجود دارد

**راه‌حل سریع:**
```nginx
# موقتاً HSTS را غیرفعال کنید
# add_header Strict-Transport-Security "max-age=604800" always;
add_header Strict-Transport-Security "max-age=0" always;
```

```bash
sudo systemctl reload nginx
```

سپس مشکل SSL را حل کنید و دوباره فعال کنید.

#### مشکل 2: Subdomain قابل دسترسی نیست

**علت**: `includeSubDomains` فعال است اما subdomain SSL ندارد

**راه‌حل:**
```nginx
# حذف includeSubDomains
add_header Strict-Transport-Security "max-age=604800" always;
```

یا SSL برای subdomain نصب کنید:
```bash
# با Let's Encrypt
sudo certbot --nginx -d subdomain.your-domain.com
```

#### مشکل 3: PageSpeed هنوز Warning نشان می‌دهد

**بررسی:**
```bash
# آیا header ارسال می‌شود؟
curl -I https://your-domain.com | grep -i strict

# آیا در صفحات error هم هست؟
curl -I https://your-domain.com/404-page | grep -i strict
```

**راه‌حل:** اطمینان حاصل کنید `always` directive وجود دارد:
```nginx
add_header Strict-Transport-Security "max-age=604800" always;
                                                        ^^^^^^
```

---

### 📞 تماس با تیم DevOps

در صورت بروز مشکل در production:

1. **فوری:** غیرفعال کردن HSTS:
   ```nginx
   add_header Strict-Transport-Security "max-age=0" always;
   ```

2. **بررسی logs:**
   ```bash
   sudo tail -100 /var/log/nginx/error.log
   ```

3. **تست SSL:**
   ```bash
   openssl s_client -connect your-domain.com:443 -servername your-domain.com
   ```

4. **برگرداندن تغییرات:**
   ```bash
   # بازگشت به آخرین commit
   git revert HEAD
   sudo systemctl reload nginx
   ```

---

### 📊 Monitoring و Alerting

#### Setup Monitoring:
```bash
# Script برای بررسی HSTS header
#!/bin/bash
DOMAIN="your-domain.com"
HSTS=$(curl -s -I https://$DOMAIN | grep -i "strict-transport-security")

if [ -z "$HSTS" ]; then
    echo "⚠️ HSTS header not found!"
    # ارسال alert به Slack/Email
else
    echo "✅ HSTS is active: $HSTS"
fi
```

#### با cron job:
```bash
# هر ساعت چک کند
0 * * * * /path/to/check-hsts.sh
```

---

## 📋 خلاصه برای تیم

### برای DevOps:
1. ✅ SSL معتبر نصب شود
2. ✅ HTTP → HTTPS redirect فعال شود
3. ✅ HSTS با `max-age=604800` شروع شود (1 هفته تست)
4. ✅ بعد از 1 هفته: `max-age=2592000; includeSubDomains` (1 ماه)
5. ✅ بعد از 1 ماه: `max-age=31536000; includeSubDomains; preload` (نهایی)
6. ✅ Monitor logs و error reports

### برای SEO:
1. ✅ بررسی Mixed Content در همه صفحات
2. ✅ Update sitemap با HTTPS URLs
3. ✅ بررسی canonical URLs
4. ✅ تست در Google Search Console
5. ✅ Monitor crawl errors
6. ✅ بررسی PageSpeed Insights

### فایل‌های مهم:
- **Nginx**: `/etc/nginx/sites-available/your-domain.conf`
- **Apache**: `/etc/apache2/sites-available/your-domain.conf` یا `.htaccess`
- **Logs**: `/var/log/nginx/error.log` یا `/var/log/apache2/error.log`

---

## �📚 منابع بیشتر

### Documentation:
- [MDN: Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)
- [OWASP: HSTS Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)

### Test Tools:
- [Security Headers Checker](https://securityheaders.com/)
- [SSL Labs Test](https://www.ssllabs.com/ssltest/)
- [HSTS Preload Status](https://hstspreload.org/)

### Related Docs:
- [HTTP Headers Optimization](HTTP-HEADERS-OPTIMIZATION.md)
- [CSP Security](CSP_SECURITY.md)
- [PageSpeed Optimizations](PAGESPEED-OPTIMIZATIONS.md)

---

## 📝 خلاصه

✅ **پیکربندی شده:**
```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
```

✅ **مزایا:**
- 🔒 محافظت از Man-in-the-Middle attacks
- 🚫 جلوگیری از SSL stripping attacks  
- ⚡ بهبود امتیاز امنیتی
- ✅ رفع warning در PageSpeed Insights

✅ **تست:**
```bash
curl -I https://your-domain.com | grep -i strict
```

⚠️ **نکات مهم:**
- برای اولین بار از `max-age` کوتاه شروع کنید
- همه subdomain‌ها باید SSL داشته باشند
- Preload list فقط در صورت اطمینان کامل

---

**آخرین بروزرسانی:** 23 دسامبر 2025  
**وضعیت:** ✅ فعال و تست شده
