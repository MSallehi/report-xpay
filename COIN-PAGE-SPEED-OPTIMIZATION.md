# 🚀 بهینه‌سازی سرعت صفحات Coin (Request-Level Cache + AJAX Loader)

> **نسخه:** 1.0.0  
> **تاریخ:** فوریه 2026  
> **وضعیت:** 🟢 فعال

---

## 📋 خلاصه

این مستند شامل دو بهینه‌سازی اصلی برای بهبود سرعت لود صفحات coin است:

1. **Request-Level Cache**: کاهش تعداد API calls از 4 به 1 در هر request
2. **AJAX Price Loader**: لود قیمت‌ها به صورت asynchronous بعد از render صفحه

---

## 🔴 مشکل قبلی

### وضعیت قبل از بهینه‌سازی

هنگام لود یک صفحه coin (مثل `/coin/tron/`)، چندین API call مجزا به سرور xpay.co زده می‌شد:

```
API Call 1: get_current_coin('TRX')     → calculator-coin.php
API Call 2: get_current_coin('TRX')     → coin_live_data shortcode  
API Call 3: get_current_coin('TRX')     → coin_change_data shortcode
API Call 4: get_current_coin('USDT')    → getUSDTData() for volume calculation
```

**نتیجه:** هر API call حدود 5-10 ثانیه timeout داشت → **TTFB: 16-18 ثانیه!**

---

## 🟢 راه‌حل 1: Request-Level Cache

### توضیح

یک cache در سطح PHP request که اطمینان حاصل می‌کند هر symbol فقط **یکبار** از API خوانده شود.

### تغییرات کد

#### فایل: `app/Support/helpers.php`

```php
function get_current_coin($symbol, $force_refresh = false)
{
    $symbol = strtoupper(trim($symbol));
    
    if (empty($symbol)) {
        return null;
    }

    // Initialize global request-level cache if not exists
    if (!isset($GLOBALS['xpay_coin_request_cache'])) {
        $GLOBALS['xpay_coin_request_cache'] = [];
    }

    // Check request-level cache first (unless force refresh)
    if (!$force_refresh && isset($GLOBALS['xpay_coin_request_cache'][$symbol])) {
        return $GLOBALS['xpay_coin_request_cache'][$symbol];
    }

    // ... API call logic ...

    // Store in request-level cache
    if ($coin_data !== null) {
        $GLOBALS['xpay_coin_request_cache'][$symbol] = $coin_data;
    }

    return $coin_data;
}
```

#### فایل: `app/Controllers/ShortcodeController.php`

```php
private function getUSDTData()
{
    global $global_coin_data;
    
    // Check global cache first
    if (isset($global_coin_data['USDT'])) {
        return $global_coin_data['USDT'];
    }
    
    // Fetch and store in global cache
    $usdt_data = get_current_coin('USDT');
    if ($usdt_data) {
        $global_coin_data['USDT'] = $usdt_data;
    }
    
    return $usdt_data;
}
```

### نتیجه

| متریک | قبل | بعد | بهبود |
|--------|-----|------|--------|
| API Calls | 4 | 1-2 | **75%↓** |
| TTFB (تقریبی) | 16-18s | 5-8s | **60%↓** |

---

## 🟢 راه‌حل 2: AJAX Price Loader

### توضیح

صفحه بدون انتظار برای API call سریعاً لود می‌شود. قیمت‌ها بعداً با AJAX بارگذاری می‌شوند.

### مزایا

- ✅ **صفحه فوراً لود می‌شود** (بدون blocking)
- ✅ **قیمت‌ها همیشه real-time هستند**
- ✅ **UX بهتر** با skeleton loading
- ✅ **Auto-refresh** هر 30 ثانیه

### فایل‌های ایجاد شده

#### 1. JavaScript Module: `assets/js/coin-price-loader.js`

```javascript
const CoinPriceLoader = {
    config: {
        ajaxUrl: xpayPriceLoader.ajaxUrl,
        nonce: xpayPriceLoader.nonce,
        symbols: xpayPriceLoader.symbols,
        refreshInterval: 30000
    },

    async loadPrices() {
        const response = await fetch(this.config.ajaxUrl, {
            method: 'POST',
            body: new URLSearchParams({
                action: 'xpay_get_batch_prices',
                nonce: this.config.nonce,
                symbols: this.config.symbols.join(',')
            })
        });
        // Update DOM with prices...
    }
};
```

