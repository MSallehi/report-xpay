# 📝 تاریخچه تغییرات قالب XPay

تمام تغییرات مهم در این پروژه در این فایل مستند می‌شود.

---

## [2.2.0] - 2025-12-08

### ⚡ بهینه‌سازی عملکرد (Performance)

#### 🎯 رفع کامل Forced Reflow در صفحات Coin
- **مشکل:** PageSpeed Insights 373ms forced reflow گزارش می‌کرد
- **راه‌حل:**
  - پیاده‌سازی `reflow-fix.js` - سیستم جامع DOMQueue
  - Cache کردن scroll position برای جلوگیری از repeated reads
  - Batch کردن تمام DOM reads و writes
  - استفاده از IntersectionObserver به جای manual scroll checking
- **نتیجه:** کاهش 97% زمان forced reflow (373ms → <10ms)

#### 📦 ماژول `reflow-fix.js` (Core Performance Module)
**ویژگی‌های اصلی:**

- `window.DOMQueue` - صف بهینه برای batch operations
  - `DOMQueue.read()` - جمع‌آوری تمام DOM reads
  - `DOMQueue.write()` - اعمال تمام DOM writes
  - Automatic scheduling با requestAnimationFrame

- `window.getScrollPositionSafe()` - دریافت scroll بدون reflow
  - Cache شده per-frame
  - Auto-update با passive listeners
  - صفر overhead برای multiple calls

- `window.getDimensionsSafe(element)` - همه dimensions با یک read
  - Width, height, client, scroll dimensions
  - Cache تا آخر frame
  - Perfect برای responsive calculations

- `element.getBoundingClientRectCached()` - cached rect queries
  - Override بهینه شده getBoundingClientRect
  - WeakMap caching
  - Frame-based invalidation

- `window.isElementVisibleSafe(element)` - visibility بدون reflow
  - استفاده از IntersectionObserver
  - هیچ getBoundingClientRect ندارد
  - Auto-observes و caches results

- Helper functions:
  - `animateElementSafe()` - animations بدون reflow
  - `setStylesSafe()` - batch style changes
  - jQuery integration برای سازگاری

#### 🔧 بهینه‌سازی `custom-coins.js`
- Refactor تمام scroll handlers با `getScrollPositionSafe()`
- Batch کردن TOC scroll operations با DOMQueue
- بهینه‌سازی gift box scrolling
- جداسازی کامل read/write operations

#### 📊 نتایج Benchmark

**قبل:**
```
[unattributed]       155 ms ⛔
/coin/uvoucher/:1456  92 ms ⛔
/coin/uvoucher/:1457  92 ms ⛔
custom-coins.js        2 ms
app-vendor.js         32 ms
─────────────────────────
مجموع:              373 ms ⛔
```

**بعد:**
```
reflow-fix.js          0 ms ✅
custom-coins.js        0 ms ✅
app-vendor.js         ~5 ms ✅
─────────────────────────
مجموع:              < 10 ms 🎉
```

#### 🌐 بهینه‌سازی CDN Cache Lifetime
- **مشکل:** PageSpeed هشدار "Use efficient cache lifetimes" می‌داد
  - فایل‌های CDN فقط 7 روز cache می‌شدند
  - Estimated savings: 38 KiB
- **راه‌حل:**
  - ایجاد فایل `.htaccess` اختصاصی برای CDN
  - تنظیم Cache-Control: `max-age=31536000` (1 سال)
  - اضافه کردن `immutable` directive برای فایل‌های استاتیک
  - پیکربندی CORS headers برای cross-origin requests
- **فایل‌های تأثیرگذار:**
  - تصاویر: jpg, jpeg, png, gif, webp, svg, ico
  - فونت‌ها: woff, woff2, ttf, eot, otf
  - CSS/JS: با Vary: Accept-Encoding
  - مدیا: mp4, webm, ogg, mp3, pdf
- **نتیجه:** 
  - ✅ حذف کامل PageSpeed warning
  - ✅ کاهش بار سرور و CDN
  - ✅ بهبود سرعت برای بازدیدکنندگان تکراری
  - ✅ بهبود امتیاز PageSpeed: +3 تا +5
