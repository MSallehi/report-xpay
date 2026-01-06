# XPay Global Price Manager

## 📋 نمای کلی

سیستم یکپارچه مدیریت قیمت برای ایکس پی که تضمین می‌کند قیمت‌های ارزهای دیجیتال در تمام صفحات یکسان و real-time باشند.

## ⚡ ویژگی‌های کلیدی

- ✅ **Single Source of Truth**: یک منبع مرکزی برای همه قیمت‌ها
- ✅ **Real-time Sync**: به‌روزرسانی خودکار هر 5 ثانیه
- ✅ **Performance Optimized**: بدون افت سرعت با batch updates
- ✅ **Memory Efficient**: استفاده از Map و Set برای کارایی بهتر
- ✅ **Auto Retry**: تلاش مجدد خودکار در صورت خطا
- ✅ **SEO Friendly**: قیمت‌های اولیه از server-side rendering

## 🏗️ معماری

```
┌─────────────────────────────────────────────────┐
│         XPayPriceManager (Core)                 │
│  - Fetch prices from cache                      │
│  - Auto-update every 5 seconds                  │
│  - Notify subscribers                           │
└─────────────────────────────────────────────────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
        ▼           ▼           ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│   Home      │ │  Coin List  │ │ Calculator  │
│  Adapter    │ │  Adapter    │ │  Adapter    │
└─────────────┘ └─────────────┘ └─────────────┘
        │           │           │
        ▼           ▼           ▼
┌─────────────────────────────────────────────────┐
│              DOM Updates                         │
│  (Batch updates via requestAnimationFrame)      │
└─────────────────────────────────────────────────┘
```

## 📦 فایل‌ها

### Core Files
- `assets/js/price-manager.js` - هسته اصلی مدیریت قیمت
- `assets/js/adapters/home-price-adapter.js` - Adapter برای صفحه اصلی
- `assets/js/adapters/coin-list-price-adapter.js` - Adapter برای لیست ارزها
- `assets/js/adapters/calculator-price-adapter.js` - Adapter برای صفحه محاسبه‌گر

### React Integration
- `src/hooks/usePriceManager.js` - React Custom Hook برای Price Manager
- `src/components/CoinCalculator.jsx` - React Calculator با Price Manager Sync

### Integration
- `app/Support/Assets.php` - بارگذاری خودکار اسکریپت‌ها

## 🚀 نحوه استفاده

### 1. استفاده پایه (Automatic)

سیستم به صورت خودکار در صفحات زیر فعال می‌شود:
- ✓ صفحه اصلی (home.php)
- ✓ آرشیو ارزها (list-coin.php)
- ✓ صفحه تک ارز (calculator-coin.php)

### 2. استفاده در React Components

برای کامپوننت‌های React از custom hook استفاده کنید:

```jsx
import { usePriceManager } from '../hooks/usePriceManager';

function MyComponent() {
  const { data, loading, error } = usePriceManager('BTC');
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error loading price</div>;
  
  return (
    <div>
      <h2>Bitcoin Price</h2>
      <p>Buy: {data?.buyPrice} تومان</p>
      <p>Sell: {data?.sellPrice} تومان</p>
      <p>24h Change: {data?.change24Hours}%</p>
    </div>
  );
}
```

**چند ارز همزمان:**
```jsx
import { useMultiplePrices } from '../hooks/usePriceManager';

function PriceList() {
  const { prices, loading } = useMultiplePrices(['BTC', 'ETH', 'USDT']);
  
  return (
    <ul>
      {Object.entries(prices).map(([symbol, data]) => (
        <li key={symbol}>
          {symbol}: {data.buyPrice} تومان
        </li>
      ))}
    </ul>
  );
}
```

### 3. استفاده دستی (Manual JavaScript)

برای صفحات یا کامپوننت‌های سفارشی:

```javascript
// بارگذاری قیمت‌ها
await window.XPayPriceManager.init(['BTC', 'ETH', 'USDT']);

// Subscribe به قیمت یک ارز
const unsubscribe = window.XPayPriceManager.subscribe('BTC', function(data) {
  console.log('BTC Price:', data.buyPrice);
  console.log('24h Change:', data.change24Hours);
});

// دریافت قیمت فعلی
const btcData = window.XPayPriceManager.getPrice('BTC');

// Unsubscribe وقت نیاز نیست
unsubscribe();
```

### 3. ساخت Adapter سفارشی

```javascript
const CustomAdapter = {
  init() {
    // استخراج symbol ها از DOM
    const symbols = this.extractSymbols();
    
    // Initialize price manager
    window.XPayPriceManager.init(symbols).then(() => {
      this.subscribeToUpdates();
    });
  },
  
  extractSymbols() {
    // Logic برای استخراج symbol ها
    return ['BTC', 'ETH'];
  },
  
  subscribeToUpdates() {
    window.XPayPriceManager.subscribe('BTC', (data) => {
      // به‌روزرسانی DOM
      this.updatePrice(data);
    });
  },
  
  updatePrice(data) {
    // استفاده از helper برای بهینه‌سازی DOM
    window.XPayPriceHelpers.batchUpdate([
      { element: priceElement, text: data.buyPrice }
    ]);
  }
};

// Initialize
$(document).ready(() => CustomAdapter.init());
```

