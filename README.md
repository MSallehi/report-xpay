# 📚 XPay Theme Documentation

> **مستندات فنی کامل قالب وردپرس XPay**  
> Documentation for XPay WordPress Theme

[![GitHub Pages](https://img.shields.io/badge/docs-GitHub%20Pages-blue)](https://msallehi.github.io/report-xpay/)
[![Version](https://img.shields.io/badge/version-5.5.8-green)]()
[![Status](https://img.shields.io/badge/status-active-success)]()

---

## 🌐 مشاهده مستندات آنلاین

**🔗 [msallehi.github.io/report-xpay](https://msallehi.github.io/report-xpay/)**

این مستندات شامل:
- 📊 گزارش بهینه‌سازی عملکرد
- 🚀 راهنمای بهینه‌سازی PageSpeed
- 👨‍💻 راهنمای توسعه‌دهندگان
- 🏗️ معماری MVC
- 🔒 تنظیمات امنیتی
- و بیشتر...

---

## 📖 فهرست سریع

### مستندات اصلی
- [**صفحه اصلی**](https://msallehi.github.io/report-xpay/) - راهنمای کامل
- [**گزارش عملکرد**](https://msallehi.github.io/report-xpay/PERFORMANCE-REPORT) - نتایج بهینه‌سازی
- [**بهینه‌سازی PageSpeed**](https://msallehi.github.io/report-xpay/PAGESPEED-OPTIMIZATIONS) - راهنمای کامل
- [**راهنمای توسعه**](https://msallehi.github.io/report-xpay/DEVELOPER-GUIDE) - برای توسعه‌دهندگان

### راهنماهای فنی
- [معماری MVC](https://msallehi.github.io/report-xpay/mvc-architecture)
- [تنظیمات CSP](https://msallehi.github.io/report-xpay/CSP_SECURITY)
- [بهینه‌سازی Reflow](https://msallehi.github.io/report-xpay/FORCED_REFLOW_OPTIMIZATION)
- [بهینه‌سازی Cache در CDN](https://msallehi.github.io/report-xpay/CDN-CACHE-OPTIMIZATION) 🆕
- [بهینه‌سازی JavaScript Execution](https://msallehi.github.io/report-xpay/JS-EXECUTION-OPTIMIZATION) 🆕
- [کنترل Cache Version Redirect](https://msallehi.github.io/report-xpay/CACHE-VERSION-REDIRECT) 🆕
- [راهنمای Deploy](https://msallehi.github.io/report-xpay/DEPLOYMENT)

---

## 🚀 ویژگی‌های مستندات

- ✅ **راست‌چین (RTL)** - پشتیبانی کامل از فارسی
- ✅ **طراحی مدرن** - استایل حرفه‌ای با گرادیانت
- ✅ **Responsive** - سازگار با موبایل و تبلت
- ✅ **Dark Mode** - پشتیبانی از حالت تاریک
- ✅ **جستجو** - قابلیت جستجو در مستندات
- ✅ **Navigation** - منوی راحت و سریع
- ✅ **کپی کد** - دکمه کپی برای بلاک‌های کد

---

## 📊 نتایج بهینه‌سازی

| متریک | قبل | بعد | بهبود |
|-------|-----|-----|-------|
| **Forced Reflows** | 400ms | <50ms | ✅ 87.5% |
| **Highcharts** | 170ms | <10ms | ✅ 94% |
| **Console Errors** | 12+ | 0 | ✅ 100% |
| **CSP Violations** | 8 | 0 | ✅ 100% |
| **CORS Errors** | 2 | 0 | ✅ 100% |

[**📖 مطالعه گزارش کامل →**](https://msallehi.github.io/report-xpay/PERFORMANCE-REPORT)

---

## 🛠️ توسعه محلی

### نصب Jekyll (اختیاری)

برای تست محلی مستندات:

```bash
# نصب Ruby و Jekyll
gem install jekyll bundler

# Clone repository
git clone https://github.com/MSallehi/report-xpay.git
cd report-xpay

# نصب وابستگی‌ها
bundle install

# اجرای سرور محلی
bundle exec jekyll serve

# مشاهده در:
# http://localhost:4000/report-xpay/
```

### ساختار پروژه

```
docs/
├── index.md                        # صفحه اصلی
├── PERFORMANCE-REPORT.md           # گزارش عملکرد
├── PAGESPEED-OPTIMIZATIONS.md      # بهینه‌سازی PageSpeed
├── _config.yml                     # تنظیمات Jekyll
├── _layouts/
│   └── default.html                # Layout اصلی
└── assets/
    └── css/
        └── style.scss              # استایل‌های سفارشی
```

---

## 📝 نحوه مشارکت

1. Fork کنید
2. Branch جدید بسازید: `git checkout -b docs/new-feature`
3. تغییرات را Commit کنید: `git commit -m 'Add new documentation'`
4. Push کنید: `git push origin docs/new-feature`
5. Pull Request باز کنید

---

## 🎨 تم و استایل

این مستندات از **Jekyll Theme Slate** با سفارشی‌سازی‌های زیر استفاده می‌کند:

- **RTL Support** - راست‌چین کامل
- **Persian Fonts** - Vazirmatn
- **Custom Colors** - گرادیانت بنفش-آبی
- **Enhanced Tables** - جداول زیبا با hover
- **Code Blocks** - سینتکس هایلایت
- **Dark Mode** - حالت تاریک خودکار

---

## 📞 پشتیبانی

- **Website:** [xpay.co](https://xpay.co)
- **Documentation:** [msallehi.github.io/report-xpay](https://msallehi.github.io/report-xpay/)
- **Issues:** [GitHub Issues](https://github.com/MSallehi/report-xpay/issues)

---

## 📅 تاریخچه

- **v5.5.8** (دسامبر 2025) - بهینه‌سازی کامل عملکرد
- **v5.5.0-5.5.7** - بهینه‌سازی‌های PageSpeed و معماری MVC

---

## 📄 لایسنس

این مستندات تحت لایسنس اختصاصی XPay منتشر شده است.

---

**© 2025 XPay. All rights reserved.**