- **مستندات:** مراجعه کنید به `docs/CDN-CACHE-OPTIMIZATION.md`

#### ⚡ بهینه‌سازی JavaScript Execution Time
- **مشکل:** PageSpeed هشدار "Reduce JavaScript execution time: 2.1s"
  - `app-vendor.js` خیلی سنگین: 1.25 MB (1,497ms CPU time)
  - همه کتابخانه‌ها در یک فایل: React, Highcharts, moment, axios
  - Main-thread blocking برای مدت طولانی
- **راه‌حل:**
  - **Code Splitting**: تقسیم به 8 bundle جداگانه
    - `runtime.js` (3.32 KB) - webpack runtime
    - `app-react.js` (133 KB) - React + ReactDOM
    - `app-highcharts.js` (726 KB) - Highcharts
    - `app-datetime.js` (308 KB) - moment + dayjs
    - `app-vendor.js` (118 KB) - بقیه vendorها
    - `app-chart.js` (29 KB)
    - `app-calculator.js` (21 KB)
    - `app-coins.js` (1.5 KB)
  - **TerserPlugin**: minification پیشرفته
    - حذف console.log در production
    - حذف کدهای غیرفعال (dead code)
    - حذف کامنت‌ها
    - Parallel processing با multi-core
  - **Webpack Optimization**:
    - Tree shaking برای حذف کدهای استفاده نشده
    - Priority-based cacheGroups
    - Runtime chunk جداگانه
- **فایل‌های تغییریافته:**
  - `webpack.config.js` - افزودن TerserPlugin و splitChunks پیشرفته
  - `app/Support/Assets.php` - بروزرسانی enqueue با dependencies صحیح
  - `app/Admin/PageSpeedController.php` - بروزرسانی defer_scripts
- **نتیجه:**
  - ✅ کاهش 45% در JS execution time (2,743ms → ~1,500ms)
  - ✅ کاهش 60% در parse time برای vendor bundle
  - ✅ Parallel loading فایل‌ها توسط مرورگر
  - ✅ بهبود browser cache (تغییر یک کتابخانه بقیه را invalidate نمی‌کند)
  - ✅ کاهش main-thread blocking
- **مستندات:** مراجعه کنید به `docs/JS-EXECUTION-OPTIMIZATION.md`

#### 📦 بهینه‌سازی Unused JavaScript (Tree Shaking)
- **مشکل:** PageSpeed هشدار "Reduce unused JavaScript: 271 KiB"
  - `app-highcharts.js`: 157 KB unused (69% استفاده نشده)
  - `app-datetime.js`: 57.5 KB unused (moment-jalaali سنگین)
  - `swiper.js`: 22.6 KB unused
- **راه‌حل:**
  - **Dynamic Import Highcharts**:
    - Lazy load با `import()` به جای static import
    - فقط زمانی بارگذاری می‌شود که نمودار نمایش داده شود
    - Chunk‌های جداگانه: core, react, boost
  - **جایگزینی moment-jalaali با dayjs + jalaliday**:
    - moment-jalaali: ~300 KB
    - dayjs + jalaliday: ~15 KB (✅ 95% کاهش)
  - **Webpack Tree Shaking**:
    - `sideEffects: false` در package.json
    - `usedExports: true` در webpack.config
    - Terser passes: 2 برای compression بهتر
- **فایل‌های تغییریافته:**
  - `src/components/CoinChart.jsx` - dynamic import Highcharts, dayjs
  - `webpack.config.js` - tree shaking فعال
  - `package.json` - sideEffects: false, jalaliday اضافه شد
- **نتیجه:**
  - ✅ کاهش ~60% unused code در Highcharts
  - ✅ حذف app-datetime.js (285 KB کاهش)
  - ✅ dayjs: 15 KB vs moment: 300 KB
  - ✅ Lazy loading: chart فقط وقتی لازم است load می‌شود
- **مستندات:** مراجعه کنید به `docs/JS-EXECUTION-OPTIMIZATION.md`

