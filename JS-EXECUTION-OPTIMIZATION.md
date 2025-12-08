# 🚀 بهینه‌سازی JavaScript Execution Time

## 📋 مقدمه

این مستند راهنمای کامل برای کاهش زمان اجرای JavaScript و بهینه‌سازی bundle size در قالب XPay را ارائه می‌دهد.

---

## 🎯 مشکل

### گزارش PageSpeed Insights:
```
⚠️ Reduce JavaScript execution time: 2.1 s

URL                                    Total CPU Time    Script Evaluation    Script Parse
xpay.co 1st party                          2,743 ms           1,073 ms            93 ms
  app-vendor.v5.5.14.js                    1,497 ms           1,020 ms            88 ms
  /coin/uvoucher/                          1,246 ms              53 ms             5 ms
Unattributable                               335 ms              21 ms             0 ms
```

**مشکلات اصلی:**
1. ❌ `app-vendor.js` خیلی سنگین: **1.25 MB** (1,497ms CPU time)
2. ❌ همه کتابخانه‌ها در یک فایل: React, Highcharts, moment, axios, ...
3. ❌ Browser باید همه را parse و compile کند قبل از اجرا
4. ❌ Main-thread blocking برای مدت طولانی

---

## ✅ راه‌حل: Code Splitting و Webpack Optimization

### استراتژی:
1. **Code Splitting**: تقسیم `app-vendor.js` به چند bundle کوچک‌تر
2. **Tree Shaking**: حذف کدهای استفاده نشده
3. **Minification پیشرفته**: با TerserPlugin
4. **Lazy Loading**: بارگذاری فقط در صورت نیاز

---

## 🔧 تغییرات اعمال شده

### 1️⃣ بهینه‌سازی `webpack.config.js`

#### الف) نصب TerserPlugin:
```bash
npm install --save-dev terser-webpack-plugin
```

#### ب) افزودن Terser با تنظیمات بهینه:
```javascript
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  // ...
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true,      // حذف console.log در production
            dead_code: true,         // حذف کدهای غیرفعال
            unused: true,            // حذف متغیرهای استفاده نشده
            pure_funcs: [
              'console.info',
              'console.debug',
              'console.warn'
            ],
          },
          mangle: {
            safari10: true,          // سازگاری با Safari 10
          },
          format: {
            comments: false,         // حذف کامنت‌ها
          },
        },
        extractComments: false,
        parallel: true,              // استفاده از multi-core
      }),
    ],
    // ...
  },
};
```

#### ج) Code Splitting پیشرفته:
```javascript
optimization: {
  splitChunks: {
    chunks: "all",
    maxInitialRequests: Infinity,
    minSize: 20000,                  // فقط chunk‌های بزرگتر از 20KB
    cacheGroups: {
      // جداسازی React و ReactDOM
      react: {
        test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
        name: "app-react",
        priority: 20,
      },
      // جداسازی Highcharts
      highcharts: {
        test: /[\\/]node_modules[\\/]highcharts/,
        name: "app-highcharts",
        priority: 15,
      },
      // جداسازی moment و dayjs
      datetime: {
        test: /[\\/]node_modules[\\/](moment|dayjs|moment-jalaali)[\\/]/,
        name: "app-datetime",
        priority: 10,
      },
      // بقیه vendor ها
      vendors: {
        test: /[\\/]node_modules[\\/]/,
        name: "app-vendor",
        priority: 5,
      },
    },
  },
  runtimeChunk: 'single',            // جداسازی webpack runtime
},
```

#### د) Performance Hints:
```javascript
performance: {
  hints: 'warning',
  maxEntrypointSize: 512000,         // 500 KB
  maxAssetSize: 512000,              // 500 KB
},
```

---

## 📊 نتایج بهینه‌سازی

### قبل از بهینه‌سازی:
```
❌ app-vendor.js: 1.25 MB (1,497ms CPU time)
   - React + ReactDOM + Highcharts + moment + axios + ...
   - همه در یک فایل سنگین
```

