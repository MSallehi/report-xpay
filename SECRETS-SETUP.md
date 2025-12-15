# 🔐 راهنمای تنظیم GitHub Secrets

## مسیر تنظیمات:
```
Repository → Settings → Secrets and Variables → Actions → New repository secret
```

یا مستقیم:
```
https://github.com/Xpay-wp/xpay-wp/settings/secrets/actions
```

---

## 🎯 Secrets مورد نیاز برای Staging (develop branch):

### 1. FTP_SERVER_STAGING
```
مثال: ftp.staging-host.com
یا: 123.456.789.10
```

### 2. FTP_USERNAME_STAGING
```
مثال: staging_user@xpay.co
```

### 3. FTP_PASSWORD_STAGING
```
رمز عبور FTP staging
```

### 4. FTP_PORT_STAGING
```
مثال: 21 (پیش‌فرض)
یا: 9004
یا: 2121
```

### 5. FTP_DIR_STAGING
```
مثال: //staging.xpay.co/wp-content/themes/xpay_main_theme/
یا: /public_html/wp-content/themes/xpay_main_theme/
یا: /home/username/staging.xpay.co/wp-content/themes/xpay_main_theme/
```

---

## 🚀 Secrets مورد نیاز برای Production (master branch):

### 1. FTP_SERVER_PRODUCTION
```
مثال: ftp.xpay.co
یا: 987.654.321.10
```

### 2. FTP_USERNAME_PRODUCTION
```
مثال: production_user@xpay.co
```

### 3. FTP_PASSWORD_PRODUCTION
```
رمز عبور FTP production
```

### 4. FTP_PORT_PRODUCTION
```
مثال: 21 (پیش‌فرض)
یا: 2121
```

### 5. FTP_DIR_PRODUCTION
```
مثال: //public_html/wp-content/themes/xpay_main_theme/
یا: /home/username/xpay.co/wp-content/themes/xpay_main_theme/
```

---

## 📋 Secrets فعلی (قدیمی) که می‌توانید حذف کنید:

بعد از تنظیم secrets جدید، این موارد دیگر استفاده نمی‌شوند:

- ❌ `FTP_SERVER` (جایگزین: FTP_SERVER_STAGING و FTP_SERVER_PRODUCTION)
- ❌ `FTP_USERNAME` (جایگزین: FTP_USERNAME_STAGING و FTP_USERNAME_PRODUCTION)
- ❌ `FTP_PASSWORD` (جایگزین: FTP_PASSWORD_STAGING و FTP_PASSWORD_PRODUCTION)

---

## ✅ Checklist تنظیمات:

### Staging:
- [ ] FTP_SERVER_STAGING
- [ ] FTP_USERNAME_STAGING
- [ ] FTP_PASSWORD_STAGING
- [ ] FTP_PORT_STAGING
- [ ] FTP_DIR_STAGING

### Production:
- [ ] FTP_SERVER_PRODUCTION
- [ ] FTP_USERNAME_PRODUCTION
- [ ] FTP_PASSWORD_PRODUCTION
- [ ] FTP_PORT_PRODUCTION
- [ ] FTP_DIR_PRODUCTION

### سایر (از قبل موجود):
- [ ] DOCS_REPO_TOKEN

---

## 🧪 تست Deploy:

### تست Staging:
```bash
git checkout develop
git commit --allow-empty -m "test: staging deploy"
git push origin develop
```

سپس بررسی کنید:
- https://github.com/Xpay-wp/xpay-wp/actions

### تست Production:
```bash
git checkout master
git commit --allow-empty -m "test: production deploy"
git push origin master
```

سپس بررسی کنید:
- https://github.com/Xpay-wp/xpay-wp/actions

---

## ⚠️ نکات مهم:

### 1. مسیر FTP (server-dir):

اگر مسیر را نمی‌دانید، می‌توانید با FileZilla متصل شوید و مسیر را ببینید:

```
// دو اسلش در اول (برای برخی سرورها)
//public_html/wp-content/themes/xpay_main_theme/

// یک اسلش در اول (برای برخی سرورها)
/public_html/wp-content/themes/xpay_main_theme/

// مسیر کامل (برای برخی سرورها)
/home/username/public_html/wp-content/themes/xpay_main_theme/
```

### 2. FTP Server:

می‌توانید از domain یا IP استفاده کنید:

```
# Domain
ftp.xpay.co

# IP (توصیه می‌شود - مستقل از DNS)
123.456.789.10
```

---

## 🔍 عیب‌یابی:

### خطای "530 Login authentication failed"
- بررسی username و password
- ممکن است نیاز به @ باشد: `user@domain.com`

### خطای "550 Permission denied"
- بررسی مسیر server-dir
- بررسی دسترسی FTP user

### خطای "Connection timeout"
- بررسی FTP_SERVER (IP یا domain)
- بررسی firewall و port

---

**تهیه شده توسط:** XPay Development Team  
**تاریخ:** 2025-12-15
