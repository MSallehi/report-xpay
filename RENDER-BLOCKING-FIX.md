# 🚀 رفع Render Blocking Resources

> **تاریخ:** ژانویه 2025  
> **نوع:** Performance Optimization  
> **بهبود:** ~3,830ms کاهش در render-blocking time

---

## 📋 مشکل

PageSpeed Insights گزارش می‌داد که منابع زیادی render را block می‌کنند:

```
Render blocking requests — Est savings of 3,830 ms

| فایل | حجم | زمان |
|------|-----|------|
| app-vendor.js | 298.2 KiB | 2,680 ms |
| jquery.min.js | 30.0 KiB | 1,910 ms |
| swiper.js | 40.5 KiB | 1,530 ms |
| performance-optimizer.js | 4.9 KiB | 1,150 ms |
| dom-interceptor.js | 4.8 KiB | 1,150 ms |
| reflow-optimizer.js | 2.6 KiB | 1,150 ms |
| app-coins.css | 1.0 KiB | 1,150 ms |
| inp-optimizer.js | 5.0 KiB | 380 ms |
| ... و سایر فایل‌ها |
```

### علت

فایل‌های JavaScript در `<head>` بدون `defer` یا `async` لود می‌شدند که باعث می‌شد:
1. Parser HTML متوقف شود
2. فایل‌ها دانلود و اجرا شوند
3. سپس rendering ادامه یابد

---

## ✅ راه‌حل

### تغییرات در `PageSpeedController.php`

#### 1. حذف jQuery از لیست sync scripts

```php
// قبل - jQuery sync بود و render را block می‌کرد
$header_sync_scripts = [
    'jquery',
    'jquery-core',
    'jquery-migrate',
    'coin-filter-script'
];

// بعد - لیست خالی شد
$header_sync_scripts = [
    // همه اسکریپت‌ها defer می‌شوند
];
```

#### 2. گسترش لیست defer scripts

```php
$defer_scripts = [
    // jQuery (deferred but maintains order)
    'jquery',                   // 30 KiB
    'jquery-core',
    'jquery-migrate',
    
    // Performance Optimization Scripts
    'dom-interceptor',          // DOM method override
    'reflow-optimizer',         // Reflow batching
    'performance-optimizer',    // LCP, CLS, TBT
    'inp-optimizer',            // INP optimization
    
    // Swiper (carousel)
    'swiper-script',            // 40.5 KiB
    'swiper-wrapper',
    
    // React/Vendor bundles (largest)
    'app-vendor',               // 298.2 KiB
    'app-coins',
    'app-calculator',  
    'app-chart',
    
    // Other scripts
    'coin-filter-script',
    'coin-search-script',
    'bottom-sheet-script',
    'view-more-coins-script',
    'main-script',
    'custom-coins',
    'popup-noties',
    'glightbox',
    'coin-zoom',
    'symbol-updater',
    'pako',
    'gift-box',
];
```

---

## 🔬 چرا `defer` ایمن است؟

### مزایای defer:

1. **Parallel Download**: فایل همزمان با parsing HTML دانلود می‌شود
2. **Execution Order**: ترتیب اجرا حفظ می‌شود (برخلاف async)
3. **DOM Ready**: اسکریپت بعد از parse شدن HTML اجرا می‌شود
4. **No Blocking**: Render را block نمی‌کند

### نمودار:

```
بدون defer (Render-Blocking):
HTML Parsing ----| STOP |---- Script Download ----| Script Execute |---- Continue Parsing

با defer (Non-Blocking):
HTML Parsing -----------------------> Parse Complete
         |                                |
         Script Download ------> Execute |
                                        ↓
                                   DOM Ready
```

### سازگاری با jQuery:

```javascript
// این کد با defer کار می‌کند ✅
$(document).ready(function() {
    // ...
});

// یا
jQuery(function($) {
    // ...
});

// یا
document.addEventListener('DOMContentLoaded', function() {
    // ...
});
```

---

## ⚠️ نکات مهم

### 1. CSS همچنان Critical است

`main.css` و `price-table.css` همچنان synchronous لود می‌شوند چون:
- CSS برای First Paint ضروری است
- FOUC (Flash of Unstyled Content) اتفاق می‌افتد اگر async شوند

### 2. ترتیب Dependencies حفظ می‌شود

`defer` برخلاف `async` ترتیب را حفظ می‌کند:
```
jquery → dom-interceptor → reflow-optimizer → app-vendor → app-coins
```

### 3. مقدار Default

اگر اسکریپتی در لیست نباشد، default defer می‌گیرد:
```php
// Default: defer all other scripts
if (strpos($tag, 'defer') === false && strpos($tag, 'async') === false) {
    return str_replace(' src', ' defer src', $tag);
}
```

---

## 📊 نتایج مورد انتظار

| معیار | قبل | بعد | بهبود |
|-------|------|------|--------|
| Render Blocking Time | ~3,830ms | ~380ms | **90%** |
| First Contentful Paint | ~2.5s | ~1.2s | **52%** |
| Largest Contentful Paint | ~4.0s | ~2.0s | **50%** |

---

## 📁 فایل‌های تغییر یافته

| فایل | تغییر |
|------|-------|
| `app/Admin/PageSpeedController.php` | گسترش defer list، حذف sync list |
| `.env` | ASSET_VERSION → 5.6.6 |

---

## 🧪 تست

### بررسی defer در DevTools:

1. Chrome DevTools → Network tab
2. فیلتر: JS
3. بررسی کنید که همه فایل‌ها `defer` دارند
4. Initiator نباید Parser-blocking باشد

### PageSpeed Test:

1. بروید به https://pagespeed.web.dev/
2. URL صفحه را تست کنید
3. بخش "Eliminate render-blocking resources" را بررسی کنید

---

## 🔗 منابع

- [MDN: defer attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#defer)
- [web.dev: Render-blocking resources](https://web.dev/render-blocking-resources/)
- [Chrome: Script loading strategies](https://web.dev/efficiently-load-third-party-javascript/)