### بعد از بهینه‌سازی:
```
✅ runtime.js:         3.32 KB  (webpack runtime)
✅ app-react.js:       133 KB   (React + ReactDOM)
✅ app-highcharts.js:  726 KB   (Highcharts + modules)
✅ app-datetime.js:    308 KB   (moment + dayjs)
✅ app-vendor.js:      118 KB   (axios, pako, ...)
✅ app-chart.js:       28.8 KB  (Chart component)
✅ app-calculator.js:  21.3 KB  (Calculator component)
✅ app-coins.js:       1.51 KB  (Entry point)
──────────────────────────────────
مجموع:               1.31 MB
```

### مزایا:
- ✅ **Parallel Loading**: مرورگر می‌تواند فایل‌ها را موازی دانلود کند
- ✅ **Browser Cache**: تغییر یک کتابخانه، بقیه را invalidate نمی‌کند
- ✅ **Parse Time**: فایل‌های کوچک‌تر سریع‌تر parse می‌شوند
- ✅ **Main-thread**: کمتر block می‌شود

---

## 🔄 تغییرات در `Assets.php`

### الف) اضافه کردن asset های جدید:
```php
// Webpack code-split bundles (optimized for better performance)
$manager->addAsset('/assets/js', 'runtime', 'js', ['runtime.js'], 'assets_version', $debugAsset);
$manager->addAsset('/assets/js', 'app-react', 'js', ['app-react.js'], 'assets_version', $debugAsset);
$manager->addAsset('/assets/js', 'app-highcharts', 'js', ['app-highcharts.js'], 'assets_version', $debugAsset);
$manager->addAsset('/assets/js', 'app-datetime', 'js', ['app-datetime.js'], 'assets_version', $debugAsset);
$manager->addAsset('/assets/js', 'app-vendor', 'js', ['app-vendor.js'], 'assets_version', $debugAsset);
```

### ب) بروزرسانی defer_scripts:
```php
$defer_scripts = [
    'runtime',           // Webpack runtime - load first
    'app-react',         // React/ReactDOM bundle
    'app-highcharts',    // Highcharts bundle
    'app-datetime',      // moment/dayjs bundle
    'app-vendor',        // Other vendor libs
    'app-coins',         // Coin-specific JS
    'app-calculator',    // Calculator JS
    'app-chart',         // Chart JS
    // ... بقیه
];
```

### ج) بروزرسانی enqueue با dependencies صحیح:
```php
// Load in correct order with dependencies
wp_enqueue_script('runtime', $manager->getUrl('runtime', 'js'), array(), $assetVersion, false);
wp_enqueue_script('app-react', $manager->getUrl('app-react', 'js'), array('runtime'), $assetVersion, false);
wp_enqueue_script('app-highcharts', $manager->getUrl('app-highcharts', 'js'), array('runtime'), $assetVersion, false);
wp_enqueue_script('app-datetime', $manager->getUrl('app-datetime', 'js'), array('runtime'), $assetVersion, false);
wp_enqueue_script('app-vendor', $manager->getUrl('app-vendor', 'js'), array('runtime'), $assetVersion, false);

// App code depends on vendor bundles
wp_enqueue_script('app-coins', $manager->getUrl('app-coins', 'js'), 
    array('runtime', 'app-react', 'app-vendor'), $assetVersion, false);
wp_enqueue_script('app-calculator', $manager->getUrl('app-calculator', 'js'), 
    array('runtime', 'app-react', 'app-vendor'), $assetVersion, false);
wp_enqueue_script('app-chart', $manager->getUrl('app-chart', 'js'), 
    array('runtime', 'app-react', 'app-highcharts', 'app-datetime'), $assetVersion, false);
```

---

## 🔄 تغییرات در `PageSpeedController.php`

### بروزرسانی defer_scripts:
```php
// Scripts to defer (execute after DOM is ready) - OPTIMIZED for JS execution time
$defer_scripts = [
    'coin-search-script',
    'bottom-sheet-script',
    'view-more-coins-script',
    'contact-form-7',
    'runtime',           // Webpack runtime - load first
    'app-react',         // React/ReactDOM bundle
    'app-highcharts',    // Highcharts bundle (726 KB)
    'app-datetime',      // moment/dayjs bundle (308 KB)
    'app-vendor',        // Other vendor libs (118 KB)
    'app-coins',         // Coin-specific JS
    'app-calculator',    // Calculator JS
    'app-chart'          // Chart JS
];
```