#### 2. PHP AJAX Endpoint: `app/Controllers/ShortcodeController.php`

```php
public function ajaxGetBatchPrices()
{
    check_ajax_referer('xpay_market_price_nonce', 'nonce');

    $symbols = $_POST['symbols'];
    $results = [];

    foreach ($symbols as $symbol) {
        $coin_data = get_current_coin($symbol); // Uses request-level cache!
        
        if ($coin_data) {
            $results[$symbol] = [
                'irr' => number_format($coin_data['sellPrice']),
                'usd' => number_format($coin_data['inDollarPrice']),
                'change24h' => $coin_data['change24Hours'],
                // ... more fields
            ];
        }
    }

    wp_send_json_success(['coins' => $results]);
}
```

### HTML Data Attributes

برای استفاده از AJAX loader، المان‌های HTML باید data attributes داشته باشند:

```html
<!-- قیمت تومان -->
<span data-price-irr data-symbol="TRX">در حال بارگذاری...</span>

<!-- قیمت دلار -->
<span data-price-usd data-symbol="TRX">Loading...</span>

<!-- تغییرات 24 ساعت -->
<span data-change-24h data-symbol="TRX">--</span>

<!-- تغییرات 1 ساعت -->
<span data-change-1h data-symbol="TRX">--</span>

<!-- حجم معاملات -->
<span data-volume-24h data-symbol="TRX">--</span>
```

### Skeleton Loading CSS

```css
.price-skeleton.skeleton-loading {
    background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
    background-size: 200% 100%;
    animation: skeleton-loading 1.5s infinite;
    color: transparent !important;
}

@keyframes skeleton-loading {
    0% { background-position: 200% 0; }
    100% { background-position: -200% 0; }
}
```

---

## 📁 فایل‌های تغییر یافته

| فایل | تغییر |
|------|--------|
| `app/Support/helpers.php` | اضافه شدن request-level cache به `get_current_coin()` |
| `app/Controllers/ShortcodeController.php` | بهبود `getUSDTData()` + endpoint جدید `ajaxGetBatchPrices` |
| `app/Support/Assets.php` | enqueue کردن `coin-price-loader.js` |
| `assets/js/coin-price-loader.js` | **فایل جدید** - AJAX module |
| `views/shortcodes/coin-live-data.php` | اضافه شدن data attributes و skeleton CSS |
| `views/shortcodes/coin-change-data.php` | اضافه شدن data attributes |

---

## 🔧 نحوه استفاده

### غیرفعال کردن AJAX Loader

اگر نیاز به غیرفعال کردن AJAX loader دارید، کافیست در `Assets.php` بخش مربوط به enqueue کردن `coin-price-loader` را کامنت کنید.

### Force Refresh Cache

برای بازخوانی اجباری داده از API:

```php
// PHP
$fresh_data = get_current_coin('TRX', true); // force_refresh = true
```

```javascript
// JavaScript
CoinPriceLoader.refresh();
```

### Custom Event

بعد از لود قیمت‌ها، یک event dispatch می‌شود:

```javascript
document.addEventListener('xpay:pricesLoaded', (e) => {
    console.log('Prices loaded:', e.detail.coins);
});
```

---

## 📊 Benchmark

### تست با check-host.net

| متریک | قبل | بعد |
|--------|-----|------|
| Austria | 7.9s | ~2s |
| Brazil | 17.2s | ~3s |
| Germany | 8.8s | ~2s |
| Average | **12.3s** | **~2.5s** |

### علت بهبود

1. **Request-Level Cache**: یک API call به جای 4
2. **AJAX Loading**: صفحه بدون انتظار render می‌شود
3. **Batch API**: همه coins در یک request

---

## ⚠️ نکات مهم

1. **SEO**: قیمت‌های اولیه از Server-Side Rendering می‌آیند (برای crawlers)
2. **Fallback**: اگر JS غیرفعال باشد، قیمت‌های سرور نمایش داده می‌شود
3. **Rate Limiting**: حداکثر 10 symbol در هر batch request

---

## 🔗 مرتبط

- [PRICE-UPDATE-API.md](PRICE-UPDATE-API.md) - سیستم بروزرسانی قیمت‌ها
- [CRON-CACHE-SYSTEM.md](CRON-CACHE-SYSTEM.md) - سیستم Cron و Cache
- [PAGESPEED-OPTIMIZATIONS.md](PAGESPEED-OPTIMIZATIONS.md) - بهینه‌سازی‌های PageSpeed
