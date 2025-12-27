# ⚡ Performance Optimization - Quick Reference

> راهنمای سریع برای استفاده از PerformanceOptimizer

## 🚀 شروع سریع

### بدون نیاز به تنظیمات اضافی!

PerformanceOptimizer به صورت **خودکار** فعال می‌شود و شامل:

✅ Lazy loading برای scripts, styles, images  
✅ Defer widgets (Chat, Modal, Price)  
✅ Font optimization (font-display: swap)  
✅ Performance monitoring  

## 📊 نتایج مورد انتظار

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **LCP** | ~4.5s | ~2.0s | **-55%** |
| **FID** | ~200ms | ~50ms | **-75%** |
| **CLS** | ~0.25 | ~0.05 | **-80%** |
| **Page Load** | ~6s | ~3s | **-50%** |

## 🎯 ویژگی‌های کلیدی

### 1. Lazy Loading

```html
<!-- Script -->
<div data-lazy-script="/path/to/script.js" data-lazy-script-id="my-script"></div>

<!-- Style -->
<div data-lazy-style="/path/to/style.css" data-lazy-style-id="my-style"></div>

<!-- Image -->
<img data-lazy-src="image.jpg" alt="Image">
```

### 2. Widget Delays (Default)

- **Chat (گفتینو)**: 3s یا first interaction  
- **Modal (گردونه)**: 5s یا button click  
- **Price Widget**: 2s یا viewport entry  

### 3. Font Optimization

```css
/* Automatic font-display: swap */
@font-face {
  font-family: 'IRANSans';
  src: url('IRANSansWeb.woff2') format('woff2');
  font-display: swap; /* ⬅️ Auto-added */
}
```

## 🛠️ تنظیمات (اختیاری)

### فعال کردن Debug Mode

```javascript
// در performance-optimizer.js
config: {
  debug: true  // ⬅️ Console logs را فعال می‌کند
}
```

### تغییر Widget Delays

```javascript
// در performance-optimizer.js
widgetDelays: {
  chat: 5000,        // 5 ثانیه
  modal: 3000,       // 3 ثانیه
  priceWidget: 1000  // 1 ثانیه
}
```

## 📡 Events

```javascript
// Initialized
document.addEventListener('performance-optimizer:initialized', () => {
  console.log('Ready!');
});

// Chat loaded
document.addEventListener('performance-optimizer:chat-loaded', () => {
  console.log('Chat ready');
});

// Modal loaded
document.addEventListener('performance-optimizer:modal-loaded', () => {
  console.log('Modal ready');
});

// Price widget loaded
document.addEventListener('performance-optimizer:price-widget-loaded', () => {
  console.log('Price widget ready');
});
```

## 🔧 API Methods

```javascript
const optimizer = window.PerformanceOptimizer;

// Load script
optimizer.loadScript('/path/to/script.js', 'script-id')
  .then(() => console.log('Loaded'))
  .catch(err => console.error(err));

// Load style
optimizer.loadStyle('/path/to/style.css', 'style-id')
  .then(() => console.log('Loaded'))
  .catch(err => console.error(err));

// Check state
console.log(optimizer.state.widgetsLoaded);
// { chat: true, modal: false, priceWidget: true }
```

## 🖼️ Image Best Practices

### 1. Always set width & height

```html
<img src="image.jpg" width="800" height="600" alt="Image">
```

### 2. Use modern formats

```html
<picture>
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Image">
</picture>
```

### 3. Preload LCP image

```html
<link rel="preload" as="image" href="hero.jpg">
<img src="hero.jpg" alt="Hero" fetchpriority="high">
```

## 🔤 Font Best Practices

### 1. Preload critical fonts

```html
<link rel="preload" as="font" type="font/woff2" 
      href="/fonts/IRANSansWeb.woff2" crossorigin>
```

### 2. Use font-display: swap

```css
@font-face {
  font-family: 'IRANSans';
  src: url('IRANSansWeb.woff2') format('woff2');
  font-display: swap; /* ⬅️ Always */
}
```

## 🎮 Widget Best Practices

### Chat Widget (گفتینو)

```html
<!-- Defer تا 3 ثانیه یا first interaction -->
<!-- هیچ کد اضافه‌ای نیاز نیست! -->
```

### Modal Widget (گردونه)

```html
<!-- Add trigger button -->
<button data-modal-trigger>باز کردن گردونه</button>

<!-- Modal loads on click or after 5s -->
```

### Price Widget

```html
<!-- Add price-widget class -->
<div class="price-widget" data-coin="bitcoin">
  <!-- Chart container -->
</div>

<!-- Loads when in viewport or after 2s -->
```

## 🐛 Troubleshooting

### Widget not loading?

```javascript
// Check console
console.log(PerformanceOptimizer.state.widgetsLoaded);

// Check for errors
// با Debug Mode فعال کنید
```

### Images shifting layout?

```html
<!-- Set dimensions -->
<img src="image.jpg" width="800" height="600" alt="Image">

<!-- Or use CSS -->
<style>
  .image { aspect-ratio: 16/9; width: 100%; }
</style>
```

### Fonts not loading?

```html
<!-- Add crossorigin -->
<link rel="preload" as="font" type="font/woff2" 
      href="/fonts/font.woff2" crossorigin>
```

## 📈 Performance Testing

### PageSpeed Insights
```
https://pagespeed.web.dev/?url=https://xpay.ir
```

### WebPageTest
```
https://www.webpagetest.org/
Location: Tehran, Iran
Browser: Chrome
Connection: 4G
```

### Lighthouse (Local)
```bash
npm install -g lighthouse
lighthouse https://xpay.ir --view
```

## 📚 مستندات کامل

برای اطلاعات بیشتر:

- [PERFORMANCE-OPTIMIZATION.md](./PERFORMANCE-OPTIMIZATION.md) - راهنمای کامل
- [FORCED_REFLOW_OPTIMIZATION.md](./FORCED_REFLOW_OPTIMIZATION.md) - Reflow optimization
- [CHANGELOG.md](./CHANGELOG.md) - تغییرات نسخه‌ها

## 🎯 Load Order (اطلاعات فنی)

```
1. jQuery
2. reflow-optimizer.js
3. performance-optimizer.js  ⬅️ NEW
4. dom-interceptor.js
5. swiper-wrapper.js
6. app-vendor.js (jQuery, Swiper)
7. custom-coins.js
```

## ✅ Checklist قبل از Deploy

- [ ] Debug mode غیرفعال است
- [ ] Widget delays مناسب تنظیم شده
- [ ] تمام تصاویر width/height دارند
- [ ] فونت‌های حیاتی preload شده‌اند
- [ ] LCP image preload شده است
- [ ] PageSpeed test انجام شده
- [ ] Console errors بررسی شده

---

**نویسنده:** XPay Development Team  
**تاریخ:** 27 دسامبر 2025  
**نسخه:** 1.0.0

**⚡ Performance is a feature!**