#### 🗺️ رفع Source Maps برای Webpack Bundles
- **مشکل:** PageSpeed warning "Missing source maps for large JavaScript"
  - فایل‌های `.map` در deploy exclude شده بودند
  - SyntaxError: Unexpected token '<' در source maps
  - 404 errors برای تمام `.v5.5.x.js.map` فایل‌ها
- **راه‌حل:**
  - بروزرسانی `create-versioned-sourcemaps.js` برای شامل شدن همه bundle ها:
    - runtime, app-react, app-highcharts, app-datetime, app-vendor
    - app-calculator, app-chart, app-coins
  - بروزرسانی `.github/workflows/deploy.yml`:
    - حذف `assets/js/*.map` از exclude list
    - فقط base files (non-versioned) exclude می‌شوند
    - Versioned files با `.v5.5.x.js.map` deploy می‌شوند
- **فایل‌های تغییریافته:**
  - `create-versioned-sourcemaps.js` - لیست فایل‌ها به 8 bundle افزایش یافت
  - `.github/workflows/deploy.yml` - exclude pattern بهینه شد
- **نتیجه:**
  - ✅ تمام source maps در production در دسترس هستند
  - ✅ Browser DevTools می‌تواند کد اصلی را نمایش دهد
  - ✅ Debugging در production راحت‌تر شد
  - ✅ PageSpeed warning برطرف شد

#### 🔄 اتوماسیون Source Maps در AssetVersionManager
- **مشکل:** source map ها باید به صورت دستی توسط اسکریپت ساخته شوند
- **راه‌حل:**
  - اضافه شدن متد `createVersionedSourceMap()` به `AssetVersionManager.php`:
    - هنگام ساخت فایل JS ورژن‌دار، source map هم خودکار ساخته می‌شود
    - کپی فایل `.js.map` → `.v5.5.8.js.map`
    - به‌روزرسانی خودکار `sourceMappingURL` در فایل JS
  - به‌روزرسانی `BrowserCacheAdmin::regenerateVersionedSourceMaps()`:
    - لیست webpack bundles از 4 به 8 افزایش یافت
    - همگام با تغییرات code splitting
- **چرخه کار:**
  ```
  کاربر صفحه را باز می‌کند
      ↓
  AssetVersionManager ورژن را از .env می‌خواند (مثلا 5.5.8)
      ↓
  بررسی: app-vendor.v5.5.8.js وجود دارد?
      ↓
  اگر نه → فایل‌های ورژن‌دار + source map ساخته می‌شوند
      ↓
  ادمین روی "Clear Browser Cache" کلیک می‌کند
      ↓
  BrowserCacheAdmin ورژن را افزایش می‌دهد: 5.5.8 → 5.5.9
      ↓
  فایل‌های قدیمی پاک + فایل‌های جدید با source maps ساخته می‌شوند
  ```
- **مزایا:**
  - ✅ کاملا خودکار - نیازی به اسکریپت دستی نیست
  - ✅ همیشه source map با فایل JS همخوانی دارد
  - ✅ cache کردن برای بارگیری‌های بعدی
  - ✅ پشتیبانی کامل از 8 webpack bundle
  - ✅ sourceMappingURL همیشه صحیح است
- **فایل‌های تغییر یافته:**
  - `app/Support/AssetVersionManager.php` - اضافه شدن متد createVersionedSourceMap
  - `app/Admin/BrowserCacheAdmin.php` - به‌روزرسانی لیست webpack bundles

**بهبود:** 97% کاهش (363ms صرفه‌جویی)

#### 📚 مستندات
- اضافه شدن `FORCED-REFLOW-OPTIMIZATION.md`
  - توضیح کامل forced reflow و layout thrashing
  - مثال‌های قبل/بعد
  - Best practices
  - Testing guide
  - Integration checklist

### 🔧 بهبودها

#### Load Priority
- `reflow-fix.js` با بالاترین priority لود می‌شود
- Load در `<head>` به جای footer
- صفر dependencies - FIRST script to load
- تضمین دسترسی همه scripts به DOMQueue

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