---

## 📈 تأثیر بر Performance Metrics

### پیش‌بینی بهبود:

| متریک | قبل | بعد | بهبود |
|-------|-----|-----|-------|
| **JS Execution Time** | 2,743 ms | ~1,500 ms | ✅ 45% |
| **app-vendor parse** | 1,497 ms | ~600 ms | ✅ 60% |
| **Bundle Size** | 1.25 MB | 1.31 MB* | ⚠️ +60 KB |
| **Main-thread Blocking** | بالا | متوسط | ✅ 40% |
| **Browser Cache** | ضعیف | عالی | ✅ 80% |

**نکته:** Bundle size کل اندکی افزایش یافته (به دلیل overhead runtime.js) اما:
- ✅ Parallel download سریع‌تر است
- ✅ Parse time کمتر است
- ✅ Browser cache بهتر کار می‌کند

---

## 🧪 تست و بررسی

### 1. Build کردن Webpack:
```bash
cd c:\Docker\xpay\wordpress\wp-content\themes\xpay_main_theme
npm run build
```

**خروجی مورد انتظار:**
```
asset app-highcharts.js 726 KiB [emitted] [minimized] [big]
asset app-datetime.js 308 KiB [emitted] [minimized]
asset app-react.js 133 KiB [emitted] [minimized]
asset app-vendor.js 118 KiB [emitted] [minimized]
asset app-chart.js 28.8 KiB [emitted] [minimized]
asset app-calculator.js 21.3 KiB [emitted] [minimized]
asset runtime.js 3.32 KiB [emitted] [minimized]
asset app-coins.js 1.51 KiB [emitted] [minimized]

webpack 5.99.8 compiled with 1 warning in 8218 ms
```

### 2. بررسی فایل‌ها در سرور:
```bash
ls assets/js/app-*.js
```

باید 8 فایل ببینید:
- `app-coins.js` (1.5 KB)
- `app-react.js` (133 KB)
- `app-highcharts.js` (726 KB)
- `app-datetime.js` (308 KB)
- `app-vendor.js` (118 KB)
- `app-chart.js` (29 KB)
- `app-calculator.js` (21 KB)
- `runtime.js` (3.3 KB)

### 3. تست در مرورگر:
```
https://staging.xpay.co/coin/uvoucher/
```

**Chrome DevTools → Network Tab:**
- ✅ باید 8 فایل JS جداگانه ببینید
- ✅ همه با `defer` attribute
- ✅ بارگذاری موازی

**Chrome DevTools → Performance Tab:**
1. Start Recording
2. Reload page
3. Stop Recording
4. بررسی "Evaluate Script" events
5. باید کاهش زمان parse و compile را ببینید

### 4. PageSpeed Insights:
```
https://pagespeed.web.dev/analysis?url=https://staging.xpay.co/coin/uvoucher/
```

**نتیجه مورد انتظار:**
```
✅ Reduce JavaScript execution time
    Total: ~1,500 ms (کاهش 45%)
    app-highcharts.js: ~400 ms
    app-datetime.js: ~150 ms
    app-react.js: ~100 ms
    app-vendor.js: ~80 ms
```

---

## 📝 نکات مهم

### ⚠️ ترتیب بارگذاری مهم است:
```html
<!-- ترتیب صحیح: -->
<script src="runtime.js" defer></script>
<script src="app-react.js" defer></script>
<script src="app-highcharts.js" defer></script>
<script src="app-datetime.js" defer></script>
<script src="app-vendor.js" defer></script>
<script src="app-coins.js" defer></script>
<script src="app-calculator.js" defer></script>
<script src="app-chart.js" defer></script>
```

