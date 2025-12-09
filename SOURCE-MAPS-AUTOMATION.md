# 🗺️ Source Maps Automation - AssetVersionManager

## مستندات فنی ساخت خودکار Source Maps

تاریخ: 2025-12-09  
نسخه: 2.2.0  
نویسنده: XPay Development Team

---

## 📋 خلاصه

سیستم مدیریت خودکار source maps برای فایل‌های JavaScript که به صورت ورژن‌دار ساخته می‌شوند.

### مشکل قبلی
- Source maps باید به صورت دستی توسط اسکریپت `create-versioned-sourcemaps.js` ساخته شوند
- احتمال عدم همخوانی بین فایل JS و source map
- نیاز به اجرای دستی بعد از هر بیلد یا تغییر ورژن

### راه‌حل
- ادغام ساخت source maps در `AssetVersionManager.php`
- ساخت خودکار همزمان با فایل‌های ورژن‌دار
- به‌روزرسانی خودکار `sourceMappingURL` در فایل JS

---

## 🏗️ معماری سیستم

### کلاس `AssetVersionManager`

فایل: `app/Support/AssetVersionManager.php`

#### متد جدید: `createVersionedSourceMap()`

```php
private function createVersionedSourceMap(array $asset, ?string $version, string $versionedPath): void
{
    // بررسی وجود source map برای فایل JS
    $sourceMapPath = $asset['sourcePath'] . '.map';
    
    if (!file_exists($sourceMapPath)) {
        return;
    }
    
    // ساخت نام فایل ورژن‌دار .map
    $versionedMapFile = $version 
        ? "{$asset['filePrefix']}.v{$version}.{$asset['extension']}.map" 
        : "{$asset['filePrefix']}.{$asset['extension']}.map";
    
    $versionedMapPath = "{$asset['baseDir']}/{$versionedMapFile}";
    
    // کپی source map
    if (!file_exists($versionedMapPath)) {
        @copy($sourceMapPath, $versionedMapPath);
    }
    
    // به‌روزرسانی sourceMappingURL در فایل JS
    if (file_exists($versionedPath)) {
        $jsContent = file_get_contents($versionedPath);
        
        if ($jsContent !== false) {
            // Find and replace sourceMappingURL
            $oldMapRef = "//# sourceMappingURL={$asset['filePrefix']}.{$asset['extension']}.map";
            $newMapRef = "//# sourceMappingURL={$versionedMapFile}";
            
            $jsContent = str_replace($oldMapRef, $newMapRef, $jsContent);
            @file_put_contents($versionedPath, $jsContent);
        }
    }
}
```

#### ادغام در متد `processAll()`

```php
public function processAll()
{
    foreach ($this->assets as &$asset) {
        $version = $asset['debug'] ? null : $this->getVersion($asset);
        
        // ... ساخت فایل ورژن‌دار ...
        
        // اضافه شده: ساخت source map برای JS
        if ($asset['extension'] === 'js') {
            $this->createVersionedSourceMap($asset, $version, $versionedPath);
        }
    }
}
```

---

## 🔄 چرخه کامل کار

### 1. بارگیری صفحه توسط کاربر

```
کاربر URL را باز می‌کند
    ↓
WordPress theme load می‌شود
    ↓
AssetVersionManager::processAll() فراخوانی می‌شود
    ↓
ورژن از .env خوانده می‌شود: ASSET_VERSION=5.5.8
```

### 2. بررسی و ساخت فایل‌ها

```
برای هر asset (مثلا app-vendor.js):
    ↓
بررسی: app-vendor.v5.5.8.js وجود دارد؟
    ↓
    ├─ بله → استفاده از فایل موجود
    └─ خیر → ساخت فایل‌های جدید:
              1. کپی/minify: app-vendor.js → app-vendor.v5.5.8.js
              2. فراخوانی createVersionedSourceMap():
                 ├─ بررسی: app-vendor.js.map وجود دارد؟
                 │   ├─ بله → کپی به app-vendor.v5.5.8.js.map
                 │   └─ خیر → skip
                 └─ به‌روزرسانی sourceMappingURL در app-vendor.v5.5.8.js
```

### 3. پاکسازی کش از پنل ادمین

