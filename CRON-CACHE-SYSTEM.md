# 🔄 سیستم Cron و Cache برای قیمت ارزهای دیجیتال
# Cron-Based Cache System for Cryptocurrency Prices

> **نسخه:** 1.0.0  
> **تاریخ:** ژانویه 2026  
> **وضعیت:** 🟢 فعال و عملیاتی

---

## 📋 خلاصه تغییرات

این مستند شامل تغییرات اساسی در سیستم caching و بروزرسانی قیمت‌های ارز دیجیتال است که برای بهبود عملکرد و حذف race conditions پیاده‌سازی شده است.

### مشکلات قبلی:
- ❌ بررسی TTL در هر درخواست API
- ❌ Race condition هنگام نوشتن فایل cache
- ❌ وابستگی به JavaScript برای نمایش قیمت‌های popular coins
- ❌ بارگذاری چندباره `processMarketData()` در یک صفحه

### راه‌حل‌های جدید:
- ✅ سیستم Cron-based با Action Scheduler
- ✅ Atomic File Write برای جلوگیری از race condition
- ✅ Server-Side Rendering برای popular coins
- ✅ یکبار فراخوانی `processMarketData()` و استفاده مجدد

---

## 🏗️ معماری جدید

### 1. Batch Update System

```
┌─────────────────────────────────────────────────────────────┐
│                    Action Scheduler                          │
├─────────────────────────────────────────────────────────────┤
│  Batch 1 (00s) → Batch 2 (06s) → ... → Batch 10 (54s)       │
│     ↓              ↓                        ↓                │
│  ~10 coins     ~10 coins               ~10 coins             │
│     ↓              ↓                        ↓                │
│  Update individual symbol cache files                        │
│     ↓              ↓                        ↓                │
│  Update central all-symbols.json.gz                          │
└─────────────────────────────────────────────────────────────┘
```

### 2. Cache Architecture

```
wp-content/cache-api/
├── all-symbols.json.gz      # Central cache (updated every 5 min by cron)
├── symbols/
│   ├── BTC.json.gz          # Individual symbol cache
│   ├── ETH.json.gz
│   ├── USDT.json.gz
│   └── ...
└── coingecko-symbol.json.gz # External data
```

---

## 📁 فایل‌های تغییر یافته

### 1. ApiController.php

#### تغییرات اصلی:
- حذف بررسی TTL از `updateSymbolData()`
- `fetchMarketPrice()` فقط از cache می‌خواند
- `getAllCoins()` فقط از cache می‌خواند
- اضافه شدن `updateAllSymbolsCache()` برای cron

#### کد کلیدی - Atomic File Write:

```php
// قبلاً (مشکل race condition):
file_put_contents($cache_file, gzencode(json_encode($data), 9));

// جدید (atomic و safe):
atomic_file_put_contents($cache_file, gzencode(json_encode($data), 9));
```

### 2. CronController.php

#### تغییرات اصلی:
- اضافه شدن `$total_batches = 10`
- انتقال `registerActionCallbacks()` به constructor
- فیلترهای Action Scheduler برای concurrent batches
- متد جدید `updateAllSymbols()` برای بروزرسانی 5 دقیقه‌ای

#### کد کلیدی - Action Scheduler Configuration:

```php
// Allow concurrent batches
add_filter('action_scheduler_queue_runner_concurrent_batches', function () {
    return 5; // 5 concurrent batches
});

// Increase batch size
add_filter('action_scheduler_queue_runner_batch_size', function () {
    return 50;
});

// Reduce queue check interval
add_filter('action_scheduler_run_schedule', function ($interval) {
    return 10; // Check every 10 seconds
});
```

### 3. helpers.php

#### تابع جدید - atomic_file_put_contents():

```php
/**
 * Write content to file atomically to prevent race conditions
 * 
 * This function writes to a temporary file first, then renames it to the
 * target file. This ensures that readers never see a partial/empty file.
 */
function atomic_file_put_contents($file_path, $content)
{
    $dir = dirname($file_path);
    
    if (!file_exists($dir)) {
        wp_mkdir_p($dir);
    }
    
    // Create temp file in same directory (ensures same filesystem for rename)
    $temp_file = $dir . '/.tmp_' . basename($file_path) . '_' . uniqid();
    
    // Write to temp file
    $result = file_put_contents($temp_file, $content, LOCK_EX);
    
    if ($result === false) {
        @unlink($temp_file);
        return false;
    }
    
    // Atomic rename - this is the key to preventing race conditions
    if (!rename($temp_file, $file_path)) {
        @unlink($temp_file);
        return false;
    }
    
    return true;
}
```

