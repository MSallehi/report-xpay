# 📝 تاریخچه تغییرات قالب XPay

تمام تغییرات مهم در این پروژه در این فایل مستند می‌شود.

---

## [2.1.0] - 2025-12-07

### ✨ ویژگی‌های جدید

#### 🗺️ Source Maps برای Webpack
- **افزودن پشتیبانی کامل از Source Maps**
  - پیکربندی webpack برای تولید source maps
  - ساخت خودکار فایل‌های versioned با source maps
  - رفع خطای PageSpeed "Missing source maps for large JavaScript"
  - اسکریپت‌های خودکار برای ساخت نسخه‌های versioned:
    - `create-versioned-sourcemaps.js` (Node.js)
    - `create-versioned-sourcemaps.ps1` (PowerShell)
  - بهینه‌سازی Browser Cache Management برای regenerate کردن source maps

#### 🔔 بهینه‌سازی Najva Notifications
- **رفع مشکل درخواست Permission بدون Context**
  - Load کردن Najva بر اساس user interaction (scroll, click, touch)
  - بررسی هوشمند وضعیت notification permission
  - Load فوری برای کاربرانی که قبلاً permission داده‌اند
  - قابلیت غیرفعال‌سازی از طریق `.env` (`ENABLE_NAJVA`)
  - غیرفعال خودکار در debug mode
  - رفع خطای PageSpeed: "Requests notification permission on page load"

### 🔧 بهبودها

#### Security
- پیاده‌سازی Content Security Policy (CSP) قوی
- افزودن CSP headers به جای meta tag (امنیت بالاتر)
- محافظت در برابر حملات XSS
- تعریف دایرکتیوهای script-src و object-src
- بلاک کردن اجرای اسکریپت‌های ناامن
- تنظیم object-src به 'none' برای جلوگیری از injection
- محدود کردن منابع قابل اجرا به دامنه‌های مشخص
- **به‌روزرسانی CSP برای رفع Console Errors**
  - اضافه `api.xpay.co` به connect-src
  - اضافه `s1.mediaad.org` به script-src
  - رفع خطای "Refused to connect" برای API requests
  - رفع خطای "Refused to load script" برای mediaad retargeting

#### Performance
- **افزودن Page Loader برای جلوگیری از FOUC**
  - نمایش لودینگ اولیه تا آماده شدن کامل صفحه
  - جلوگیری از Flash of Unstyled Content
  - انیمیشن fade out نرم و بهینه
  - Fallback خودکار بعد از 3 ثانیه
- **بهینه‌سازی Forced Reflow در custom-coins.js**
  - اضافه `batchReadWrite()` برای جداسازی DOM reads/writes
  - اضافه `batchMutate()` برای batch کردن DOM mutations
  - بهینه‌سازی Mobile Menu operations
  - جداسازی خواندن computed styles از DOM mutations
  - کاهش reflow time از 70ms به زیر 10ms
- بهبود load time با delay در Najva script
- استفاده از `passive: true` در event listeners
- استفاده از `once: true` برای حذف خودکار listeners
- کاهش main thread blocking

#### Images & Assets
- **رفع مشکل Aspect Ratio تصاویر**
  - تصحیح ابعاد `header-qr.webp` از 200x200 به 200x71
  - حفظ نسبت تصویر واقعی (2.80:1) برای جلوگیری از distortion
  - رفع خطای PageSpeed: "Displays images with incorrect aspect ratio"
- **اضافه خودکار width و height به تصاویر محتوا**
  - Filter خودکار برای `the_content`
  - خواندن ابعاد از metadata تصاویر
  - پشتیبانی از کلاس wp-image-{id}
  - بهبود CLS (Cumulative Layout Shift)
  - رفع خطای PageSpeed: "Image elements do not have explicit width and height"

### 🐛 رفع باگ‌ها

#### Error Handling
- **بهبود Error Handling در Page Loader**
  - اضافه try-catch برای removeChild
  - بررسی وجود body قبل از classList
  - Double check برای وجود loader قبل از حذف
  - رفع TypeError: "Cannot read properties of null"

### 🔄 CI/CD

- **بهینه‌سازی GitHub Actions Deploy Workflow**
  - اضافه Node.js setup به workflow
  - نصب خودکار NPM dependencies
  - اجرای خودکار `npm run build:production`
  - ساخت خودکار versioned assets قبل از deploy
  - آپلود فایل‌های versioned به سرور
  - حذف فایل‌های بیلد از exclude list

#### Developer Experience
- اضافه شدن npm script جدید: `build:production`
- مستندسازی کامل Source Maps در `docs/SOURCE_MAPS_README.md`
- مستندسازی کامل Najva در `docs/NAJVA_OPTIMIZATION.md`
- راهنمای گام به گام برای build و deploy

