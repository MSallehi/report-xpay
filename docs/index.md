# 📊 مستندات فنی قالب XPay
# XPay WordPress Theme Documentation

> **نسخه:** 5.5.8  
> **آخرین بروزرسانی:** 7 دسامبر 2025  
> **وضعیت:** 🟢 فعال و در حال توسعه

---

## 🎯 درباره این مستندات

این مجموعه شامل مستندات کامل فنی، راهنماهای توسعه، و گزارش‌های بهینه‌سازی قالب وردپرس XPay است که برای استفاده در GitHub Pages طراحی شده است.

---

## 📚 دسته‌بندی مستندات

### 🚀 شروع سریع
ابتدا از اینجا شروع کنید!

- [**📖 راهنمای شروع سریع**](QUICK-START.md)
  - نصب و راه‌اندازی
  - پیکربندی اولیه
  - اولین Template

- [**🎯 راهنمای توسعه‌دهندگان**](DEVELOPER-GUIDE.md)
  - افزودن Page Template
  - افزودن Archive Template
  - ساخت Controller جدید
  - کار با View ها

### 🏗️ معماری و ساختار

- [**🔷 معماری MVC**](mvc-architecture.md)
  - ساختار MVC (مشابه Laravel)
  - سیستم Routing
  - Controllers و Views
  - مثال‌های کامل

- [**📦 Migration Guide**](MIGRATION-GUIDE.md)
  - مراحل migration به MVC
  - رفع مشکلات رایج
  - Best Practices

### ⚡ عملکرد و بهینه‌سازی

- [**📊 گزارش بهینه‌سازی عملکرد**](PERFORMANCE-REPORT.md) 🆕
  - خلاصه تغییرات
  - نتایج قبل و بعد
  - راه‌حل‌های اعمال شده
  - تست و تأیید

- [**🚀 بهینه‌سازی‌های PageSpeed**](PAGESPEED-OPTIMIZATIONS.md)
  - Critical Path Optimization
  - Resource Loading
  - Asset Versioning
  - 2,500+ خطوط مستندات

- [**⚡ بهینه‌سازی Forced Reflow**](FORCED_REFLOW_OPTIMIZATION.md)
  - تحلیل Reflows
  - Highcharts Optimization
  - React Rendering
  - CSS Containment

- [**🎛️ کنترلر PageSpeed**](PAGESPEED_CONTROLLER.md)
  - API مدیریت PageSpeed
  - پنل ادمین
  - تنظیمات پیشرفته

- [**📱 بهینه‌سازی Najva**](NAJVA_OPTIMIZATION.md)
  - Lazy Loading
  - Performance Tips

### 🔒 امنیت

- [**🛡️ Content Security Policy**](CSP_SECURITY.md)
  - تنظیمات CSP
  - رفع Violations
  - Best Practices

### 🔧 پیکربندی و تنظیمات

- [**🌐 تنظیمات CDN**](CDN-CONFIGURATION.md)
  - پیکربندی CDN
  - Optimization Tips

- [**🗺️ تنظیمات Rank Math SEO**](rank-math-configuration.md)
  - Schema و JSON-LD
  - Breadcrumb
  - Canonical URLs

- [**🌍 GeoLocation**](GEOLOCATION.md)
  - تشخیص موقعیت جغرافیایی
  - IP-based Restrictions

### 🚀 استقرار و CI/CD

- [**🚀 راهنمای Deploy**](DEPLOYMENT.md)
  - GitHub Actions
  - Staging و Production
  - Workflow

- [**📋 Deploy سریع**](DEPLOY-QUICK.md)
  - دستورالعمل‌های سریع
  - Troubleshooting

- [**🖥️ Deploy روی cPanel**](CPANEL-DEPLOYMENT.md)
  - راهنمای cPanel
  - FTP Upload

### 🔄 مهاجرت و Redirects

- [**🔀 Domain Redirect Fix**](DOMAIN-REDIRECT-FIX.md)
  - رفع مشکلات redirect
  - Environment Configuration

- [**🔗 SEO Redirects**](SEO-REDIRECTS.md)
  - مدیریت 301 Redirects
  - Rank Math Configuration

### 🛠️ ابزارها و Scripts

- [**⚙️ Git Functions Guide**](GIT-FUNCTIONS-GUIDE.md)
  - PowerShell Functions
  - Automation Scripts

- [**🗺️ Source Maps**](SOURCE_MAPS_README.md)
  - تولید Source Maps
  - Debugging

### 🔧 Accessibility

- [**♿ Accessibility Fixes**](ACCESSIBILITY-FIXES.md)
  - ARIA Labels
  - Keyboard Navigation
  - Screen Reader Support

---

## 📊 گزارش‌های فنی

### گزارش بهینه‌سازی عملکرد (جدید!)

**📊 [Performance Optimization Report](PERFORMANCE-REPORT.md)**

کامل‌ترین گزارش بهینه‌سازی شامل:

#### 📈 نتایج کلیدی:
| متریک | قبل | بعد | بهبود |
|-------|-----|-----|-------|
| Forced Reflows | 400ms | <50ms | ✅ 87.5% |
| Highcharts | 170ms | <10ms | ✅ 94% |
| Console Errors | 12+ | 0 | ✅ 100% |
| CSP Violations | 8 | 0 | ✅ 100% |