### 4. MarketDataController.php

#### تغییرات اصلی:
- اضافه شدن `getAllPositiveGrowth()` برای همه کوین‌های مثبت
- اضافه شدن `getMostExpensiveFromPositive()` برای گرون‌ترین‌ها از مثبت‌ها
- `getPopularCoins()` با `array_values()` برای reindex

#### کد کلیدی:

```php
public function getProcessedMarketData()
{
    $headerMenu = $this->getHeaderMenu();
    $listCoins = $this->getCoinsData();

    if (empty($listCoins)) {
        return $this->getEmptyData($headerMenu, $listCoins);
    }

    // Get all positive growth coins sorted by change24Hours
    $allPositiveGrowth = $this->getAllPositiveGrowth($listCoins);

    return [
        'positiveGrowth' => array_slice($allPositiveGrowth, 0, 4),
        'negativeGrowth' => $this->getTopNegativeGrowth($listCoins),
        'popularCRRS' => $this->getPopularCoins($listCoins),
        'mostExpensive' => $this->getMostExpensiveFromPositive($allPositiveGrowth),
        'topGrowth' => $this->getTopGrowth($listCoins),
        'headerMenu' => $headerMenu,
        'listCoins' => $listCoins,
    ];
}
```

---

## 🎨 Template های جدید

### 1. popular-coin.php (جدید)

قیمت USDT, TRX, BTC, ETH را از PHP می‌خواند و Server-Side Render می‌کند.

**مسیر:** `templates/coins/popular-coin.php`

**ویژگی‌ها:**
- بدون وابستگی به JavaScript/AJAX
- SSR برای SEO بهتر
- Fallback به `listCoins` اگر کوینی در `popularCRRS` نباشد

```php
<?php
// Get popular coins from processMarketData
if (isset($processMarketData['popularCRRS']) && !empty($processMarketData['popularCRRS'])) {
    $popularCoins = $processMarketData['popularCRRS'];
}

// If popularCRRS is missing some coins, get them from listCoins
if (isset($processMarketData['listCoins']) && !empty($processMarketData['listCoins'])) {
    $popularSymbols = ['USDT', 'TRX', 'BTC', 'ETH'];
    // ... fill missing coins
}
?>

<section class="swiper currencies-status-box">
    <?php foreach ($orderedSymbols as $symbol) : ?>
        <!-- Render coin with price and change -->
    <?php endforeach; ?>
</section>
```

### 2. growth-coin.php (آپدیت)

**تغییرات:**
- استفاده از `positiveGrowth` بجای `topGrowth` برای "بیشترین سود امروز"
- Initialize کردن متغیرها قبل از استفاده
- محاسبه `mostExpensive` فقط از کوین‌های مثبت

```php
// Initialize variables
$topGrowth = [];
$negativeGrowth = [];
$mostExpensive = [];
$headerMenu = [];

// Use positiveGrowth for "Most Profit Today"
if (isset($processMarketData['positiveGrowth'])) {
    $topGrowth = $processMarketData['positiveGrowth'];
}

// mostExpensive from positive coins only
if (isset($processMarketData['positiveGrowth'])) {
    $positiveCoins = $processMarketData['positiveGrowth'];
    usort($positiveCoins, fn($a, $b) => ($b['sellPrice'] ?? 0) <=> ($a['sellPrice'] ?? 0));
    $mostExpensive = array_slice($positiveCoins, 0, 4);
}
```

---

## 📄 View های آپدیت شده

### 1. home.php

**قبلاً:**
```php
<!-- 150+ lines of hardcoded HTML for USDT, TRX, BTC, ETH -->
<section class="swiper currencies-status-box">
    <div class="swiper-slide crr-box" data-symbol="USDT">
        <!-- Empty price, filled by JS -->
        <p class="price"></p>
        <span class="change"></span>
    </div>
    ...
</section>

<?php get_template_part('templates/coins/growth', 'coin', ['processMarketData' => processMarketData()]); ?>
```

**جدید:**
```php
<?php
// Get market data once and reuse
$marketData = processMarketData();

get_template_part('templates/coins/popular', 'coin', ['processMarketData' => $marketData]);
get_template_part('templates/coins/growth', 'coin', ['processMarketData' => $marketData]);
?>
```