### 📚 مستندات
- انتقال تمام فایل‌های `.md` به پوشه `docs/`
- ایجاد پوشه `docs/changelog/`
- به‌روزرسانی `README.md` با لینک‌های جدید
- افزودن بخش Changelog به README اصلی

---

## [2.0.0] - 2025-11-01

### ✨ ویژگی‌های جدید

#### 🏗️ معماری MVC
- **پیاده‌سازی کامل معماری MVC** (مشابه Laravel)
  - ایجاد BaseController برای تمام کنترلرها
  - جداسازی Controllers: PageController, ArchiveController, SingleController
  - سیستم View و Template مشابه Laravel
  - پشتیبانی از Partial Views و Components

#### 🔄 سیستم Routing
- **سیستم Routing خودکار**
  - Route Registration برای تمام templates
  - پشتیبانی از Custom Post Types
  - پشتیبانی از Taxonomies
  - مدیریت خودکار 404 errors

#### 📁 ساختار فایل‌ها
- **Migration کامل به ساختار MVC**
  - انتقال تمام templates به `views/`
  - ایجاد partial views در `views/partials/`
  - سازماندهی مجدد asset files
  - جداسازی concerns در `app/`

### 🚀 بهینه‌سازی‌های PageSpeed

#### Critical Path Optimization
- **کاهش 47% در Critical Path Latency**
  - تبدیل CSS های blocking به non-blocking
  - Inline کردن Critical CSS
  - Lazy loading برای above-the-fold content
  - Preload برای فونت‌های مهم

#### JavaScript Optimization
- **حذف 100% Forced Reflows**
  - بهینه‌سازی DOM manipulation
  - استفاده از DocumentFragment
  - Batch DOM updates
  - requestAnimationFrame برای animations

#### YouTube Embed Optimization
- **کاهش 90% در Initial Load**
  - YouTube Facade برای preview
  - Load on click به جای autoload
  - کاهش از 700KB به 70KB
  - بهبود LCP و TBT

#### CSS Optimization
- **حذف Block Library CSS (13.8 KB)**
  - غیرفعال‌سازی Gutenberg styles
  - حذف CSS های استفاده نشده
  - Minification و compression
  - Critical CSS extraction

### 🔧 بهبودهای SEO

#### Rank Math Integration
- **بهینه‌سازی Schema و JSON-LD**
  - سفارشی‌سازی کامل Schema Types
  - Breadcrumb خودکار
  - Canonical URLs
  - Robots Meta Tags
  - FAQ Schema optimization

#### Redirects و URL Management
- **SEO Redirects خودکار**
  - رفع Duplicate Content با Trailing Slash
  - 301 Redirects برای تمام Post Types
  - پشتیبانی از Taxonomies
  - بهینه‌سازی Crawl Budget

### 🌍 GeoLocation

#### IP-based Location Detection
- **سیستم تشخیص موقعیت جغرافیایی**
  - تشخیص کشور از IP
  - محدودسازی دسترسی (ایران)
  - پشتیبانی از چندین API Provider
  - Fallback خودکار در صورت خطا
  - Helper functions: `xpay_is_user_from_iran()`

### 📚 مستندات

#### راهنماهای توسعه
- **مستندات جامع برای توسعه‌دهندگان**
  - DEVELOPER-GUIDE.md: راهنمای افزودن Template
  - MVC-ARCHITECTURE.md: معماری و ساختار
  - MIGRATION-GUIDE.md: راهنمای Migration
  - DEPLOYMENT.md: راهنمای Deploy
  - PAGESPEED-DEVELOPER-GUIDE.md: راهنمای PageSpeed
  - GIT-FUNCTIONS-GUIDE.md: توابع PowerShell

### 🔄 GitHub Actions

#### CI/CD Pipeline
- **Deploy خودکار**
  - Workflow برای Staging
  - Workflow برای Production
  - Pull Request automation
  - Error handling و notifications

---

## [1.0.0] - 2025-10-15

### ✨ نسخه اولیه

#### ویژگی‌های اصلی
- نصب و راه‌اندازی قالب WordPress
- پیاده‌سازی اولیه معماری
- ادغام با Advanced Custom Fields
- پیکربندی اولیه Rank Math
- راه‌اندازی Docker environment

---

## راهنمای نگارش Changelog

### انواع تغییرات
- **✨ Added**: ویژگی‌های جدید
- **🔧 Changed**: تغییرات در عملکرد موجود
- **⚠️ Deprecated**: ویژگی‌هایی که به زودی حذف می‌شوند
- **🗑️ Removed**: ویژگی‌های حذف شده
- **🐛 Fixed**: رفع باگ‌ها
- **🔒 Security**: رفع مشکلات امنیتی
- **🚀 Performance**: بهبود عملکرد
- **📚 Documentation**: تغییرات در مستندات

---

**نگهداری شده توسط:** تیم توسعه XPay  
**آخرین به‌روزرسانی:** 2025-12-07
