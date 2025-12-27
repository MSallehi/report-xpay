# بهینه‌سازی INP (Interaction to Next Paint)
# INP Optimization Guide

> **نسخه:** 1.4.0  
> **تاریخ:** 27 دسامبر 2025  
> **وضعیت:** 🟢 Production Ready

---

## 📋 فهرست مطالب

1. [معرفی](#معرفی)
2. [INP چیست؟](#inp-چیست)
3. [مشکلات شناسایی شده](#مشکلات-شناسایی-شده)
4. [ماژول INPOptimizer](#ماژول-inpoptimizer)
5. [API و استفاده](#api-و-استفاده)
6. [بهینه‌سازی‌های پیاده‌شده](#بهینه‌سازی‌های-پیاده‌شده)
7. [نتایج و متریک‌ها](#نتایج-و-متریک‌ها)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)

---

## معرفی

**INP (Interaction to Next Paint)** یکی از Core Web Vitals است که زمان پاسخگویی سایت به تعاملات کاربر را اندازه‌گیری می‌کند.

### چرا INP مهم است؟

- ✅ **تجربه کاربری:** پاسخ سریع به کلیک‌ها، تایپ، اسکرول
- ✅ **SEO:** تأثیر مستقیم روی رتبه‌بندی Google
- ✅ **Conversion Rate:** کاهش Bounce Rate و افزایش تعامل

### نتایج پیش‌بینی شده

| Metric | قبل | بعد | بهبود |
|--------|-----|-----|-------|
| **INP (Interaction to Next Paint)** | ~400ms | ~100ms | **75% ⬇️** |
| **Long Tasks (>50ms)** | 15+ | <5 | **67% ⬇️** |
| **Event Handler Delay** | ~150ms | ~30ms | **80% ⬇️** |
| **Form Validation Time** | ~200ms | ~40ms | **80% ⬇️** |
| **Search Response Time** | ~350ms | ~50ms | **86% ⬇️** |

---

## INP چیست؟

### تعریف

**INP** زمان از **شروع تعامل کاربر** تا **نمایش بصری پاسخ** را اندازه‌گیری می‌کند.

```
User Action → Input Delay → Event Handler → Rendering → Visual Feedback
             └─────────────── INP ───────────────┘
```

### آستانه‌های INP

- ✅ **Good:** < 200ms (سبز)
- ⚠️ **Needs Improvement:** 200ms - 500ms (نارنجی)
- ❌ **Poor:** > 500ms (قرمز)

### عوامل تأثیرگذار بر INP

1. **Long Tasks:** Tasks بیش از 50ms که main thread را block می‌کنند
2. **Heavy Event Handlers:** JavaScript کدهای سنگین در event handlers
3. **Layout Thrashing:** Forced reflows و layout recalculations
4. **Large DOM:** DOM بزرگ که rendering را کند می‌کند
5. **Blocking Scripts:** Scripts که rendering را block می‌کنند

---

## مشکلات شناسایی شده

### 1. Interactive Components روی Main Thread

**مشکل:**
```javascript
// Components بلافاصله initialize می‌شوند
document.addEventListener('DOMContentLoaded', () => {
  initModals();      // Blocks 50ms
  initTooltips();    // Blocks 30ms
  initAnimations();  // Blocks 40ms
  initForms();       // Blocks 60ms
  // Total: ~180ms blocking time
});
```

**تأثیر:** INP = ~400ms

### 2. Long Tasks بدون Break

**مشکل:**
```javascript
// Process 1000 items without yielding
for (let i = 0; i < 1000; i++) {
  processItem(items[i]);  // Total: 250ms blocking
}
```

**تأثیر:** Main thread blocked for 250ms

### 3. yieldToMain تعریف شده اما استفاده نشده

**مشکل:**
```javascript
// yieldToMain در header.php تعریف شده:
window.yieldToMain = function() { ... }

// اما در کدها استفاده نمی‌شود:
heavyTask1();  // 100ms
heavyTask2();  // 80ms
heavyTask3();  // 120ms
// Total: 300ms without yielding
```

### 4. Event Handlers بدون Debounce/Throttle

**مشکل:**
```javascript
// Search input بدون debounce
searchInput.addEventListener('input', (e) => {
  performSearch(e.target.value);  // اجرا می‌شود با هر keystroke
});

// Scroll handler بدون throttle
window.addEventListener('scroll', () => {
  updateScrollPosition();  // اجرا می‌شود هر چند میلی‌ثانیه
});
```

**تأثیر:** INP spike to 500ms+ during typing/scrolling

---

## ماژول INPOptimizer

### معماری

```
INPOptimizer
├── Task Scheduler (Priority Queue)
│   ├── High Priority Queue
│   ├── Normal Priority Queue
│   └── Low Priority Queue
│
├── Task Processor (with Yielding)
│   ├── processInChunks()
│   ├── scheduleTask()
│   └── yieldToMain()
│
├── Component Optimizers
│   ├── optimizeModals()
│   ├── optimizeTooltips()
│   ├── deferAnimations()
│   ├── optimizeForms()
│   └── optimizeSearch()
│
├── Event Optimizations
│   ├── debounce()
│   ├── throttle()
│   ├── Scroll Handler (throttled)
│   └── Resize Handler (throttled)
│
└── Performance Monitoring
    ├── Long Task Detection
    ├── INP Measurement (Event Timing API)
    └── Performance Report
```

### ویژگی‌های کلیدی

#### 1. **Task Scheduler با Priority Queue**

```javascript
// High priority - Execute ASAP
INPOptimizer.scheduleTask(() => {
  updateCriticalUI();
}, 'high');

// Normal priority - Execute when main thread free
INPOptimizer.scheduleTask(() => {
  updateNonCriticalData();
}, 'normal');

// Low priority - Execute during idle time
INPOptimizer.scheduleTask(() => {
  prefetchImages();
}, 'low');
```

#### 2. **Break Long Tasks با processInChunks()**

```javascript
// بجای blocking task:
for (let i = 0; i < 1000; i++) {
  processItem(items[i]);
}

// استفاده از chunk processing:
await INPOptimizer.processInChunks(items, async (item, index) => {
  processItem(item);
}, {
  chunkSize: 50,        // 50 items per chunk
  priority: 'normal',
  onProgress: (done, total) => {
    console.log(`Progress: ${done}/${total}`);
  }
});
```

#### 3. **Enhanced yieldToMain()**

```javascript
// استفاده از scheduler.yield() (Chrome 94+)
await window.yieldToMain();

// Fallback chain:
// 1. scheduler.yield() (Chrome 94+)
// 2. requestIdleCallback (Chrome 47+, Firefox 55+)
// 3. setTimeout(0) (All browsers)
```

#### 4. **Component Optimization**

```javascript
// Modals - Initialize on demand
<button data-modal-trigger="myModal">Open</button>

// Tooltips - Initialize on hover/focus
<span data-tooltip="Info">Hover me</span>

// Animations - Defer with Intersection Observer
<div data-animation class="animate-on-scroll">Content</div>

// Forms - Debounced validation
<input type="text" data-validate />

// Search - Debounced search
<input type="search" class="search-input" />
```

#### 5. **Event Handler Optimization**

```javascript
// Scroll handler (throttled 100ms)
INPOptimizer.onScroll((e) => {
  updateScrollPosition();
});

// Resize handler (throttled 100ms)
INPOptimizer.onResize((e) => {
  updateLayout();
});
```

---

## API و استفاده

### Configuration

```javascript
// تنظیمات در inp-optimizer.js
INPOptimizer.config = {
  longTaskThreshold: 50,      // Long task threshold (ms)
  chunkSize: 50,              // Items per chunk
  idleTimeout: 1000,          // Idle callback timeout (ms)
  debounceDelay: 300,         // Debounce delay for forms/search (ms)
  throttleDelay: 100,         // Throttle delay for scroll/resize (ms)
  debug: false,               // Enable debug logging
  
  components: {
    modals: true,             // Optimize modals
    tooltips: true,           // Optimize tooltips
    animations: true,         // Defer animations
    forms: true,              // Optimize form validation
    search: true              // Optimize search
  }
};
```

### Public Methods

#### `scheduleTask(task, priority)`

Schedule a task with priority.

```javascript
// High priority
INPOptimizer.scheduleTask(async () => {
  await updateCriticalData();
}, 'high');

// Normal priority (default)
INPOptimizer.scheduleTask(async () => {
  await updateData();
});

// Low priority
INPOptimizer.scheduleTask(async () => {
  await prefetchData();
}, 'low');
```

#### `processInChunks(items, callback, options)`

Process array items in chunks with yielding.

```javascript
const items = Array.from({ length: 1000 }, (_, i) => i);

await INPOptimizer.processInChunks(items, async (item, index) => {
  // Process item
  processItem(item);
}, {
  chunkSize: 50,
  priority: 'normal',
  onProgress: (done, total) => {
    updateProgressBar(done / total * 100);
  }
});
```

#### `getPerformanceReport()`

Get performance metrics report.

```javascript
const report = INPOptimizer.getPerformanceReport();

console.log(report);
// Output:
// {
//   longTasks: {
//     count: 3,
//     avgDuration: "65.33ms",
//     list: [...]
//   },
//   interactions: {
//     count: 12,
//     avgDuration: "120.50ms",
//     list: [...]
//   },
//   optimizedComponents: ['modals', 'tooltips', 'forms', 'search'],
//   queuedTasks: {
//     high: 0,
//     normal: 2,
//     low: 5
//   }
// }
```

### Events

#### `inp-optimizer:initialized`

Fired when optimizer is initialized.

```javascript
document.addEventListener('inp-optimizer:initialized', () => {
  console.log('INP Optimizer ready!');
});
```

#### `inp-optimizer:long-task`

Fired when long task detected.

```javascript
document.addEventListener('inp-optimizer:long-task', (e) => {
  const { name, duration } = e.detail;
  console.warn(`Long task: ${name} (${duration}ms)`);
});
```

#### `inp-optimizer:search`

Fired when search query entered.

```javascript
document.addEventListener('inp-optimizer:search', (e) => {
  const { query } = e.detail;
  performSearch(query);
});
```

---

## بهینه‌سازی‌های پیاده‌شده

### 1. Modal Optimization

**قبل:**
```javascript
// همه modals در DOMContentLoaded initialize می‌شوند
$(document).ready(() => {
  $('.modal').modal();  // Blocks 50ms
});
```

**بعد:**
```javascript
// Modals فقط هنگام کلیک روی trigger initialize می‌شوند
<button data-modal-trigger="myModal">Open Modal</button>

// INPOptimizer automatically:
// 1. Defers initialization
// 2. Yields before heavy initialization
// 3. Adds optimized click handler
```

**نتیجه:** 50ms → 0ms blocking time

---

### 2. Tooltip Optimization

**قبل:**
```javascript
// همه tooltips در DOMContentLoaded bind می‌شوند
$('[data-tooltip]').tooltip();  // Blocks 30ms
```

**بعد:**
```javascript
// Tooltips فقط هنگام hover/focus initialize می‌شوند
<span data-tooltip="Info">Hover me</span>

// INPOptimizer automatically:
// 1. Defers tooltip binding
// 2. Initializes on first hover/focus
// 3. Uses { once: true } for initialization
```

**نتیجه:** 30ms → 0ms blocking time

---

### 3. Animation Optimization

**قبل:**
```javascript
// Animations بلافاصله start می‌شوند
$('.animate').addClass('animated');  // All elements animate
```

**بعد:**
```javascript
// Animations با Intersection Observer start می‌شوند
<div data-animation class="animate-on-scroll">Content</div>

// INPOptimizer automatically:
// 1. Observes elements with Intersection Observer
// 2. Yields before adding animation class
// 3. Only animates visible elements
```

**نتیجه:** 40ms → 0ms blocking time

---

### 4. Form Validation Optimization

**قبل:**
```javascript
// Validation بلافاصله اجرا می‌شود
input.addEventListener('input', () => {
  validateInput(input);  // Runs every keystroke
});
```

**بعد:**
```javascript
// Validation با debounce (300ms) اجرا می‌شود
<input type="text" data-validate />

// INPOptimizer automatically:
// 1. Debounces validation (300ms)
// 2. Yields before validation
// 3. Uses passive event listeners
```

**نتیجه:** 200ms → 40ms average response time

---

### 5. Search Optimization

**قبل:**
```javascript
// Search بلافاصله اجرا می‌شود
searchInput.addEventListener('input', () => {
  performSearch(searchInput.value);  // Every keystroke
});
```

**بعد:**
```javascript
// Search با debounce (300ms) اجرا می‌شود
<input type="search" class="search-input" />

// INPOptimizer automatically:
// 1. Debounces search (300ms)
// 2. Yields before search
// 3. Dispatches custom event
// 4. Minimum 2 characters

document.addEventListener('inp-optimizer:search', (e) => {
  performSearch(e.detail.query);
});
```

**نتیجه:** 350ms → 50ms average response time

---

### 6. Scroll Handler Optimization

**قبل:**
```javascript
// Scroll handler هر چند میلی‌ثانیه اجرا می‌شود
window.addEventListener('scroll', () => {
  updateScrollPosition();  // Runs ~100 times per second
});
```

**بعد:**
```javascript
// Scroll handler با throttle (100ms) اجرا می‌شود
INPOptimizer.onScroll(async () => {
  await window.yieldToMain();
  updateScrollPosition();
});
```

**نتیجه:** ~100 calls/sec → ~10 calls/sec

---

### 7. Long Task Breaking

**قبل:**
```javascript
// Process 1000 items without yielding
for (let i = 0; i < 1000; i++) {
  processItem(items[i]);  // Total: 250ms blocking
}
```

**بعد:**
```javascript
// Process in chunks of 50 with yielding
await INPOptimizer.processInChunks(items, (item) => {
  processItem(item);
}, {
  chunkSize: 50  // Yields every 50 items
});

// Effective blocking: ~12ms per chunk
// Total time: ~270ms (with yields)
// But main thread available every 12ms
```

**نتیجه:** 250ms blocking → 12ms max per chunk

---

## نتایج و متریک‌ها

### Expected Results

| Metric | قبل | بعد | بهبود |
|--------|-----|-----|-------|
| **INP (Interaction to Next Paint)** | ~400ms | ~100ms | **75% ⬇️** |
| **Long Tasks Count** | 15+ | <5 | **67% ⬇️** |
| **Long Tasks Avg Duration** | ~85ms | ~45ms | **47% ⬇️** |
| **Event Handler Delay** | ~150ms | ~30ms | **80% ⬇️** |
| **Form Validation Time** | ~200ms | ~40ms | **80% ⬇️** |
| **Search Response Time** | ~350ms | ~50ms | **86% ⬇️** |
| **Modal Initialization** | 50ms blocking | 0ms blocking | **100% ⬇️** |
| **Tooltip Initialization** | 30ms blocking | 0ms blocking | **100% ⬇️** |
| **Animation Start** | 40ms blocking | 0ms blocking | **100% ⬇️** |
| **Scroll Handler Frequency** | ~100/sec | ~10/sec | **90% ⬇️** |

### Performance Monitoring

#### Enable Debug Mode

```javascript
// در console browser:
INPOptimizer.config.debug = true;
```

#### Get Performance Report

```javascript
const report = INPOptimizer.getPerformanceReport();
console.table(report.longTasks.list);
console.table(report.interactions.list);
```

#### Monitor Events

```javascript
// Long task alerts
document.addEventListener('inp-optimizer:long-task', (e) => {
  alert(`Long task: ${e.detail.name} (${e.detail.duration}ms)`);
});

// Interaction tracking
let interactionCount = 0;
document.addEventListener('inp-optimizer:search', () => {
  interactionCount++;
  console.log(`Total searches: ${interactionCount}`);
});
```

---

## Troubleshooting

### مشکل 1: INP هنوز بالا است (>300ms)

**علل احتمالی:**
- Custom event handlers که optimize نشده‌اند
- Third-party scripts که optimize نشده‌اند
- Large bundle sizes

**راه‌حل:**
```javascript
// 1. شناسایی long tasks
const report = INPOptimizer.getPerformanceReport();
console.table(report.longTasks.list);

// 2. Optimize custom handlers
document.querySelector('.my-button').addEventListener('click', async () => {
  await window.yieldToMain();  // Yield before heavy work
  doHeavyWork();
});

// 3. Break large bundles
// Use dynamic imports:
const module = await import('./heavy-module.js');
```

---

### مشکل 2: Modals/Tooltips کار نمی‌کنند

**علل احتمالی:**
- Bootstrap/jQuery loaded بعد از INPOptimizer
- Custom initialization code

**راه‌حل:**
```javascript
// 1. Disable component optimization temporarily
INPOptimizer.config.components.modals = false;
INPOptimizer.config.components.tooltips = false;

// 2. Initialize manually after INPOptimizer
document.addEventListener('inp-optimizer:initialized', () => {
  // Initialize your modals/tooltips here
  $('.modal').modal();
  $('[data-tooltip]').tooltip();
});
```

---

### مشکل 3: Event handlers اجرا نمی‌شوند

**علل احتمالی:**
- Event listeners attached قبل از INPOptimizer
- Conflicting event handlers

**راه‌حل:**
```javascript
// Use INPOptimizer API for handlers
INPOptimizer.onScroll((e) => {
  // Your scroll handler
});

INPOptimizer.onResize((e) => {
  // Your resize handler
});
```

---

### مشکل 4: Console errors در browser

**علل احتمالی:**
- Browser قدیمی که از scheduler.yield() پشتیبانی نمی‌کند
- PerformanceObserver not supported

**راه‌حل:**
```javascript
// INPOptimizer automatically falls back:
// 1. scheduler.yield() → requestIdleCallback → setTimeout
// 2. PerformanceObserver errors are caught and logged

// Check browser support:
console.log('Scheduler API:', 'scheduler' in window && 'yield' in scheduler);
console.log('PerformanceObserver:', 'PerformanceObserver' in window);
```

---

## Best Practices

### 1. **Always Yield in Long Tasks**

```javascript
// ❌ Bad
for (let i = 0; i < 1000; i++) {
  processItem(items[i]);
}

// ✅ Good
await INPOptimizer.processInChunks(items, (item) => {
  processItem(item);
});
```

---

### 2. **Use Priority Queue Wisely**

```javascript
// ✅ Good
// Critical UI update - high priority
INPOptimizer.scheduleTask(() => {
  updateCartCount();
}, 'high');

// Data fetch - normal priority
INPOptimizer.scheduleTask(() => {
  fetchRecommendations();
}, 'normal');

// Prefetch - low priority
INPOptimizer.scheduleTask(() => {
  prefetchImages();
}, 'low');
```

---

### 3. **Debounce Form Validation**

```javascript
// ❌ Bad
input.addEventListener('input', () => {
  validateInput(input);  // Every keystroke
});

// ✅ Good
<input type="text" data-validate />
// INPOptimizer automatically debounces
```

---

### 4. **Defer Non-Critical Components**

```javascript
// ✅ Good
// Modals
<button data-modal-trigger="myModal">Open</button>

// Tooltips
<span data-tooltip="Info">Hover</span>

// Animations
<div data-animation class="animate-on-scroll">Content</div>
```

---

### 5. **Monitor Performance Regularly**

```javascript
// Enable monitoring in development
if (location.hostname === 'localhost') {
  INPOptimizer.config.debug = true;
  
  setInterval(() => {
    const report = INPOptimizer.getPerformanceReport();
    console.log('Long tasks:', report.longTasks.count);
    console.log('Avg INP:', report.interactions.avgDuration);
  }, 10000);
}
```

---

### 6. **Use Passive Event Listeners**

```javascript
// ✅ Good
element.addEventListener('touchstart', handler, { passive: true });
element.addEventListener('wheel', handler, { passive: true });

// INPOptimizer automatically uses passive listeners
```

---

### 7. **Optimize Third-Party Scripts**

```javascript
// Defer third-party scripts
INPOptimizer.scheduleTask(async () => {
  await window.yieldToMain();
  
  // Load analytics
  loadGoogleAnalytics();
  
  await window.yieldToMain();
  
  // Load chat widget
  loadChatWidget();
}, 'low');
```

---

## Testing & Validation

### Chrome DevTools

#### Performance Tab

1. Open DevTools → **Performance** tab
2. Record page interaction
3. Look for:
   - **Long Tasks** (red bars >50ms)
   - **Event Handlers** (green bars)
   - **Rendering** (purple bars)

#### Performance Insights

1. DevTools → **Performance Insights** tab
2. Record interaction
3. Check **INP** metric
4. Identify blocking tasks

---

### PageSpeed Insights

```
https://pagespeed.web.dev/?url=https://xpay.co
```

Check:
- ✅ **INP** < 200ms (Good)
- ✅ **Total Blocking Time** < 200ms
- ✅ **Long Tasks** < 5

---

### Web Vitals Extension

Install: [Chrome Web Store](https://chrome.google.com/webstore/detail/web-vitals/)

Monitor:
- **INP** (real-time)
- **CLS** (Cumulative Layout Shift)
- **LCP** (Largest Contentful Paint)

---

### Console Commands

```javascript
// Get performance report
const report = INPOptimizer.getPerformanceReport();
console.table(report.longTasks.list);
console.table(report.interactions.list);

// Enable debug mode
INPOptimizer.config.debug = true;

// Schedule test task
INPOptimizer.scheduleTask(() => {
  console.log('Task executed!');
}, 'high');

// Test chunk processing
await INPOptimizer.processInChunks(
  Array.from({ length: 100 }, (_, i) => i),
  (item) => console.log(item)
);
```

---

## مقایسه با Performance Optimizer

| Feature | Performance Optimizer | INP Optimizer |
|---------|----------------------|---------------|
| **هدف** | LCP, CLS, TBT | INP, Event Response |
| **Lazy Loading** | ✅ Scripts, Styles, Images | ❌ |
| **Widget Deferring** | ✅ Chat, Modal, Price | ❌ |
| **Font Optimization** | ✅ font-display, preload | ❌ |
| **Task Scheduler** | ❌ | ✅ Priority Queue |
| **Long Task Breaking** | ❌ | ✅ processInChunks() |
| **Component Optimization** | ❌ | ✅ Modals, Tooltips, Forms |
| **Event Optimization** | ❌ | ✅ Debounce, Throttle |
| **Performance Monitoring** | ⚠️ Basic | ✅ Advanced (Long Tasks, INP) |

**نتیجه:** هر دو ماژول باید همزمان استفاده شوند.

---

## منابع و مراجع

### مستندات رسمی

- [INP (web.dev)](https://web.dev/inp/)
- [Optimize INP (web.dev)](https://web.dev/optimize-inp/)
- [Event Timing API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Event_timing_API)
- [Scheduler API (Chrome)](https://developer.chrome.com/blog/scheduling-tasks-api/)
- [requestIdleCallback (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback)

### ابزارهای تست

- [PageSpeed Insights](https://pagespeed.web.dev/)
- [Chrome DevTools Performance](https://developer.chrome.com/docs/devtools/performance/)
- [Web Vitals Extension](https://chrome.google.com/webstore/detail/web-vitals/)

### مقالات مرتبط

- [Optimize long tasks (web.dev)](https://web.dev/optimize-long-tasks/)
- [Debouncing and Throttling (CSS-Tricks)](https://css-tricks.com/debouncing-throttling-explained-examples/)
- [Main Thread Performance (web.dev)](https://web.dev/mainthread-work-breakdown/)

---

## Changelog

### Version 1.4.0 (2025-12-27)

- ✅ Created INPOptimizer module
- ✅ Task scheduler with priority queue (high, normal, low)
- ✅ processInChunks() for breaking long tasks
- ✅ Enhanced yieldToMain() with scheduler.yield()
- ✅ Component optimization (modals, tooltips, animations, forms, search)
- ✅ Event handler optimization (debounce, throttle)
- ✅ Performance monitoring (Long Tasks, INP)
- ✅ Complete API documentation
- ✅ Best practices guide
- ✅ Expected results: 75% INP reduction

---

**📊 پایان مستندات INP Optimization**

> آخرین بروزرسانی: 27 دسامبر 2025 | نسخه: 1.4.0 | وضعیت: 🟢 Production Ready