### 2. coin.php (Archive)

**همان تغییرات home.php** - استفاده از template جدید و یکبار فراخوانی `processMarketData()`

---

## 🔧 تغییرات JavaScript

### app.js

**تغییرات:**
- کامنت شدن `populateSectionCrr("#currencies-status-items", popularCRRS)`
- بهبود error handling برای API response

```javascript
function processAndPopulateData(data) {
    const { positiveGrowth, negativeGrowth, popularCRRS, mostExpensive } = processMarketData(data);

    // Now handled by PHP (popular-coin.php)
    // populateSectionCrr("#currencies-status-items", popularCRRS);

    populateSIngleMomentValue("#moment-value-of-coins-items", popularCRRS);
    // ...
}
```

---

## 📊 جدول زمانی Cron Jobs

| Job | Interval | Description |
|-----|----------|-------------|
| `xpay_batch_update_0` | 60s | Update batch 0 (first ~10 coins) |
| `xpay_batch_update_1` | 60s + 6s | Update batch 1 |
| ... | ... | ... |
| `xpay_batch_update_9` | 60s + 54s | Update batch 9 (last ~10 coins) |
| `xpay_update_all_symbols` | 300s | Update central cache |

---

## ✅ مزایا

### Performance
- 🚀 **SSR**: قیمت‌ها در HTML اولیه موجود (بهتر برای SEO)
- 🚀 **کاهش AJAX**: حذف درخواست برای popular coins
- 🚀 **یکبار فراخوانی**: `processMarketData()` فقط یکبار صدا زده می‌شود

### Reliability
- 🔒 **Atomic Write**: هیچوقت فایل خالی یا نیمه‌کاره خوانده نمی‌شود
- 🔒 **Race Condition Free**: کاربران همیشه داده کامل می‌بینند
- 🔒 **Fallback**: اگر کوینی در cache نباشد، از listCoins گرفته می‌شود

### Maintainability
- 📦 **Modular**: هر بخش در template جداگانه
- 📦 **Reusable**: `$marketData` یکبار ایجاد و چندبار استفاده
- 📦 **Clean Code**: حذف 150+ خط HTML تکراری از view ها

---

## 🐛 مشکلات رفع شده

| مشکل | علت | راه‌حل |
|------|------|--------|
| دیتا خالی هنگام رفرش | Race condition در file write | `atomic_file_put_contents()` |
| "بیشترین سود" با مقدار منفی | استفاده از `topGrowth` بجای `positiveGrowth` | فیلتر فقط مثبت‌ها |
| "برترین بازار" با کوین منفی | ترکیب مثبت و منفی در `getMostExpensive` | فقط از مثبت‌ها |
| BTC در لیست نیست | `array_filter` ایندکس‌ها را حفظ می‌کند | `array_values()` برای reindex |
| لودر نمایش داده می‌شود | JS منتظر AJAX است | SSR بدون لودر |
| Action Scheduler callback error | ثبت callback بعد از schedule | انتقال به constructor |

---

## 🧪 تست

### تست Atomic Write:
```bash
# چک کنید که فایل‌های temp وجود ندارند
docker exec wp-app ls -la /var/www/html/wp-content/cache-api/
# نباید فایلی با پیشوند .tmp_ وجود داشته باشد
```

### تست Cache Content:
```bash
# چک کنید که cache خالی نیست
docker exec wp-app php -r \
  'echo count(json_decode(gzdecode(file_get_contents("/var/www/html/wp-content/cache-api/all-symbols.json.gz")), true)["value"]);'
# باید عدد > 0 برگرداند
```

### تست Popular Coins:
در browser، صفحه اصلی یا /coin/ را باز کنید و با View Source چک کنید که قیمت‌ها در HTML هستند (نه خالی).

---

## 📚 فایل‌های مرتبط

- `app/Controllers/ApiController.php`
- `app/Controllers/CronController.php`
- `app/Controllers/MarketDataController.php`
- `app/Support/helpers.php`
- `templates/coins/popular-coin.php`
- `templates/coins/growth-coin.php`
- `views/pages/home.php`
- `views/archives/coin.php`
- `assets/js/app.js`

---

## 🔗 مستندات مرتبط

- [PRICE-UPDATE-API.md](PRICE-UPDATE-API.md) - مستندات API قیمت‌ها
- [PERFORMANCE-OPTIMIZATION.md](PERFORMANCE-OPTIMIZATION.md) - بهینه‌سازی عملکرد
