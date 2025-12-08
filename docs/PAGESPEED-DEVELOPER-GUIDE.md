# راهنمای توسعه دهندگان - PageSpeed System

این مستند راهنمای جامعی برای توسعه دهندگان جهت کار با سیستم بهینه‌سازی PageSpeed است.

---

## 📚 فهرست مطالب

1. [معرفی سیستم](#معرفی-سیستم)
2. [اضافه کردن CSS جدید](#اضافه-کردن-css-جدید)
3. [اضافه کردن JavaScript جدید](#اضافه-کردن-javascript-جدید)
4. [پیکربندی PageSpeedController](#پیکربندی-pagespeedcontroller)
5. [Automatic Asset Detection](#automatic-asset-detection)
6. [Best Practices](#best-practices)
7. [Troubleshooting](#troubleshooting)

---

## معرفی سیستم

سیستم PageSpeed شامل **سه لایه اصلی** است:

### 1. Asset Management Layer (`app/Support/Assets.php`)
- مدیریت enqueue کردن CSS/JS
- Asset versioning
- Conditional loading

### 2. PageSpeed Controller (`app/Admin/PageSpeedController.php`)
- Critical CSS inline
- Async/Defer optimization
- Resource hints
- Page loader

### 3. Admin Panel (`app/Admin/PageSpeedAdmin.php`)
- تنظیمات از پنل ادمین
- AJAX save/load
- Real-time preview

---

## اضافه کردن CSS جدید

### روش 1: Manual Registration (فعلی)

```php
// app/Support/Assets.php

public function enqueueStyles(): void
{
    $manager = AssetVersionManager::getInstance();

    // 1. Register your CSS
    wp_enqueue_style(
        'my-custom-style',                           // Handle (unique ID)
        TEMPLATE_DIR . '/assets/css/my-style.css',   // URL
        array(),                                      // Dependencies
        ASSET_MAIN_VERSION,                          // Version (DON'T use null!)
        false                                         // Load in <head>
    );
}
```

### روش 2: با Minification

```php
// Use AssetVersionManager for automatic minification
wp_enqueue_style(
    'my-style',
    $manager->getUrl('my-style', 'css'),  // Auto creates my-style.v4.8.9.css
    array(),
    null,  // Manager handles versioning
    false
);
```

### Critical vs Non-Critical CSS

#### Critical CSS (Load synchronously):
```php
// باید در initial render لود شود
$critical_styles = [
    'main-style',
    'price-table-style',
    'header-style'
];

wp_enqueue_style('main-style', $url, [], ASSET_MAIN_VERSION, false);
```

#### Non-Critical CSS (Load async):
```php
// app/Admin/PageSpeedController.php
private function getNonCriticalStyles(): array
{
    return [
        'block-library-style',
        'comments-style',
        'popup-noties-style',
        'my-new-style'  // ⬅️ اضافه کنید اینجا
    ];
}
```

### Conditional CSS (Page-specific)

```php
// فقط در صفحه About
if (is_page(30)) {
    wp_enqueue_style(
        'about-style',
        TEMPLATE_DIR . '/assets/css/about.css',
        array(),
        ASSET_MAIN_VERSION,
        false
    );
}

// فقط در صفحات Blog
if (is_post_type_archive('blog')) {
    wp_enqueue_style(
        'blog-style',
        $manager->getUrl('blog', 'css'),
        array(),
        null,
        false
    );
}
```

---

## اضافه کردن JavaScript جدید

### روش 1: Manual Registration

```php
// app/Support/Assets.php

public function enqueueScripts(): void
{
    $manager = AssetVersionManager::getInstance();

    // 1. Register your JavaScript
    wp_enqueue_script(
        'my-custom-script',                          // Handle
        TEMPLATE_DIR . '/assets/js/my-script.js',    // URL
        ['jquery'],                                   // Dependencies (empty array if none)
        ASSET_MAIN_VERSION,                          // Version
        true                                          // Load in footer (true = footer)
    );
}
```

### روش 2: با Minification

```php
wp_enqueue_script(
    'my-script',
    $manager->getUrl('my-script', 'js'),  // Auto creates my-script.v4.8.9.js
    ['jquery'],
    null,  // Manager handles versioning
    true
);
```

### Script Loading Strategies

#### 1. Header Sync (Blocking - فقط برای critical scripts)
```php
$header_sync_scripts = [
    'jquery',
    'jquery-core',
    'coin-filter-script'  // Critical for homepage
];

wp_enqueue_script(
    'coin-filter-script',
    $manager->getUrl('coin-filter', 'js'),
    ['jquery'],
    null,
    false  // ⬅️ false = header
);
```

#### 2. Async (Non-blocking, Execute ASAP)
```php
// app/Admin/PageSpeedController.php
private function getAsyncScripts(): array
{
    return [
        'google-analytics',
        'gtag',
        'my-tracking-script'  // ⬅️ اضافه کنید اینجا
    ];
}

// در Assets.php:
wp_enqueue_script('my-tracking-script', $url, [], ASSET_MAIN_VERSION, true);
```

**نتیجه:**
```html
<script async src="/assets/js/my-tracking-script.js?ver=4.8.9"></script>
```

#### 3. Defer (Execute after DOM ready)
```php
// app/Admin/PageSpeedController.php
private function getDeferScripts(): array
{
    return [
        'coin-search-script',
        'bottom-sheet-script',
        'my-interactive-script'  // ⬅️ اضافه کنید اینجا
    ];
}
```

**نتیجه:**
```html
<script defer src="/assets/js/my-interactive-script.js?ver=4.8.9"></script>
```

### Conditional JavaScript

```php
// Homepage فقط
if (is_home() || is_front_page()) {
    wp_enqueue_script('homepage-script', $url, ['jquery'], ASSET_MAIN_VERSION, true);
}

// Single posts فقط
if (is_single()) {
    wp_enqueue_script('single-post-script', $url, [], ASSET_MAIN_VERSION, true);
}

// Coin archive فقط
if (is_post_type_archive('coin')) {
    wp_enqueue_script('coin-archive-script', $url, ['jquery'], ASSET_MAIN_VERSION, true);
}
```

---

## پیکربندی PageSpeedController

### ساختار فایل

```php
class PageSpeedController
{
    // 1. تنظیمات
    private function getSettings(): array { }
    
    // 2. Critical CSS
    private function getCriticalCss(): string { }
    
    // 3. Script Categories
    private function getAsyncScripts(): array { }
    private function getDeferScripts(): array { }
    
    // 4. Style Categories
    private function getNonCriticalStyles(): array { }
    
    // 5. Resource Hints
    private function getCriticalFonts(): array { }
    private function getExternalDomains(): array { }
}
```

### اضافه کردن Critical CSS

```php
// app/Admin/PageSpeedController.php - Line ~600

private function getCriticalCss(): string
{
    $settings = $this->getSettings();
    
    if (!empty($settings['critical_css'])) {
        return $settings['critical_css'];
    }

    // Default critical CSS
    return <<<CSS
    /* Your critical CSS here */
    body { margin: 0; padding: 0; }
    .main-header { /* ... */ }
    
    /* اضافه کردن CSS جدید */
    .my-critical-component {
        display: flex;
        /* ... */
    }
CSS;
}
```

### اضافه کردن External Domains برای DNS Prefetch

```php
private function getExternalDomains(): array
{
    return [
        'https://www.googletagmanager.com',
        'https://www.google-analytics.com',
        'https://fonts.googleapis.com',
        'https://fonts.gstatic.com',
        'https://cdn.example.com',  // ⬅️ اضافه کنید اینجا
    ];
}
```

**نتیجه:**
```html
<link rel="dns-prefetch" href="https://cdn.example.com">
```

### اضافه کردن Font Preload

```php
private function getCriticalFonts(): array
{
    return [
        [
            'url' => TEMPLATE_DIR . '/assets/fonts/dot4/woff2/IRANSansX-RegularD4.woff2',
            'type' => 'woff2'
        ],
        [
            'url' => 'https://cdn.example.com/myfont.woff2',  // ⬅️ اضافه کنید
            'type' => 'woff2'
        ]
    ];
}
```

**نتیجه:**
```html
<link rel="preload" href="..." as="font" type="font/woff2" crossorigin>
```

---

## Automatic Asset Detection

### مشکل فعلی:
هر بار که CSS/JS جدید اضافه می‌شود، باید دستی در چندین جا تغییر دهیم.

### راه‌حل: Dynamic Asset Scanner

```php
// app/Admin/PageSpeedController.php - اضافه کنید این متد را

/**
 * Automatically detect and categorize assets
 * 
 * @return array ['critical' => [], 'non_critical' => [], 'async' => [], 'defer' => []]
 */
private function detectAssets(): array
{
    global $wp_styles, $wp_scripts;
    
    $detected = [
        'critical_styles' => [],
        'non_critical_styles' => [],
        'async_scripts' => [],
        'defer_scripts' => []
    ];
    
    // Scan registered stylesheets
    if (!empty($wp_styles->registered)) {
        foreach ($wp_styles->registered as $handle => $style) {
            // Check if it's a theme asset
            if (strpos($style->src, get_template_directory_uri()) !== false) {
                
                // Categorize based on filename patterns
                if (preg_match('/(critical|main|header|above-fold)/i', $handle)) {
                    $detected['critical_styles'][] = $handle;
                } else {
                    $detected['non_critical_styles'][] = $handle;
                }
            }
        }
    }
    
    // Scan registered scripts
    if (!empty($wp_scripts->registered)) {
        foreach ($wp_scripts->registered as $handle => $script) {
            if (strpos($script->src, get_template_directory_uri()) !== false) {
                
                // Categorize based on filename patterns
                if (preg_match('/(analytics|tracking|gtag)/i', $handle)) {
                    $detected['async_scripts'][] = $handle;
                } 
                elseif (preg_match('/(search|sheet|modal|popup)/i', $handle)) {
                    $detected['defer_scripts'][] = $handle;
                }
            }
        }
    }
    
    return $detected;
}

/**
 * Use detected assets (call in __construct)
 */
private function applyDetectedAssets(): void
{
    $detected = $this->detectAssets();
    
    // Merge with manual configuration
    $this->criticalStyles = array_merge(
        $this->getCriticalStyles(),
        $detected['critical_styles']
    );
    
    $this->nonCriticalStyles = array_merge(
        $this->getNonCriticalStyles(),
        $detected['non_critical_styles']
    );
    
    // Similar for scripts...
}
```

### استفاده:

```php
public function __construct()
{
    // ... existing code ...
    
    // Auto-detect and apply
    add_action('wp_enqueue_scripts', [$this, 'applyDetectedAssets'], 999);
}
```

### Naming Conventions برای Auto-Detection

```
Critical Assets (Load immediately):
- *-critical.css
- *-above-fold.css
- main.css
- header.css

Non-Critical Assets (Async load):
- *-below-fold.css
- comments.css
- popup.css
- modal.css

Async Scripts:
- *-analytics.js
- *-tracking.js
- gtag.js

Defer Scripts:
- *-search.js
- *-sheet.js
- *-modal.js
```

---

## Best Practices

### 1. Asset Versioning
```php
// ❌ BAD: Cache issues
wp_enqueue_style('style', $url, [], null, false);
wp_enqueue_style('style', $url, [], '1.0', false);

// ✅ GOOD: Auto cache-bust
wp_enqueue_style('style', $url, [], ASSET_MAIN_VERSION, false);
```

### 2. Dependencies
```php
// ❌ BAD: No dependencies declared
wp_enqueue_script('my-script', $url, [], ASSET_MAIN_VERSION, true);
// If script uses jQuery, it might fail!

// ✅ GOOD: Declare dependencies
wp_enqueue_script('my-script', $url, ['jquery'], ASSET_MAIN_VERSION, true);
```

### 3. Conditional Loading
```php
// ❌ BAD: Load everywhere
wp_enqueue_style('blog-style', $url, [], ASSET_MAIN_VERSION, false);

// ✅ GOOD: Load only where needed
if (is_post_type_archive('blog')) {
    wp_enqueue_style('blog-style', $url, [], ASSET_MAIN_VERSION, false);
}
```

### 4. Critical CSS
```php
// ❌ BAD: Large CSS inline
// Don't inline 100KB+ CSS!

// ✅ GOOD: Only above-the-fold styles
// Inline 10-20KB max
// Rest loads async
```

### 5. Third-Party Scripts
```php
// ❌ BAD: Blocking third-party
<script src="https://external.com/script.js"></script>

// ✅ GOOD: Async/Defer or delayed
<script async src="https://external.com/script.js"></script>

// ✅ BETTER: Load on interaction (like GTM)
window.addEventListener('scroll', loadThirdParty, { once: true });
```

---

## Troubleshooting

### مشکل 1: CSS به درستی لود نمی‌شود

**علت:** Asset minified ساخته نشده

**راه‌حل:**
```bash
# حذف فایل‌های minified
Remove-Item c:\Docker\xpay\wordpress\wp-content\themes\xpay_main_theme\assets\css\*.v*.css

# رفرش صفحه → AssetVersionManager خودش می‌سازد
```

### مشکل 2: JavaScript error "jQuery is not defined"

**علت:** Dependency order اشتباه است

**راه‌حل:**
```php
// اطمینان حاصل کنید jquery در dependencies است
wp_enqueue_script(
    'my-script',
    $url,
    ['jquery'],  // ⬅️ این را فراموش نکنید!
    ASSET_MAIN_VERSION,
    true
);
```

### مشکل 3: Forced Reflow Warning

**علت:** DOM geometry property قبل از paint

**راه‌حل:**
```javascript
// ❌ BAD
const width = element.offsetWidth;
element.style.width = width + 'px';

// ✅ GOOD
requestAnimationFrame(() => {
    const width = element.offsetWidth;
    element.style.width = width + 'px';
});
```

### مشکل 4: Cache not updating

**علت:** Browser cache یا CDN cache

**راه‌حل:**
```php
// 1. Increment ASSET_MAIN_VERSION
define('ASSET_MAIN_VERSION', '4.9.0'); // was 4.8.9

// 2. Hard refresh
// Ctrl + Shift + R (Windows)
// Cmd + Shift + R (Mac)

// 3. Clear server cache
// به بسته caching بستگی دارد
```

### مشکل 5: PageSpeed score not improving

**چک‌لیست:**
```
□ Critical CSS inline شده؟
□ Non-critical CSS async است?
□ Scripts defer/async هستند؟
□ Images lazy load می‌شوند؟
□ Third-party scripts delayed هستند؟
□ Asset versioning فعال است؟
□ Gzip/Brotli compression فعال است؟
□ Server response time < 200ms است؟
```

---

## مثال کامل: اضافه کردن Feature جدید

فرض کنید می‌خواهیم یک **Product Comparison** feature اضافه کنیم.

### Step 1: Create Assets

```css
/* assets/css/product-comparison.css */
.product-comparison {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
}

.comparison-card {
    border: 1px solid #ddd;
    padding: 20px;
}
```

```javascript
/* assets/js/product-comparison.js */
document.addEventListener('DOMContentLoaded', function() {
    const comparisons = document.querySelectorAll('.product-comparison');
    
    comparisons.forEach(function(comp) {
        // Your logic here
    });
});
```

### Step 2: Register in Assets.php

```php
// app/Support/Assets.php

public function enqueueStyles(): void
{
    // ... existing code ...
    
    // Conditional: only on product pages
    if (is_singular('product') || is_page('compare')) {
        wp_enqueue_style(
            'product-comparison-style',
            TEMPLATE_DIR . '/assets/css/product-comparison.css',
            array(), // No dependencies
            ASSET_MAIN_VERSION,
            false // Load in header
        );
    }
}

public function enqueueScripts(): void
{
    // ... existing code ...
    
    if (is_singular('product') || is_page('compare')) {
        $manager = AssetVersionManager::getInstance();
        
        wp_enqueue_script(
            'product-comparison-script',
            $manager->getUrl('product-comparison', 'js'),
            ['jquery'], // Depends on jQuery
            null, // Manager handles version
            true // Load in footer
        );
    }
}
```

### Step 3: Configure PageSpeedController

```php
// app/Admin/PageSpeedController.php

// اگر non-critical است:
private function getNonCriticalStyles(): array
{
    return [
        'block-library-style',
        'comments-style',
        'popup-noties-style',
        'product-comparison-style'  // ⬅️ Async load
    ];
}

// اگر defer می‌خواهید:
private function getDeferScripts(): array
{
    return [
        'coin-search-script',
        'bottom-sheet-script',
        'product-comparison-script'  // ⬅️ Defer load
    ];
}
```

### Step 4: Test

```bash
# 1. Clear cache
Remove-Item assets/**/*.v*.{css,js}

# 2. Hard refresh
# Ctrl + Shift + R

# 3. Check PageSpeed Insights
# https://pagespeed.web.dev/
```

---

## Advanced: Custom Loading Strategy

### Lazy Load Module on Demand

```javascript
// assets/js/lazy-loader.js

window.LazyLoader = {
    loaded: {},
    
    loadCSS: function(url, callback) {
        if (this.loaded[url]) {
            callback && callback();
            return;
        }
        
        var link = document.createElement('link');
        link.rel = 'stylesheet';
        link.href = url;
        link.onload = function() {
            window.LazyLoader.loaded[url] = true;
            callback && callback();
        };
        document.head.appendChild(link);
    },
    
    loadJS: function(url, callback) {
        if (this.loaded[url]) {
            callback && callback();
            return;
        }
        
        var script = document.createElement('script');
        script.src = url;
        script.onload = function() {
            window.LazyLoader.loaded[url] = true;
            callback && callback();
        };
        document.body.appendChild(script);
    },
    
    loadModule: function(moduleName) {
        var baseUrl = '/wp-content/themes/xpay_main_theme/assets';
        
        this.loadCSS(baseUrl + '/css/' + moduleName + '.css');
        this.loadJS(baseUrl + '/js/' + moduleName + '.js', function() {
            console.log('Module ' + moduleName + ' loaded');
        });
    }
};

// استفاده:
document.querySelector('.load-comparison').addEventListener('click', function() {
    window.LazyLoader.loadModule('product-comparison');
});
```

---

## 🔧 جلوگیری از Forced Reflows

### مشکلات رایج و راه حل‌ها

#### 1. Swiper.js Initialization
**❌ اشتباه:**
```javascript
var mySwiper = new Swiper('.swiper-container', {
    slidesPerView: 3,
    spaceBetween: 30
});
```

**✅ صحیح:**
```javascript
requestAnimationFrame(() => {
    var mySwiper = new Swiper('.swiper-container', {
        slidesPerView: 3,
        spaceBetween: 30
    });
});
```

**دلیل:** Swiper در initialization نیاز به اندازه‌گیری DOM دارد که باعث reflow می‌شود.

---

#### 2. jQuery Geometry Methods
**❌ اشتباه:**
```javascript
$element.on('click', function(e) {
    var rect = this.getBoundingClientRect();
    var x = e.clientX - rect.left; // Forced reflow!
});
```

**✅ صحیح:**
```javascript
$element.on('click', function(e) {
    var clientX = e.clientX; // Cache event property
    var element = this;
    
    requestAnimationFrame(() => {
        var rect = element.getBoundingClientRect();
        var x = clientX - rect.left; // Safe!
    });
});
```

**دلیل:** `getBoundingClientRect()` باعث forced reflow می‌شود.

---

#### 3. Third-Party Scripts (GTM, Najva, etc.)
**❌ اشتباه:**
```javascript
// مستقیم در <head>
<script src="https://third-party.com/script.js"></script>
```

**✅ صحیح:**
```javascript
// با requestIdleCallback
window.loadThirdPartyScript = function(src, attributes) {
    var load = function() {
        var script = document.createElement('script');
        script.src = src;
        script.async = true;
        script.defer = true;
        // ...
        document.head.appendChild(script);
    };
    
    if ('requestIdleCallback' in window) {
        requestIdleCallback(load, { timeout: 2000 });
    } else {
        setTimeout(load, 100);
    }
};

// استفاده:
window.addEventListener('load', function() {
    window.loadThirdPartyScript('https://third-party.com/script.js', {
        'id': 'my-script'
    });
});
```

**دلیل:** Third-party scripts اغلب DOM manipulation دارند که باعث reflow می‌شوند.

---

#### 4. Scroll Event Handlers
**❌ اشتباه:**
```javascript
window.addEventListener('scroll', function() {
    var scrollTop = window.pageYOffset; // Forced reflow on every scroll!
    // Do something...
});
```

**✅ صحیح:**
```javascript
var ticking = false;

window.addEventListener('scroll', function() {
    if (!ticking) {
        requestAnimationFrame(() => {
            var scrollTop = window.pageYOffset;
            // Do something...
            ticking = false;
        });
        ticking = true;
    }
}, { passive: true });
```

**دلیل:** Scroll events خیلی سریع trigger می‌شوند و باید throttle شوند.

---

### Performance Utilities در XPay

```javascript
// app.js - Lines 1-28
const PerformanceUtils = {
    scheduleRead: function(callback) {
        requestAnimationFrame(() => callback());
    },
    
    scheduleWrite: function(callback) {
        requestAnimationFrame(() => {
            requestAnimationFrame(() => callback());
        });
    },
    
    triggerReflow: function(element) {
        requestAnimationFrame(() => {
            void element.offsetWidth;
        });
    }
};

window.PerformanceUtils = PerformanceUtils;

// jQuery Safe Methods (Lines 30-50)
if (typeof jQuery !== 'undefined') {
    jQuery.fn.offsetSafe = function() {
        let result;
        requestAnimationFrame(() => {
            result = jQuery.fn.offset.call(this);
        });
        return result;
    };
}
```

**استفاده:**
```javascript
// Read DOM
PerformanceUtils.scheduleRead(() => {
    var height = element.offsetHeight;
    console.log('Height:', height);
});

// Write to DOM
PerformanceUtils.scheduleWrite(() => {
    element.style.width = '100px';
});
```

---

## Checklist برای Developer

هر بار که asset جدید اضافه می‌کنید:

- [ ] نام فایل مطابق naming convention است؟
- [ ] در `Assets.php` register شده؟
- [ ] `ASSET_MAIN_VERSION` برای versioning استفاده شده؟
- [ ] Dependencies درست تعیین شده؟
- [ ] Conditional loading برای صفحات خاص؟
- [ ] در `PageSpeedController` طبقه‌بندی شده؟ (critical/non-critical/async/defer)
- [ ] فایل minified ساخته شده؟
- [ ] PageSpeed test انجام شده؟
- [ ] Forced reflow check شده؟ (Chrome DevTools Performance)
- [ ] Swiper initialization با `requestAnimationFrame` wrap شده؟
- [ ] `getBoundingClientRect()` داخل event handler با `requestAnimationFrame` wrap شده؟
- [ ] Third-party scripts با `requestIdleCallback` یا `loadThirdPartyScript` لود می‌شوند؟
- [ ] Scroll handlers throttled شده‌اند؟
- [ ] Browser compatibility test شده؟

---

## منابع بیشتر

- [PageSpeed Insights](https://pagespeed.web.dev/)
- [Web.dev Performance](https://web.dev/performance/)
- [MDN: requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)
- [MDN: requestIdleCallback](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback)
- [Avoiding Forced Reflows](https://web.dev/avoid-large-complex-layouts-and-layout-thrashing/)
- [Critical Rendering Path](https://web.dev/critical-rendering-path/)
- [Resource Hints](https://www.w3.org/TR/resource-hints/)

---

**آخرین به‌روزرسانی:** 10 نوامبر 2025  
**نسخه:** 4.9.2  
**نگهدارنده:** XPay Development Team

**سوالات؟** Issue باز کنید یا با تیم توسعه تماس بگیرید.