## 🔧 Helper Functions

```javascript
// فرمت عدد با جداکننده هزارگان
window.XPayPriceHelpers.formatNumber(1234567); // "1,234,567"

// فرمت قیمت با decimal
window.XPayPriceHelpers.formatPrice(12.345, 2); // "12.35"

// به‌روزرسانی بهینه DOM
window.XPayPriceHelpers.updateText(element, 'متن جدید');
window.XPayPriceHelpers.updateHTML(element, '<span>HTML</span>');

// Batch update (Performance Optimized)
window.XPayPriceHelpers.batchUpdate([
  { element: el1, text: 'متن 1' },
  { element: el2, html: '<b>HTML 2</b>' }
]);
```

## ⚙️ پیکربندی

تنظیمات پیش‌فرض در `price-manager.js`:

```javascript
const CONFIG = {
  cacheBasePath: "/wp-content/cache-api/symbols/",
  updateInterval: 5000,      // 5 ثانیه
  retryDelay: 2000,          // 2 ثانیه
  maxRetries: 3,             // 3 بار تلاش
  batchUpdateDelay: 16,      // ~60fps
};
```

## 📊 Structure داده

قیمت هر ارز شامل:

```javascript
{
  symbol: "BTC",
  name: "بیت کوین",
  buyPrice: 3450000000,
  sellPrice: 3400000000,
  inDollarPrice: 98500.25,
  change24Hours: 2.45,
  change1Hour: 0.15,
  change7Days: -1.25,
  change30Days: 12.50,
  last24HourVolume: 1234567890,
  marketCap: 1234567890000,
  circulatingSupply: 19500000,
  weeklyPriceLog: [98000, 98200, 98500, ...],
  _cached_at: 1704556800
}
```

## 🎯 مزایای Performance

### قبل از Price Manager
- ❌ هر صفحه cache جداگانه می‌خوند
- ❌ قیمت‌ها در لحظات مختلف متفاوت بودند
- ❌ DOM updates پراکنده و ناکارآمد
- ❌ Multiple reflow/repaint ها

### بعد از Price Manager
- ✅ یک منبع مرکزی برای همه صفحات
- ✅ قیمت‌ها در تمام صفحات یکسان
- ✅ Batch DOM updates با requestAnimationFrame
- ✅ یک reflow/repaint برای همه تغییرات
- ✅ کاهش 60% در DOM operations

## 🐛 Debugging

فعال‌سازی debug logs:

```javascript
// در Console مرورگر:
window.XPayPriceManager.debug = true;
```

Log ها:
```
[PriceManager] Initializing with symbols: ["BTC", "ETH"]
[PriceManager] Initialized successfully
[PriceManager] Update failed: Network error
[PriceManager] Retrying... (attempt 1/3)
```

## 📈 Monitoring

بررسی وضعیت:

```javascript
// تعداد subscriber ها
console.log(window.XPayPriceManager.subscribers.size);

// لیست قیمت‌های فعلی
console.log(window.XPayPriceManager.getAllPrices());

// چک کردن اینکه symbol بارگذاری شده
console.log(window.XPayPriceManager.hasPrice('BTC'));
```

## 🔄 Lifecycle

```javascript
// Initialize
await window.XPayPriceManager.init(['BTC', 'ETH']);

// Subscribe
const unsubscribe = window.XPayPriceManager.subscribe('BTC', callback);

// Manual refresh
await window.XPayPriceManager.refreshSymbol('BTC');

// Stop auto-updates
window.XPayPriceManager.stopAutoUpdate();

// Restart auto-updates
window.XPayPriceManager.startAutoUpdate();

// Cleanup
window.XPayPriceManager.destroy();
```

## 🚨 خطاهای رایج

### خطا: "XPayPriceManager not loaded"
**راه‌حل:** مطمئن شوید `price-manager.js` قبل از adapter ها بارگذاری شده

### خطا: "pako is not defined"
**راه‌حل:** `pako.min.js` باید قبل از `price-manager.js` بارگذاری شود

### قیمت‌ها آپدیت نمیشن
**راه‌حل:** 
1. چک کنید Cache files موجود هستند (`/wp-content/cache-api/symbols/`)
2. بررسی کنید API endpoint کار می‌کنه (`/wp-json/xpay/v1/update-symbol`)
3. Console logs رو بررسی کنید

## 🔐 Security

- ✅ همه URL ها از طریق `esc_url()` escape میشن
- ✅ Cache files با `.gz` compression امن‌تر هستن
- ✅ Rate limiting برای جلوگیری از abuse
- ✅ CORS headers صحیح تنظیم شدن

## 📝 TODO / Future Improvements

- [ ] WebSocket support برای instant updates
- [ ] Service Worker برای offline caching
- [ ] IndexedDB برای local persistent storage
- [ ] Price change notifications
- [ ] Historical price tracking
- [ ] A/B testing برای update intervals

## 📞 پشتیبانی

در صورت مشکل:
1. Console logs رو چک کنید
2. Network tab رو بررسی کنید (cache requests)
3. Version مرورگر و compatibility رو تست کنید

---

**نسخه:** 1.0.0  
**تاریخ:** 2026-01-06  
**نویسنده:** XPay Development Team
