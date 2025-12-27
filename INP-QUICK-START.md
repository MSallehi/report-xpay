# راهنمای سریع INP Optimizer
# INP Optimizer Quick Start

> **نسخه:** 1.4.0  
> **بروزرسانی:** 27 دسامبر 2025

---

## 🚀 شروع سریع

### فعال‌سازی

1. **برو به:** WP Admin → Xpay Management → **PageSpeed**
2. **فعال کن:**
   - ✅ Enable PageSpeed Optimizations
   - ✅ Enable Performance Optimizer (LCP, CLS, TBT)
   - ✅ Enable INP Optimizer (Interaction to Next Paint)
3. **ذخیره کن** و **Hard Refresh** (`Ctrl+Shift+R`)

---

## 📊 نتایج مورد انتظار

| Metric | قبل | بعد | بهبود |
|--------|-----|-----|-------|
| **INP** | ~400ms | ~100ms | ✅ 75% |
| **Long Tasks** | 15+ | <5 | ✅ 67% |
| **Event Delay** | ~150ms | ~30ms | ✅ 80% |
| **Form Validation** | ~200ms | ~40ms | ✅ 80% |
| **Search Response** | ~350ms | ~50ms | ✅ 86% |

---

## 🎯 استفاده در کد

### 1. Break Long Tasks

```javascript
// ❌ قبل - Long task (250ms blocking)
for (let i = 0; i < 1000; i++) {
  processItem(items[i]);
}

// ✅ بعد - Chunked with yielding
await INPOptimizer.processInChunks(items, (item) => {
  processItem(item);
}, {
  chunkSize: 50,
  priority: 'normal'
});
```

---

### 2. Schedule Tasks با Priority

```javascript
// High priority - Critical UI update
INPOptimizer.scheduleTask(() => {
  updateCartCount();
}, 'high');

// Normal priority - Data update
INPOptimizer.scheduleTask(() => {
  fetchData();
}, 'normal');

// Low priority - Prefetch
INPOptimizer.scheduleTask(() => {
  prefetchImages();
}, 'low');
```

---

### 3. Optimize Event Handlers

```javascript
// Scroll handler (throttled automatically)
INPOptimizer.onScroll((e) => {
  updateScrollPosition();
});

// Resize handler (throttled automatically)
INPOptimizer.onResize((e) => {
  updateLayout();
});
```

---

### 4. Component Optimization

```html
<!-- Modals - Load on click -->
<button data-modal-trigger="myModal">Open Modal</button>

<!-- Tooltips - Load on hover -->
<span data-tooltip="Info">Hover me</span>

<!-- Animations - Load when visible -->
<div data-animation class="animate-on-scroll">Content</div>

<!-- Forms - Debounced validation -->
<input type="text" data-validate />

<!-- Search - Debounced search -->
<input type="search" class="search-input" />
```

---

## 🛠️ Configuration

```javascript
// فعال کردن debug mode
INPOptimizer.config.debug = true;

// تنظیم chunk size
INPOptimizer.config.chunkSize = 100;

// تنظیم debounce delay
INPOptimizer.config.debounceDelay = 500;

// غیرفعال کردن component optimization
INPOptimizer.config.components.modals = false;
```

---

## 📈 Performance Monitoring

```javascript
// دریافت گزارش عملکرد
const report = INPOptimizer.getPerformanceReport();
console.table(report.longTasks.list);
console.table(report.interactions.list);

// Monitor long tasks
document.addEventListener('inp-optimizer:long-task', (e) => {
  console.warn('Long task:', e.detail.name, e.detail.duration);
});
```

---

## 🔍 Troubleshooting

### INP هنوز بالا است

```javascript
// 1. بررسی long tasks
const report = INPOptimizer.getPerformanceReport();
console.table(report.longTasks.list);

// 2. فعال کردن debug mode
INPOptimizer.config.debug = true;

// 3. بررسی custom handlers
// آیا event handlers سنگین دارید که optimize نشده‌اند؟
```

---

### Components کار نمی‌کنند

```javascript
// Disable component optimization موقتاً
INPOptimizer.config.components.modals = false;
INPOptimizer.config.components.tooltips = false;

// یا initialize manual بعد از INPOptimizer
document.addEventListener('inp-optimizer:initialized', () => {
  $('.modal').modal();
  $('[data-tooltip]').tooltip();
});
```

---

## ✅ Checklist قبل از Deploy

- [ ] **PageSpeed Admin**: Enable INP Optimizer فعال است
- [ ] **Console**: بدون error
- [ ] **INP < 200ms**: در PageSpeed Insights سبز است
- [ ] **Long Tasks < 5**: تعداد کم است
- [ ] **Components**: Modals, Tooltips کار می‌کنند
- [ ] **Forms**: Validation کار می‌کند
- [ ] **Search**: نتایج نمایش داده می‌شود
- [ ] **Scroll**: Smooth است بدون lag
- [ ] **Resize**: بدون مشکل است

---

## 🧪 Testing Commands

```javascript
// Test task scheduling
INPOptimizer.scheduleTask(() => {
  console.log('High priority task');
}, 'high');

// Test chunk processing
await INPOptimizer.processInChunks(
  Array.from({ length: 100 }, (_, i) => i),
  (item) => console.log('Processing:', item)
);

// Test performance report
const report = INPOptimizer.getPerformanceReport();
console.log('Long tasks:', report.longTasks.count);
console.log('Interactions:', report.interactions.count);
```

---

## 📚 مستندات کامل

برای جزئیات بیشتر:
- [INP-OPTIMIZATION.md](INP-OPTIMIZATION.md) - راهنمای کامل 700+ خطی
- [CHANGELOG.md](CHANGELOG.md) - تاریخچه تغییرات
- [PERFORMANCE-OPTIMIZATION.md](PERFORMANCE-OPTIMIZATION.md) - Performance Optimizer

---

## 🆘 پشتیبانی

- **مشکلی دارید؟** [GitHub Issues](https://github.com/Xpay-wp/xpay-wp/issues)
- **سوال دارید؟** support@xpay.co

---

**📖 پایان Quick Start**

> آخرین بروزرسانی: 27 دسامبر 2025 | نسخه: 1.4.0