#### 🎯 بخش‌های گزارش:
1. **خلاصه اجرایی** - نتایج کلی
2. **مشکلات و راه‌حل‌ها** - تحلیل دقیق هر مشکل
3. **کدهای اعمال شده** - مثال‌های واقعی
4. **تست و تأیید** - نتایج آزمایش
5. **پیشنهادات آینده** - Next Steps

[**📖 مطالعه گزارش کامل →**](PERFORMANCE-REPORT.md)

---

## 🗂️ ساختار پروژه

```
xpay_main_theme/
├── 📁 app/
│   ├── Admin/              # پنل مدیریت
│   ├── Controllers/        # کنترلرها
│   ├── Core/              # هسته سیستم
│   ├── Services/          # سرویس‌ها
│   └── Support/           # کلاس‌های کمکی
│
├── 📁 assets/
│   ├── css/               # استایل‌ها
│   ├── js/                # اسکریپت‌ها
│   ├── fonts/             # فونت‌ها
│   └── img/               # تصاویر
│
├── 📁 src/
│   ├── js/                # React/JavaScript
│   ├── components/        # React Components
│   └── css/               # Source Styles
│
├── 📁 views/
│   ├── pages/             # صفحات اصلی
│   ├── partials/          # بخش‌های قابل استفاده مجدد
│   └── admin/             # صفحات ادمین
│
├── 📁 templates/          # Page Templates
├── 📁 docs/               # 📚 مستندات (اینجا هستید!)
│
└── 📄 functions.php       # تنظیمات اصلی WordPress
```

---

## 🔗 لینک‌های مفید

### مستندات داخلی
- [Changelog فارسی](changelog/CHANGELOG-FA.md)
- [Changelog انگلیسی](changelog/CHANGELOG-EN.md)
- [README اصلی](../README.md)

### ابزارهای توسعه
- [PageSpeed Insights](https://pagespeed.web.dev/)
- [Chrome DevTools](https://developer.chrome.com/docs/devtools/)
- [Web Vitals Extension](https://chrome.google.com/webstore/detail/web-vitals/)

### مراجع خارجی
- [WordPress Developer Resources](https://developer.wordpress.org/)
- [React Documentation](https://react.dev/)
- [Highcharts API](https://api.highcharts.com/highstock/)
- [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

---

## 🤝 مشارکت

برای مشارکت در توسعه:

1. **Fork** کنید
2. **Branch** جدید بسازید (`git checkout -b feature/amazing-feature`)
3. **Commit** کنید (`git commit -m 'Add amazing feature'`)
4. **Push** کنید (`git push origin feature/amazing-feature`)
5. **Pull Request** باز کنید

---

## 📞 پشتیبانی

- **Website:** [xpay.co](https://xpay.co)
- **Email:** support@xpay.co
- **Documentation Issues:** [GitHub Issues](https://github.com/Xpay-wp/xpay-wp/issues)

---

## 📝 لایسنس

این پروژه تحت لایسنس اختصاصی XPay است.

---

## 🏆 مشارکت‌کنندگان

- **توسعه دهنده اصلی:** Mohammad Salehi
- **تیم فنی XPay**
- **مستند سازی:** GitHub Copilot

---

## 📅 تاریخچه بروزرسانی‌ها

### نسخه 5.5.8 (7 دسامبر 2025)
- ✅ بهینه‌سازی کامل Forced Reflows
- ✅ رفع مشکلات CSP و CORS
- ✅ پیاده‌سازی Page Loader
- ✅ افزودن CI/CD Pipeline
- ✅ مستندات کامل Performance Report

### نسخه 5.5.0-5.5.7
- بهینه‌سازی‌های PageSpeed
- پیاده‌سازی معماری MVC
- تنظیمات Rank Math
- و بیشتر...

[**📖 مشاهده Changelog کامل →**](changelog/CHANGELOG-FA.md)

---

## 🎯 Quick Links

<table>
<tr>
<td width="50%">

### 🚀 برای شروع کار
- [Quick Start](QUICK-START.md)
- [Developer Guide](DEVELOPER-GUIDE.md)
- [MVC Architecture](mvc-architecture.md)

</td>
<td width="50%">

### ⚡ برای بهینه‌سازی
- [Performance Report](PERFORMANCE-REPORT.md) 🆕
- [PageSpeed Optimizations](PAGESPEED-OPTIMIZATIONS.md)
- [Forced Reflow Fix](FORCED_REFLOW_OPTIMIZATION.md)

</td>
</tr>
<tr>
<td width="50%">

### 🔒 برای امنیت
- [CSP Security](CSP_SECURITY.md)
- [GeoLocation](GEOLOCATION.md)

</td>
<td width="50%">

### 🚀 برای Deploy
- [Deployment Guide](DEPLOYMENT.md)
- [cPanel Deploy](CPANEL-DEPLOYMENT.md)
- [Quick Deploy](DEPLOY-QUICK.md)

</td>
</tr>
</table>

---

**📚 پایان صفحه اصلی مستندات**

> آخرین بروزرسانی: 7 دسامبر 2025 | نسخه: 5.5.8 | وضعیت: 🟢 Active
