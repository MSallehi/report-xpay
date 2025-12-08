# بهینه‌سازی Forced Reflow

## مشکل
مرورگر PageSpeed Insights گزارش می‌داد که forced reflow های متعددی در Highcharts وجود دارد که باعث کاهش عملکرد می‌شود:
- `app-vendor.js`: 159ms, 68ms, 90ms
- صفحه کلی: 170ms

## راه‌حل‌های پیاده‌سازی شده

### 1. غیرفعال کردن انیمیشن‌ها (CoinChart.jsx)
```javascript
// Configure Highcharts to minimize forced reflows
Highcharts.setOptions({
  chart: {
    animation: false,
  },
  plotOptions: {
    series: {
      animation: false,
    },
  },
});
```

### 2. فعال‌سازی Boost Module
```javascript
import HighchartsBoost from "highcharts/modules/boost";
HighchartsBoost(Highcharts);
```

این ماژول از GPU استفاده می‌کند و rendering را بهینه می‌کند.

### 3. تنظیمات Chart Options
```javascript
const options = {
  chart: {
    animation: false,
    reflow: false, // Disable automatic reflow
  },
  boost: {
    enabled: true,
    useGPUTranslations: true,
  },
  series: [{
    animation: false,
    states: {
      hover: {
        animation: false,
      },
    },
  }],
};
```

### 4. Manual Reflow Management
```javascript
useEffect(() => {
  if (chartRef.current?.chart && data.length > 0) {
    requestAnimationFrame(() => {
      try {
        chartRef.current.chart.reflow();
      } catch (e) {
        // Silently ignore reflow errors
      }
    });
  }
}, [data]);
```

فقط یک بار پس از لود داده reflow می‌کنیم.

### 5. Batched React Rendering (index.js)
```javascript
const batchRender = (elements) => {
  // Read phase - get all DOM measurements at once
  requestAnimationFrame(() => {
    const measurements = elements.map(({ id }) => {
      const el = document.getElementById(id);
      return { id, element: el, exists: !!el };
    });

    // Write phase - render all components at once
    requestAnimationFrame(() => {
      measurements.forEach(({ id, element, exists }, index) => {
        if (exists && element) {
          const { component } = elements[index];
          const root = createRoot(element);
          root.render(component);
        }
      });
    });
  });
};
```

تمام DOM reads را یک‌جا انجام می‌دهیم، سپس تمام writes را.

### 6. CSS Containment (header.php)
```css
.coin-chart-container {
  min-height: 600px;
  contain: layout style;
}

#chart-root {
  min-height: 600px;
  contain: layout style;
  content-visibility: auto;
}
```

به مرورگر می‌گوییم که این المان‌ها مستقل هستند و نباید باعث reflow کل صفحه شوند.

## نتایج

### قبل از بهینه‌سازی:
- **Total Reflow Time**: ~400ms
- app-vendor.js: 159ms, 68ms, 90ms
- [unattributed]: 170ms

### بعد از بهینه‌سازی:
- **هدف**: کاهش 80-90% زمان reflow
- انیمیشن‌های غیرضروری حذف شدند
- Reflow ها به صورت batch انجام می‌شوند
- از GPU برای rendering استفاده می‌شود

## نکات فنی

1. **CSS Containment**: `contain: layout style` به مرورگر می‌گوید این المان مستقل است
2. **Content Visibility**: `content-visibility: auto` rendering را تا زمان نیاز به تاخیر می‌اندازد
3. **RequestAnimationFrame**: تمام DOM changes را در یک frame می‌چینیم
4. **Boost Module**: Highcharts از Canvas به جای SVG برای داده‌های زیاد استفاده می‌کند

## تست

برای تست:
1. PageSpeed Insights را اجرا کنید
2. به بخش "Diagnostics" → "Forced reflow" بروید
3. باید کاهش قابل توجهی در زمان‌ها ببینید

## فایل‌های تغییر یافته

- `src/components/CoinChart.jsx`: تنظیمات Highcharts + boost
- `src/js/index.js`: Batched rendering
- `header.php`: CSS containment rules

## منابع

- [CSS Containment](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Containment)
- [Highcharts Boost Module](https://www.highcharts.com/docs/advanced-chart-features/boost-module)
- [Layout Thrashing](https://kellegous.com/j/2013/01/26/layout-performance/)
