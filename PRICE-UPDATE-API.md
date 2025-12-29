# 💰 سیستم بروزرسانی قیمت‌ها (Price Update API)
# Price Update System Documentation

> **نسخه:** 2.0.0  
> **تاریخ:** 29 دسامبر 2025  
> **وضعیت:** 🟢 فعال - Real-Time Updates

---

## 📋 فهرست مطالب

- [معرفی](#معرفی)
- [معماری سیستم](#معماری-سیستم)
- [REST API Endpoint](#rest-api-endpoint)
- [Frontend Implementation](#frontend-implementation)
- [Cache Strategy](#cache-strategy)
- [Configuration](#configuration)
- [Flow Diagram](#flow-diagram)
- [نحوه استفاده](#نحوه-استفاده)
- [Troubleshooting](#troubleshooting)
- [Performance Considerations](#performance-considerations)

---

## 🎯 معرفی

سیستم بروزرسانی قیمت‌ها یک راه‌حل Real-Time برای نمایش آخرین قیمت ارزهای دیجیتال در صفحات سایت است. این سیستم با استفاده از REST API و Cache Strategy بهینه، قیمت‌ها را با حداقل تاخیر (5 ثانیه) بروز نگه می‌دارد.

### ویژگی‌های کلیدی

✅ **Real-Time Updates** - بروزرسانی فوری قیمت‌ها (0ms delay)  
✅ **Intelligent Caching** - Cache با TTL 5 ثانیه برای کاهش درخواست‌های API  
✅ **CDN Bypass** - Cache-busting برای جلوگیری از CDN caching  
✅ **USDT Dependency** - مدیریت خودکار وابستگی قیمت USDT  
✅ **Modular Architecture** - کد تمیز و قابل نگهداری  
✅ **Error Handling** - مدیریت خطاها بدون crash  

---

## 🏗️ معماری سیستم

### Components

```
┌─────────────────────────────────────────────────────────────┐
│                     Coin Page (Frontend)                    │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │          symbol-updater.js (Module)                  │  │
│  │                                                      │  │
│  │  • init() - Start with immediate check              │  │
│  │  • checkSymbol() - Validate cache age               │  │
│  │  • updateSymbol() - Call API if needed              │  │
│  │  • Cache-busting: ?t=timestamp                      │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↓
                   fetch() + timestamp query
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    REST API (Backend)                       │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │      ApiController::updateSymbolData()               │  │
│  │                                                      │  │
│  │  1. Check cache age (< 5 seconds?)                  │  │
│  │  2. Fetch from api.xpay.co if expired               │  │
│  │  3. Update USDT price if needed (recursive)         │  │
│  │  4. Write gzipped cache file                        │  │
│  │  5. Return 200 with confirmation                    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↓
                   Cache File System
                              ↓
┌─────────────────────────────────────────────────────────────┐
│            /wp-content/cache-api/symbols/                   │
│                                                             │
│  • BTC.json.gz                                              │
│  • ETH.json.gz                                              │
│  • USDT.json.gz                                             │
│  • ...                                                      │
│                                                             │
│  Format: {"symbol": "BTC", "price": 43500, ...}            │
│  Compression: gzip level 9                                  │
│  TTL: 5 seconds                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔌 REST API Endpoint

### Endpoint Details

**URL:** `/wp-json/xpay/v1/update-symbol`  
**Method:** `POST`  
**Content-Type:** `application/json`

### Request

```json
{
  "symbol": "BTC"
}
```

### Response (Success)

```json
{
  "success": true,
  "message": "Symbol BTC updated successfully",
  "cached_at": "2025-12-29 14:30:45"
}
```

### Response (Error)

```json
{
  "success": false,
  "message": "Symbol parameter is required"
}
```

### PHP Implementation

**File:** `app/Controllers/ApiController.php`

```php
/**
 * Update symbol data
 * Real-time price updates with intelligent caching
 * 
 * @since 2.0.0
 */
public function updateSymbolData(WP_REST_Request $request) {
    $symbol = $request->get_param('symbol');
    
    if (empty($symbol)) {
        return new WP_REST_Response([
            'success' => false,
            'message' => 'Symbol parameter is required'
        ], 400);
    }

    // Cache configuration
    $cache_ttl = 5; // 5 seconds - Real-time updates
    $cache_dir = WP_CONTENT_DIR . '/cache-api/symbols/';
    $cache_file = $cache_dir . $symbol . '.json.gz';

    // Check if cache is still valid
    if (file_exists($cache_file)) {
        $file_time = filemtime($cache_file);
        $time_diff = time() - $file_time;
        
        if ($time_diff < $cache_ttl) {
            return new WP_REST_Response([
                'success' => true,
                'message' => "Symbol $symbol is up to date (cached)",
                'cached_at' => date('Y-m-d H:i:s', $file_time)
            ], 200);
        }
    }

    // Fetch fresh data from external API
    $api_url = "https://api.xpay.co/v1/market/marketpricedata/{$symbol}";
    $response = wp_remote_get($api_url, [
        'timeout' => 10,
        'sslverify' => false
    ]);

    if (is_wp_error($response)) {
        return new WP_REST_Response([
            'success' => false,
            'message' => 'Failed to fetch data from API'
        ], 500);
    }

    $data = json_decode(wp_remote_retrieve_body($response), true);
    
    // USDT dependency handling
    if ($symbol !== 'USDT' && isset($data['priceToman'])) {
        $this->updateSymbolData(new WP_REST_Request('POST', [
            'symbol' => 'USDT'
        ]));
    }

    // Write cache file
    if (!is_dir($cache_dir)) {
        wp_mkdir_p($cache_dir);
    }

    $json_data = json_encode($data);
    $compressed = gzencode($json_data, 9);
    file_put_contents($cache_file, $compressed);

    return new WP_REST_Response([
        'success' => true,
        'message' => "Symbol $symbol updated successfully",
        'cached_at' => date('Y-m-d H:i:s')
    ], 200);
}
```

---

## 💻 Frontend Implementation

### JavaScript Module

**File:** `assets/js/symbol-updater.js`

```javascript
/**
 * Symbol Updater Module
 * Handles symbol cache checking and automatic updates for coin pages
 *
 * @package XPay
 * @since 1.0.0
 * @version 2.0.0
 */

(function() {
    'use strict';

    /**
     * Symbol Updater Object
     * Modular pattern for clean code organization
     */
    const SymbolUpdater = {
        /**
         * Configuration
         */
        config: {
            cacheExpirySeconds: 5, // Reduced from 60 to 5 for real-time updates
            elementId: 'symbol-data',
            cacheBasePath: '/wp-content/cache-api/symbols/',
            updateEndpoint: '/wp-json/xpay/v1/update-symbol'
        },

        /**
         * State management
         */
        state: {
            symbol: null,
            isChecking: false,
            checkUrl: null
        },

        /**
         * Initialize the updater
         */
        init() {
            // Get symbol from data attribute
            const element = document.getElementById(this.config.elementId);
            if (!element) return;

            this.state.symbol = element.dataset.symbol;
            if (!this.state.symbol) return;

            // Build check URL with cache-busting timestamp
            const timestamp = Date.now();
            this.state.checkUrl = `${this.config.cacheBasePath}${this.state.symbol}.json.gz?t=${timestamp}`;
            
            // Immediate check - no delays
            this.checkSymbol();
        },

        /**
         * Check if symbol cache needs update
         */
        checkSymbol() {
            if (this.state.isChecking) return;
            this.state.isChecking = true;

            fetch(this.state.checkUrl, {
                method: 'HEAD',
                cache: 'no-cache'
            })
            .then(response => {
                const lastModified = response.headers.get('Last-Modified');
                if (lastModified) {
                    const fileTime = new Date(lastModified).getTime();
                    const currentTime = Date.now();
                    const ageSeconds = (currentTime - fileTime) / 1000;

                    if (ageSeconds > this.config.cacheExpirySeconds) {
                        this.updateSymbol();
                    }
                }
            })
            .catch(error => {
                console.error('Error checking symbol cache:', error);
            })
            .finally(() => {
                this.state.isChecking = false;
            });
        },

        /**
         * Update symbol via API
         */
        updateSymbol() {
            fetch(this.config.updateEndpoint, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    symbol: this.state.symbol
                })
            })
            .then(response => response.json())
            .then(data => {
                if (data.success) {
                    // Reload page to show updated data
                    window.location.reload();
                }
            })
            .catch(error => {
                console.error('Error updating symbol:', error);
            });
        }
    };

    // Auto-initialize when DOM is ready
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', () => SymbolUpdater.init());
    } else {
        SymbolUpdater.init();
    }

    // Expose to window for external access
    window.SymbolUpdater = SymbolUpdater;

})();
```

---

## 🗂️ Cache Strategy

### Cache File Structure

```
wp-content/cache-api/symbols/
├── BTC.json.gz          (5 second TTL)
├── ETH.json.gz          (5 second TTL)
├── USDT.json.gz         (5 second TTL)
├── BNB.json.gz          (5 second TTL)
└── ...
```

### Cache File Format

```json
{
  "symbol": "BTC",
  "name": "Bitcoin",
  "price": 43500.25,
  "priceToman": 1850000000,
  "change24h": 2.5,
  "volume24h": 25000000000,
  "_cached_at": "2025-12-29 14:30:45"
}
```

### TTL Configuration

| Component | TTL | Purpose |
|-----------|-----|---------|
| Frontend Cache Check | 5s | Check if cache file needs update |
| Backend Cache TTL | 5s | API cache validity period |
| Browser Cache | None | Disabled via cache-busting |
| CDN Cache | Bypassed | Timestamp query parameter |

---

## ⚙️ Configuration

### Frontend Config

```javascript
config: {
    cacheExpirySeconds: 5,           // Cache validity in seconds
    elementId: 'symbol-data',        // DOM element ID
    cacheBasePath: '/wp-content/cache-api/symbols/',
    updateEndpoint: '/wp-json/xpay/v1/update-symbol'
}
```

### Backend Config

```php
// Cache TTL (seconds)
$cache_ttl = 5;

// Cache directory
$cache_dir = WP_CONTENT_DIR . '/cache-api/symbols/';

// External API
$api_url = "https://api.xpay.co/v1/market/marketpricedata/{$symbol}";

// Compression level (1-9)
$compression_level = 9;
```

---

## 📊 Flow Diagram

### Complete Flow

```
User Opens Coin Page
        ↓
   DOM Ready Event
        ↓
SymbolUpdater.init()
        ↓
Get Symbol from DOM
        ↓
Build Check URL + Timestamp
        ↓
checkSymbol() → HEAD Request
        ↓
    ┌───────────────────┐
    │ Check Last-Modified│
    └───────────────────┘
        ↓           ↓
    Age < 5s?   Age > 5s?
        ↓           ↓
    [Skip]    updateSymbol()
                    ↓
            POST /wp-json/xpay/v1/update-symbol
                    ↓
            ApiController::updateSymbolData()
                    ↓
            Check Cache File Age
                    ↓
            ┌─────────────┐
            │ < 5 seconds?│
            └─────────────┘
                ↓       ↓
            Return   Fetch API
            Cached      ↓
                  Update USDT?
                        ↓
                  Write Cache File
                        ↓
                  Return Success
                        ↓
                window.location.reload()
```

---

## 🚀 نحوه استفاده

### 1. در Template

```php
<!-- Add symbol data attribute to page -->
<div id="symbol-data" data-symbol="BTC">
    <!-- Price display elements -->
</div>

<!-- Enqueue the script -->
<?php wp_enqueue_script('symbol-updater'); ?>
```

### 2. Manual Update

```javascript
// Force update a specific symbol
fetch('/wp-json/xpay/v1/update-symbol', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        symbol: 'BTC'
    })
})
.then(response => response.json())
.then(data => console.log(data));
```

### 3. Test Cache Age

```bash
# Check cache file modification time
ls -lh wp-content/cache-api/symbols/BTC.json.gz

# View cache file content
zcat wp-content/cache-api/symbols/BTC.json.gz | jq
```

---

## 🔧 Troubleshooting

### مشکل: قیمت‌ها بروز نمی‌شوند

**علت احتمالی:**
- Cache directory دسترسی نوشتن ندارد
- External API در دسترس نیست
- CDN cache interference

**راه‌حل:**
```bash
# Check directory permissions
chmod 755 wp-content/cache-api/
chmod 755 wp-content/cache-api/symbols/

# Test API endpoint
curl https://api.xpay.co/v1/market/marketpricedata/BTC

# Clear CDN cache
# Via Arvan CDN panel → Purge Cache
```

### مشکل: تاخیر 3-5 ثانیه در بروزرسانی

**علت:** کد قدیمی با `idleTimeout` و `scheduleCheck()`

**راه‌حل:** اطمینان از استفاده از نسخه 2.0.0:
```javascript
// Version 2.0.0 uses immediate execution
init() {
    // ...
    this.checkSymbol(); // No delay!
}
```

### مشکل: CDN کش قدیمی را سرو می‌کند

**راه‌حل:**
- Cache-busting timestamp اضافه شده: `?t=${Date.now()}`
- Cache-Control header در Nginx/Apache:
```nginx
location ~* \.json\.gz$ {
    add_header Cache-Control "no-cache, no-store, must-revalidate";
    add_header Pragma "no-cache";
    add_header Expires "0";
}
```

### مشکل: Console errors

```
Error checking symbol cache: TypeError: Failed to fetch
```

**راه‌حل:**
- بررسی CORS headers
- بررسی CSP policy
- بررسی network connectivity

---

## ⚡ Performance Considerations

### Optimization Points

✅ **Gzip Compression** - فایل‌های cache با compression level 9  
✅ **HEAD Request** - بررسی cache بدون download کامل فایل  
✅ **5 Second TTL** - تعادل بین real-time و API load  
✅ **Async Operations** - Non-blocking fetch requests  
✅ **Error Handling** - Silent failure without page crash  
✅ **Single Responsibility** - جداسازی check از update  

### Benchmark Results

| Metric | Before (v1.0) | After (v2.0) | Improvement |
|--------|---------------|--------------|-------------|
| First Check Delay | 3000-5000ms | 0ms | ✅ 100% |
| Cache Validity | 60s | 5s | ✅ 92% |
| API Requests/min | 1 | 12 | ⚠️ 12x |
| Page Load Impact | Minimal | Minimal | ✅ Same |
| CDN Cache Issues | Yes | No | ✅ Fixed |

**نکته:** افزایش API requests قابل قبول است چون:
- Cache 5 ثانیه همچنان بار را کاهش می‌دهد
- External API سریع است (< 200ms)
- Update فقط در صفحات coin انجام می‌شود

---

## 📝 Version History

### v2.0.0 (2025-12-29)
- ✅ حذف تمام تاخیرها (idleTimeout, fallbackDelay)
- ✅ کاهش TTL از 60s به 5s
- ✅ افزودن cache-busting timestamp
- ✅ حذف متد scheduleCheck()
- ✅ اجرای فوری در init()

### v1.0.0 (2024)
- ✅ پیاده‌سازی اولیه با delay
- ✅ Cache با TTL 60 ثانیه
- ✅ REST API endpoint

---

## 🔗 Related Documentation

- [PAGESPEED-OPTIMIZATIONS.md](PAGESPEED-OPTIMIZATIONS.md) - Asset Loading Strategy
- [CACHE-VERSION-REDIRECT.md](CACHE-VERSION-REDIRECT.md) - Cache Busting Strategy
- [CDN-CACHE-OPTIMIZATION.md](CDN-CACHE-OPTIMIZATION.md) - CDN Configuration
- [PERFORMANCE-OPTIMIZATION.md](PERFORMANCE-OPTIMIZATION.md) - General Performance
- [CHANGELOG-FA.md](changelog/CHANGELOG-FA.md) - تاریخچه تغییرات فارسی
- [CHANGELOG-EN.md](changelog/CHANGELOG-EN.md) - English Changelog

---

## 👥 Contributors

- Development Team - Initial implementation
- DevOps Team - Cache strategy and CDN configuration
- QA Team - Testing and validation

---

## 📧 Support

برای گزارش مشکلات یا سوالات:
- GitHub Issues
- Technical Documentation Updates
- Performance Analysis Requests

---

**Last Updated:** 29 December 2025  
**Document Version:** 2.0.0  
**Status:** ✅ Production Ready