```
ادمین وارد WordPress Admin می‌شود
    ↓
منوی Xpay Management > Browser Cache
    ↓
کلیک روی دکمه "Clear Browser Cache"
    ↓
AJAX request به wp_ajax_browser_cache_clear
    ↓
BrowserCacheAdmin::incrementCacheVersion()
    ↓
ورژن در .env به‌روز می‌شود: 5.5.8 → 5.5.9
    ↓
deleteVersionedAssets() فراخوانی می‌شود:
    ├─ حذف فایل‌های *.v5.5.8.js
    ├─ حذف فایل‌های *.v5.5.8.js.map
    └─ حذف فایل‌های *.v5.5.8.css و *.v5.5.8.css.map
    ↓
regenerateVersionedSourceMaps() فراخوانی می‌شود:
    └─ برای 8 webpack bundle:
        ├─ کپی base JS به versioned JS
        ├─ کپی base .map به versioned .map
        └─ به‌روزرسانی sourceMappingURL
```

---

## 📦 Webpack Bundles (8 فایل)

لیست کامل bundle هایی که source map دارند:

| Bundle | حجم JS | حجم Map | توضیحات |
|--------|--------|---------|---------|
| `runtime.js` | 3.66 KB | 18.68 KB | Webpack runtime |
| `app-react.js` | 132.69 KB | 332.01 KB | React + ReactDOM |
| `app-highcharts.js` | 723.25 KB | 1.7 MB | Highcharts Stock + Boost |
| `app-datetime.js` | 307.88 KB | 998.15 KB | dayjs + jalaliday |
| `app-vendor.js` | 130.25 KB | 656.80 KB | axios, pako و سایر کتابخانه‌ها |
| `app-chart.js` | 25.55 KB | 50.30 KB | Chart component |
| `app-calculator.js` | 21.32 KB | 57.67 KB | Calculator component |
| `app-coins.js` | 1.48 KB | 4.68 KB | Entry point |

**مجموع:**
- JS: ~1.35 MB (minified)
- Source Maps: ~3.8 MB (uncompressed)

---

## 🔧 ادغام با BrowserCacheAdmin

### متد `regenerateVersionedSourceMaps()`

```php
private static function regenerateVersionedSourceMaps(): void
{
    $version = Env::get('ASSET_VERSION', '1.0.0');
    $jsDir = get_template_directory() . '/assets/js';
    
    // لیست به‌روز شده: 8 webpack bundle
    $webpackBundles = [
        'runtime',          // جدید (code splitting)
        'app-react',        // جدید (code splitting)
        'app-highcharts',   // جدید (code splitting)
        'app-datetime',     // جدید (code splitting)
        'app-vendor',       // قبلی
        'app-calculator',   // قبلی
        'app-chart',        // قبلی
        'app-coins'         // قبلی
    ];
    
    foreach ($webpackBundles as $bundle) {
        // کپی JS
        $baseJs = "{$jsDir}/{$bundle}.js";
        $versionedJs = "{$jsDir}/{$bundle}.v{$version}.js";
        @copy($baseJs, $versionedJs);
        
        // کپی و به‌روزرسانی source map
        $baseMap = "{$jsDir}/{$bundle}.js.map";
        if (file_exists($baseMap)) {
            $versionedMap = "{$jsDir}/{$bundle}.v{$version}.js.map";
            @copy($baseMap, $versionedMap);
            
            // به‌روزرسانی sourceMappingURL در JS
            $jsContent = file_get_contents($versionedJs);
            if ($jsContent !== false) {
                $oldRef = "//# sourceMappingURL={$bundle}.js.map";
                $newRef = "//# sourceMappingURL={$bundle}.v{$version}.js.map";
                $jsContent = str_replace($oldRef, $newRef, $jsContent);
                @file_put_contents($versionedJs, $jsContent);
            }
        }
    }
}
```

---

## 🧪 تست و اعتبارسنجی

### تست خودکار (Browser)

وقتی صفحه بارگیری می‌شود، اسکریپت زیر در footer اجرا می‌شود:

```javascript
// از BrowserCacheAdmin::injectCacheScript()
var currentVersion = '5.5.8';
var storedVersion = localStorage.getItem('xpay_asset_version');

if (storedVersion && storedVersion !== currentVersion) {
    console.log('XPay: Clearing browser cache (version: ' + 
                storedVersion + ' → ' + currentVersion + ')');
    
    // پاک کردن cache
    // پاک کردن localStorage
    // Reload صفحه
}
```

### تست دستی

#### 1. بررسی ساخت source map

```bash
# لیست فایل‌های ورژن‌دار
Get-ChildItem assets/js -Filter "*.v5.5.8.*" | Select Name, Length

# باید 16 فایل نمایش دهد (8 JS + 8 .map)
```

#### 2. بررسی sourceMappingURL