### 🔄 Dependencies در WordPress:
```php
// ❌ اشتباه - no dependencies
wp_enqueue_script('app-coins', '...', array(), $version);

// ✅ درست - با dependencies
wp_enqueue_script('app-coins', '...', 
    array('runtime', 'app-react', 'app-vendor'), $version);
```

### 📦 Cache Busting:
فایل‌های جدید با versioning خودکار cache می‌شوند:
```
app-react.v5.5.14.js?ver=5.5.14
```

---

## 🎯 بهینه‌سازی‌های بیشتر (اختیاری)

### 1. Gzip/Brotli Compression:
در `.htaccess` فعال کنید:
```apache
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE application/javascript
</IfModule>
```

**تأثیر:**
- `app-highcharts.js`: 726 KB → ~180 KB (75% کاهش)
- `app-datetime.js`: 308 KB → ~70 KB (77% کاهش)

### 2. HTTP/2 Push:
در `PageSpeedController.php`:
```php
header('Link: </path/to/runtime.js>; rel=preload; as=script');
header('Link: </path/to/app-react.js>; rel=preload; as=script');
```

### 3. Service Worker Caching:
برای PWA می‌توانید فایل‌ها را pre-cache کنید:
```javascript
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('js-v1').then((cache) => {
      return cache.addAll([
        '/assets/js/runtime.js',
        '/assets/js/app-react.js',
        '/assets/js/app-vendor.js',
      ]);
    })
  );
});
```

---

## 🐛 عیب‌یابی (Troubleshooting)

### مشکل: فایل‌های جدید load نمی‌شوند

**راه‌حل 1:** Cache WordPress را پاک کنید
```bash
wp cache flush
```

**راه‌حل 2:** Browser cache را پاک کنید
```
Ctrl + Shift + Delete
```

**راه‌حل 3:** بررسی dependencies در Console:
```javascript
// در Console مرورگر:
console.log(window.React);        // باید object باشد
console.log(window.Highcharts);   // باید object باشد
```

### مشکل: "Uncaught ReferenceError: React is not defined"

**علت:** ترتیب بارگذاری اشتباه است

**راه‌حل:** بررسی dependencies در `wp_enqueue_script`:
```php
// باید app-react قبل از app-coins load شود
wp_enqueue_script('app-react', '...', array('runtime'));
wp_enqueue_script('app-coins', '...', array('runtime', 'app-react'));
```

### مشکل: Warning "asset exceeds recommended size limit"

این warning عادی است برای `app-highcharts.js` (726 KB).

Highcharts کتابخانه سنگینی است اما:
- ✅ فقط در صفحات coin load می‌شود
- ✅ با gzip به ~180 KB می‌رسد
- ✅ در browser cache می‌ماند

---

## 📚 منابع مرتبط

- [Webpack Code Splitting](https://webpack.js.org/guides/code-splitting/)
- [TerserWebpackPlugin](https://webpack.js.org/plugins/terser-webpack-plugin/)
- [Reduce JavaScript Execution Time](https://web.dev/bootup-time/)
- [Optimize Long Tasks](https://web.dev/optimize-long-tasks/)

---

## 📊 خلاصه

### ✅ انجام شد:
1. ✅ نصب `terser-webpack-plugin`
2. ✅ پیکربندی Terser با تنظیمات بهینه
3. ✅ Code splitting به 8 bundle جداگانه
4. ✅ بروزرسانی `Assets.php` با dependencies صحیح
5. ✅ بروزرسانی `PageSpeedController.php`
6. ✅ Build موفق webpack

### 📈 نتایج مورد انتظار:
- ✅ کاهش 45% در JS execution time (2,743 ms → ~1,500 ms)
- ✅ کاهش 60% در parse time برای app-vendor
- ✅ بهبود browser caching
- ✅ کاهش main-thread blocking

### 🎯 بعدی:
- تست در PageSpeed Insights
- بررسی Real User Monitoring (RUM)
- بهینه‌سازی بیشتر در صورت نیاز

---

**📅 آخرین بروزرسانی:** 8 دسامبر 2025  
**✍️ نویسنده:** XPay Development Team  
**🎯 نسخه:** 2.2.0
