# 🔄 Cache Version Redirect Control

## نمای کلی (Overview)

این ویژگی امکان کنترل نحوه بارگذاری مجدد صفحه هنگام تغییر ورژن asset ها را فراهم می‌کند. می‌توانید انتخاب کنید که آیا می‌خواهید:
- با query string redirect شود (`?_v=5.5.14&_t=timestamp`)
- یا به صورت silent reload شود (بدون query string)

This feature provides control over how the page reloads when asset version changes. You can choose whether to:
- Redirect with query string (`?_v=5.5.14&_t=timestamp`)
- Or silently reload (without query string)

---

## ⚙️ تنظیمات (Settings)

### مکان تنظیمات (Settings Location)
- مسیر: **مدیریت Xpay → PageSpeed → Cache Version Redirect**
- Path: **Xpay Management → PageSpeed → Cache Version Redirect**

### گزینه‌های موجود (Available Options)

#### 1️⃣ Enable Cache Version Redirect ✅
**فعال (Enabled - پیش‌فرض)**

هنگام تغییر ورژن:
1. کاربر با query string به صفحه redirect می‌شود: `/?_v=5.5.14&_t=1766302160054`
2. این کار cache مرورگر را bypass می‌کند و asset های جدید دانلود می‌شوند
3. بلافاصله بعد از reload، URL تمیز می‌شود (query string پاک می‌شود)

**مزایا:**
- ✅ تضمین دریافت asset های جدید
- ✅ Cache مرورگر را force می‌کند که جدیدترین نسخه را بگیرد
- ✅ عدم مشکل با CDN و intermediate caches

**معایب:**
- ⚠️ کاربر برای لحظه‌ای query string را در URL می‌بیند

When version changes:
1. User is redirected with query string: `/?_v=5.5.14&_t=1766302160054`
2. This bypasses browser cache and downloads fresh assets
3. Immediately after reload, URL is cleaned (query string removed)

**Pros:**
- ✅ Guarantees fresh asset delivery
- ✅ Forces browser cache to fetch latest version
- ✅ No issues with CDN and intermediate caches

**Cons:**
- ⚠️ User briefly sees query string in URL

---

#### 2️⃣ Disable Cache Version Redirect ❌
**غیرفعال (Disabled)**

هنگام تغییر ورژن:
1. صفحه به صورت silent reload می‌شود: `location.reload(true)`
2. هیچ query string ای به URL اضافه نمی‌شود
3. مرورگر سعی می‌کند cache را bypass کند

**مزایا:**
- ✅ URL تمیز می‌ماند (بدون query string)
- ✅ تجربه کاربری بهتر (بدون تغییر URL)

**معایب:**
- ⚠️ ممکن است برخی مرورگرها همچنان asset های cached قدیمی را نمایش دهند
- ⚠️ CDN و intermediate caches ممکن است فایل‌های قدیمی را serve کنند
- ⚠️ نیاز به Clear Cache manual در برخی موارد

When version changes:
1. Page silently reloads: `location.reload(true)`
2. No query string is added to URL
3. Browser attempts to bypass cache

**Pros:**
- ✅ Clean URL (no query string)
- ✅ Better user experience (no URL change)

**Cons:**
- ⚠️ Some browsers may still show cached old assets
- ⚠️ CDN and intermediate caches may serve old files
- ⚠️ May require manual cache clearing in some cases

---

## 🔧 نحوه کار فنی (Technical Implementation)

### فلوچارت (Flowchart)

```
┌─────────────────────────┐
│  Page Load              │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Check localStorage      │
│ version: xpay_v         │
└──────────┬──────────────┘
           │
           ▼
     ┌────┴────┐
     │ Version │
     │ Changed?│
     └────┬────┘
          │
    ┌─────┴─────┐
    │           │
   Yes          No
    │           │
    ▼           ▼
┌───────┐   ┌────────┐
│ Clear │   │  Done  │
│ Cache │   └────────┘
└───┬───┘
    │
    ▼
┌───────────────────────────┐
│ Check Setting:            │
│ enable_cache_redirect     │
└─────────┬─────────────────┘
          │
    ┌─────┴─────┐
    │           │
  true         false
    │           │
    ▼           ▼
┌─────────┐  ┌──────────┐
│ Redirect│  │ Silent   │
│ with    │  │ Reload   │
│ Query   │  │ (no qs)  │
│ String  │  │          │
└────┬────┘  └──────────┘
     │
     ▼
┌─────────────────┐
│ URL with params │
│ ?_v=X&_t=Y      │
└────┬────────────┘
     │
     ▼
┌─────────────────┐
│ Detect params   │
│ on next load    │
└────┬────────────┘
     │
     ▼
┌─────────────────┐
│ Clean URL with  │
│ replaceState()  │
└─────────────────┘
```

### کد اصلی (Core Code)

```javascript
// در header.php
var enableRedirect = <?php echo $enable_cache_redirect ? 'true' : 'false'; ?>;

if (storedVersion !== currentVersion) {
    // Clear all caches
    localStorage.clear();
    sessionStorage.clear();
    
    if (enableRedirect) {
        // با query string redirect کن
        location.href = location.href.split('?')[0] + '?_v=' + v + '&_t=' + Date.now();
    } else {
        // بدون query string reload کن
        location.reload(true);
    }
}

// اگر query params وجود دارد، URL را تمیز کن
if (hasVersionParam) {
    localStorage.setItem('xpay_v', v);
    var cleanUrl = window.location.protocol + '//' + window.location.host + window.location.pathname;
    window.history.replaceState({}, document.title, cleanUrl);
}
```

---

## 📋 موارد استفاده (Use Cases)

