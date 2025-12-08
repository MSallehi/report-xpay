# CDN Configuration System

## Overview
All CDN URLs for coin images are now centrally managed through the `.env` file, allowing easy changes to CDN location and image format without modifying code.

## Configuration

### .env Variables
```env
CDN_XPAY_URL=https://cdn.xpay.co/coins/webp/
CDN_XPAY_FORMAT=webp
```

**Change CDN URL or format:** Edit these values in `.env` and the entire site will use the new configuration.

## Usage

### PHP Templates
Use the `cdn_url()` helper function:

```php
<!-- Single coin -->
<img src="<?php echo cdn_url('BTC'); ?>" alt="Bitcoin">

<!-- Dynamic symbol -->
<img src="<?php echo cdn_url($coin_symbol); ?>" alt="<?php echo $coin_name; ?>">

<!-- With esc_attr -->
<img src="<?php echo cdn_url(esc_attr($item['symbol'])); ?>" alt="coin">
```

### React/JavaScript Components
Use the `getCdnUrl()` function (automatically defined in components):

```jsx
// The config is available via window.xpayConfig
const { cdnBaseUrl, cdnFormat } = window.xpayConfig;

// Helper function (already added to components)
const getCdnUrl = (coinSymbol) => {
  const base = cdnBaseUrl.replace(/\/$/, "");
  return `${base}/${coinSymbol.toUpperCase()}.${cdnFormat}`;
};

// Usage in JSX
<img src={getCdnUrl(coin.symbol)} alt={coin.name} />
```

## Implementation Details

### Backend (PHP)

#### 1. Env Class (`app/Support/Env.php`)
```php
public static function getCdnBaseUrl(): string
{
    return self::get('CDN_XPAY_URL', 'https://cdn.xpay.co/coins/webp/');
}

public static function getCdnFormat(): string
{
    return self::get('CDN_XPAY_FORMAT', 'webp');
}

public static function getCdnUrl(string $symbol, bool $uppercase = true): string
{
    $baseUrl = rtrim(self::getCdnBaseUrl(), '/');
    $format = self::getCdnFormat();
    $coinSymbol = $uppercase ? strtoupper($symbol) : $symbol;
    
    return "{$baseUrl}/{$coinSymbol}.{$format}";
}
```

#### 2. Global Helper Function (`functions.php`)
```php
function cdn_url(string $symbol, bool $uppercase = true): string
{
    return Env::getCdnUrl($symbol, $uppercase);
}
```

#### 3. JavaScript Configuration (`app/Support/Assets.php`)
```php
wp_localize_script('main-script', 'xpayConfig', array(
    'cdnBaseUrl' => Env::getCdnBaseUrl(),
    'cdnFormat'  => Env::getCdnFormat(),
));
```

### Frontend (JavaScript/React)

The CDN configuration is automatically exposed to JavaScript via `window.xpayConfig`:

```javascript
window.xpayConfig = {
    cdnBaseUrl: "https://cdn.xpay.co/coins/webp/",
    cdnFormat: "webp"
}
```

## Updated Files

### Configuration Files
- ✅ `.env` - Added CDN_XPAY_URL and CDN_XPAY_FORMAT
- ✅ `app/Support/Env.php` - Added getCdnUrl(), getCdnBaseUrl(), getCdnFormat()
- ✅ `functions.php` - Added cdn_url() helper function
- ✅ `app/Support/Assets.php` - Added xpayConfig to wp_localize_script

### PHP Template Files (19 replacements)
- ✅ `header.php` - 8 replacements (BTC, USDT, UUSD, TRX in mobile & desktop menus)
- ✅ `views/pages/calculator.php` - 4 replacements (USDT, TRX, BTC, ETH)
- ✅ `views/shortcodes/coin-live-data.php` - 1 replacement
- ✅ `views/shortcodes/buy-coin.php` - 1 replacement
- ✅ `templates/blog/currency-blog.php` - 1 replacement
- ✅ `templates/coins/calculator-coin.php` - 2 replacements
- ✅ `templates/coins/list-coin.php` - 2 replacements
- ✅ `templates/coins/growth-coin.php` - 3 replacements

### React JSX Components (11 replacements)
- ✅ `src/components/CoinCalculator.jsx` - 4 replacements
  - selectedBuyCoin image
  - Buy dropdown options
  - selectedSellCoin image
  - Sell dropdown options
- ✅ `src/components/CoinChart.jsx` - 1 replacement (UUSD chart header)
- ✅ `src/components/CoinCalculator-old.jsx` - 4 replacements (legacy component)

### Compiled JavaScript Files
⚠️ **Action Required:** After JSX changes, rebuild React components:
```bash
npm run build
```

This will update the compiled files:
- `assets/js/app-calculator.v4.9.6.js`
- `assets/js/app-chart.js`
- `assets/js/app.v4.9.6.js`

## Migration Path

### Old Format
```php
<img src="https://cdn.xpay.co/coins/webp/BTC.webp" alt="Bitcoin">
```

### New Format
```php
<img src="<?php echo cdn_url('BTC'); ?>" alt="Bitcoin">
```

### Benefits
1. **Centralized Configuration** - Change CDN URL in one place
2. **Format Flexibility** - Switch from webp to png or svg easily
3. **Environment Support** - Different CDNs for dev/staging/production
4. **Consistency** - Same API for PHP and JavaScript
5. **Type Safety** - Helper functions ensure correct URL generation

## Testing

### Test CDN Configuration Change

1. **Change format to PNG:**
   ```env
   CDN_XPAY_URL=https://cdn.xpay.co/coins/
   CDN_XPAY_FORMAT=png
   ```

2. **Clear OPcache:**
   ```bash
   docker exec wp-app php -r "opcache_reset();"
   ```

3. **Rebuild React (if changed):**
   ```bash
   npm run build
   ```

4. **Verify:** All coin images should now load from `.../coins/SYMBOL.png`

### Expected Results
- PHP pages: `cdn_url('BTC')` returns `https://cdn.xpay.co/coins/BTC.png`
- React components: `getCdnUrl('BTC')` returns `https://cdn.xpay.co/coins/BTC.png`

## Troubleshooting

### Images Not Loading
1. Check `.env` file has correct CDN_XPAY_URL
2. Verify URL ends with `/` or doesn't (code handles both)
3. Clear OPcache: `docker exec wp-app php -r "opcache_reset();"`
4. Check browser console for 404 errors

### React Components Not Updated
1. Rebuild with `npm run build`
2. Clear browser cache (Ctrl+Shift+R)
3. Verify `window.xpayConfig` exists in browser console
4. Check compiled JS files have `getCdnUrl()` function

### Wrong Image Format
1. Verify CDN_XPAY_FORMAT in `.env`
2. Clear OPcache
3. Rebuild React components
4. Check actual URL in browser DevTools

## Future Enhancements

Possible improvements:
- Multiple CDN support (fallback CDNs)
- Automatic format detection (WebP with PNG fallback)
- CDN URL versioning (cache busting)
- Per-coin custom images
- Lazy loading optimization

## Related Documentation
- [PAGESPEED-OPTIMIZATIONS.md](./PAGESPEED-OPTIMIZATIONS.md) - Image optimization
- [CPANEL-DEPLOYMENT.md](./CPANEL-DEPLOYMENT.md) - Production deployment