```bash
# بررسی محتوای فایل JS
Get-Content assets/js/app-vendor.v5.5.8.js | Select-String "sourceMappingURL"

# خروجی مورد انتظار:
# //# sourceMappingURL=app-vendor.v5.5.8.js.map
```

#### 3. تست در Browser DevTools

1. باز کردن Chrome DevTools (F12)
2. رفتن به Sources tab
3. باز کردن فایل `app-vendor.v5.5.8.js`
4. بررسی اینکه کد original نمایش داده می‌شود (نه minified)
5. قرار دادن breakpoint و debug کردن

---

## 📊 مقایسه قبل/بعد

### قبل (Manual)

```
بیلد webpack
    ↓
اجرای دستی: node create-versioned-sourcemaps.js
    ↓
commit + push
    ↓
deploy

⚠️ مشکلات:
- نیاز به اجرای دستی
- احتمال فراموشی
- احتمال عدم همخوانی
```

### بعد (Automatic)

```
بیلد webpack
    ↓
commit + push
    ↓
deploy
    ↓
کاربر صفحه را باز می‌کند
    ↓
AssetVersionManager خودکار source maps می‌سازد

✅ مزایا:
- کاملا خودکار
- همیشه همخوان
- بدون نیاز به اسکریپت دستی
```

---

## ⚙️ پیکربندی

### .env

```env
# نسخه asset ها - خوانده می‌شود توسط AssetVersionManager
ASSET_VERSION=5.5.8

# Debug mode - اگر true باشد، فایل‌های ورژن‌دار ساخته نمی‌شوند
APP_DEBUG=false
```

### Assets.php

ثبت و enqueue کردن assets:

```php
// Runtime (باید اول لود شود)
wp_enqueue_script('runtime', 
    $assetVersionManager->getUrl('runtime', 'js'), 
    [], 
    null, 
    true
);

// Libraries
wp_enqueue_script('app-react', 
    $assetVersionManager->getUrl('app-react', 'js'), 
    ['runtime'], 
    null, 
    true
);

// و ادامه برای سایر bundle ها...
```

---

## 🚀 Deployment

### GitHub Actions Workflow

فایل: `.github/workflows/deploy.yml`

```yaml
exclude: |
  # فقط base files exclude می‌شوند
  assets/js/app-vendor.js
  assets/js/app-vendor.js.map
  assets/js/app-react.js
  assets/js/app-react.js.map
  # ... (همه 8 base file)
  
  # فایل‌های versioned deploy می‌شوند:
  # ✅ assets/js/*.v5.5.8.js
  # ✅ assets/js/*.v5.5.8.js.map
```

---

## 🔍 Troubleshooting

### مشکل: source map 404 می‌شود

**علت:** فایل ورژن‌دار ساخته نشده

**راه‌حل:**
```bash
# پاک کردن فایل‌های قدیمی
Remove-Item assets/js/*.v5.5.8.* -Force

# باز کردن صفحه - فایل‌ها خودکار ساخته می‌شوند
```

### مشکل: sourceMappingURL اشتباه است

**علت:** فایل JS به‌روز نشده

**راه‌حل:**
```bash
# بررسی محتوای فایل
Get-Content assets/js/app-vendor.v5.5.8.js | Select-String "sourceMappingURL"

# اگر اشتباه بود، فایل را پاک کنید تا دوباره ساخته شود
Remove-Item assets/js/app-vendor.v5.5.8.js
```

### مشکل: فایل .map خیلی بزرگ است

**پاسخ:** این طبیعی است! source maps شامل کد original هستند:
- `app-highcharts.js`: 723 KB (minified)
- `app-highcharts.js.map`: 1.7 MB (original source)

فایل‌های .map فقط وقتی دانلود می‌شوند که DevTools باز باشد.

---

## 📚 مراجع

- [Source Maps در MDN](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map)
- [Webpack devtool](https://webpack.js.org/configuration/devtool/)
- [JS-EXECUTION-OPTIMIZATION.md](./JS-EXECUTION-OPTIMIZATION.md)
- [CHANGELOG-FA.md](../changelog/CHANGELOG-FA.md)

---

## 📝 تاریخچه تغییرات

### v2.2.0 (2025-12-09)
- ✅ اضافه شدن `createVersionedSourceMap()` به AssetVersionManager
- ✅ به‌روزرسانی `regenerateVersionedSourceMaps()` برای 8 bundle
- ✅ ادغام در چرخه processAll
- ✅ تست کامل در development و staging

---

**تهیه شده توسط:** XPay Development Team  
**آخرین به‌روزرسانی:** 2025-12-09  
**نسخه مستند:** 1.0.0