### ✅ زمانی که باید Redirect را فعال کنید:

1. **استفاده از CDN قوی**
   - CDN ها معمولاً بر اساس URL کامل cache می‌کنند
   - Query string باعث می‌شود CDN فایل جدید را fetch کند

2. **مشکلات cache مکرر**
   - اگر کاربران همچنان نسخه‌های قدیمی را می‌بینند
   - نیاز به force cache refresh

3. **تغییرات Critical**
   - آپدیت‌های امنیتی
   - باگ‌های مهم در JavaScript/CSS

4. **محیط Production**
   - تضمین دریافت asset های جدید برای همه کاربران

### ❌ زمانی که می‌توانید Redirect را غیرفعال کنید:

1. **بدون CDN**
   - سرور مستقیم asset ها را serve می‌کند

2. **محیط Development/Staging**
   - تست و توسعه
   - عدم نیاز به query string

3. **تجربه کاربری اولویت است**
   - نمی‌خواهید URL تغییر کند
   - حتی برای لحظه‌ای

4. **Asset Versioning در نام فایل**
   - اگر از `style.v5.5.14.css` استفاده می‌کنید
   - نیازی به query string نیست

---

## 🔍 عیب‌یابی (Troubleshooting)

### مشکل: کاربران همچنان asset های قدیمی می‌بینند

**راه‌حل 1: فعال کردن Redirect**
```
مدیریت Xpay → PageSpeed → Enable Cache Version Redirect ✅
```

**راه‌حل 2: افزایش ورژن**
```
مدیریت Xpay → Browser Cache → Clear Cache
```

**راه‌حل 3: بررسی CDN**
- آیا CDN cache headers را رعایت می‌کند؟
- آیا CDN query string را در نظر می‌گیرد؟

### مشکل: Query string همچنان در URL باقی می‌ماند

**علل احتمالی:**
1. JavaScript خطا دارد (Console را چک کنید)
2. `history.replaceState` support نمی‌شود (مرورگر قدیمی)
3. Redirect multiple times اتفاق می‌افتد

**راه‌حل:**
```javascript
// Console را باز کنید و این را اجرا کنید:
console.log('Version:', localStorage.getItem('xpay_v'));
console.log('Has params:', new URLSearchParams(window.location.search).has('_v'));
```

### مشکل: Loop های بی‌نهایت

**علت:** ورژن در localStorage ذخیره نمی‌شود

**راه‌حل:**
```javascript
// Console
localStorage.setItem('xpay_v', '<?php echo ASSET_MAIN_VERSION; ?>');
location.reload();
```

---

## 📊 توصیه‌ها (Recommendations)

### 🏆 بهترین تنظیمات برای Production:
- ✅ **Enable Cache Version Redirect: ON**
- ✅ **Enable PageSpeed Optimizations: ON**
- ✅ **Enable Third-Party Scripts: ON**

### 🧪 بهترین تنظیمات برای Development:
- ❌ **Enable Cache Version Redirect: OFF**
- ✅ **Enable PageSpeed Optimizations: ON** (برای تست)
- ❌ **Enable Third-Party Scripts: OFF** (سرعت بیشتر)

---

## 🔗 ارتباط با سایر ویژگی‌ها

### Browser Cache Management
- این ویژگی با `Browser Cache Admin` تعامل دارد
- هنگامی که در Browser Cache → Clear Cache کلیک می‌کنید:
  - ورژن افزایش می‌یابد
  - این ویژگی تصمیم می‌گیرد چطور reload کند

### Asset Versioning
- فایل‌های CSS/JS با نام `style.v5.5.14.css` ساخته می‌شوند
- Query string یک لایه اضافی برای cache busting است

### PageSpeed Optimizations
- این ویژگی بخشی از سیستم کامل PageSpeed است
- با سایر optimizations (preload, defer, async) کار می‌کند

---

## 📝 تاریخچه تغییرات

### نسخه 2.3.0 (2025-12-22)
- ✨ اضافه شدن گزینه کنترل Cache Version Redirect
- 🐛 رفع مشکل query string loop
- 📚 افزودن documentation کامل

---

## 💡 نکات پیشرفته (Advanced Tips)

### برای توسعه‌دهندگان:

**چگونه به صورت برنامه‌نویسی تنظیمات را تغییر دهیم:**
```php
// در functions.php یا plugin
$settings = get_option('xpay_pagespeed_settings', []);
$settings['enable_cache_version_redirect'] = false;
update_option('xpay_pagespeed_settings', $settings);
```

**چگونه وضعیت فعلی را بررسی کنیم:**
```php
$settings = get_option('xpay_pagespeed_settings', []);
$is_enabled = isset($settings['enable_cache_version_redirect']) 
    ? $settings['enable_cache_version_redirect'] 
    : true; // default

if ($is_enabled) {
    echo "Redirect mode: WITH query string";
} else {
    echo "Redirect mode: Silent reload";
}
```

**Hook برای override کردن رفتار:**
```php
add_filter('xpay_cache_redirect_enabled', function($enabled) {
    // فقط برای admin ها query string نشان بده
    if (current_user_can('administrator')) {
        return true;
    }
    return false; // برای بقیه silent reload
});
```

---

## 🆘 پشتیبانی (Support)

اگر مشکلی دارید:
1. Console مرورگر را چک کنید (F12)
2. localStorage را بررسی کنید: `localStorage.getItem('xpay_v')`
3. تنظیمات PageSpeed را بررسی کنید
4. Cache مرورگر را پاک کنید (Ctrl+Shift+Delete)

---

**آخرین آپدیت: 2025-12-22**
**نسخه: 2.3.0**
