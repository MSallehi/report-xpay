# Content Security Policy (CSP) Implementation

## Overview
This document explains the Content Security Policy (CSP) implementation in XPay theme to protect against XSS (Cross-Site Scripting) attacks.

## What is CSP?
Content Security Policy is a security layer that helps detect and mitigate certain types of attacks, including:
- Cross-Site Scripting (XSS)
- Data injection attacks
- Clickjacking

## Implementation Details

### Location
CSP is implemented in `functions.php` using the `send_headers` action hook.

```php
add_action('send_headers', function () {
    // CSP headers are added here
}, 1);
```

### Why HTTP Headers Instead of Meta Tags?
We use HTTP headers instead of `<meta>` tags because:
- ✅ **More Secure**: Headers are processed before any page content
- ✅ **Better Performance**: No need to parse HTML
- ✅ **Full Control**: Can use all CSP directives
- ✅ **PageSpeed Compliant**: Google recommends headers over meta tags

## CSP Directives Explained

### 1. script-src
Controls which scripts can execute:
```
script-src 'self' 'unsafe-inline' 'unsafe-eval' 
  https://www.googletagmanager.com 
  https://van.najva.com 
  https://cdn.goftino.com
  https://s1.mediaad.org
```

**Why `'unsafe-inline'` and `'unsafe-eval'`?**
- Google Tag Manager requires `'unsafe-eval'`
- Some analytics tools use inline scripts
- Future improvement: Use nonces for inline scripts

**Added Domains:**
- `s1.mediaad.org` - Retargeting and analytics script

### 2. object-src
Blocks plugins like Flash, Java applets:
```
object-src 'none'
```
This is **critical** for security - blocks injection of malicious plugins.

### 3. style-src
Controls stylesheet sources:
```
style-src 'self' 'unsafe-inline' https://fonts.googleapis.com
```
Allows Google Fonts and inline styles.

### 4. img-src
Controls image sources:
```
img-src 'self' data: https: http:
```
Allows all images (data URIs, HTTPS, HTTP for compatibility).

### 5. font-src
Controls font sources:
```
font-src 'self' data: https://fonts.gstatic.com
```
Allows Google Fonts and data URIs.

### 6. connect-src
Controls AJAX, WebSocket, EventSource connections:
```
connect-src 'self' 
  https://www.google-analytics.com 
  https://van.najva.com 
  https://cdn.goftino.com
  https://api.xpay.co
```

**Added Domains:**
- `api.xpay.co` - XPay API for chart data and fiat conversions

### 7. frame-src
Controls iframe sources:
```
frame-src 'self' https://www.googletagmanager.com
```
Only allows Google Tag Manager iframes.

### 8. Other Security Directives

```
base-uri 'self'               # Prevents <base> tag hijacking
form-action 'self'            # Forms can only submit to same origin
frame-ancestors 'self'        # Prevents clickjacking
upgrade-insecure-requests     # Upgrades HTTP to HTTPS
```

## Testing CSP

### Browser Console
Open browser DevTools (F12) → Console tab. If CSP blocks something, you'll see:
```
Refused to load the script 'https://evil.com/script.js' because it violates 
the following Content Security Policy directive: "script-src 'self'..."
```

### Online Tools
- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [Mozilla Observatory](https://observatory.mozilla.org/)

### PageSpeed Insights
Test at: https://pagespeed.web.dev/
Should show ✅ for "Ensure CSP is effective against XSS attacks"

## Common Issues & Solutions

### Issue: "Script blocked by CSP"
**Solution**: Add the domain to `script-src`:
```php
"script-src 'self' 'unsafe-inline' https://new-domain.com",
```

### Issue: "Stylesheet blocked by CSP"
**Solution**: Add the domain to `style-src`:
```php
"style-src 'self' 'unsafe-inline' https://new-domain.com",
```

### Issue: "Font blocked by CSP"
**Solution**: Add the domain to `font-src`:
```php
"font-src 'self' data: https://new-domain.com",
```

## Future Improvements

### 1. Use Nonces for Inline Scripts
Instead of `'unsafe-inline'`, use nonces:
```php
$nonce = base64_encode(random_bytes(16));
header("Content-Security-Policy: script-src 'self' 'nonce-$nonce'");
```

Then in HTML:
```html
<script nonce="<?php echo $nonce; ?>">
  // Your inline script
</script>
```

### 2. Report-Only Mode
Test CSP without blocking:
```php
header("Content-Security-Policy-Report-Only: ...");
```

### 3. CSP Reporting
Log violations to server:
```php
"report-uri /csp-violation-report"
```

## File Structure
```
functions.php                     # CSP implementation
docs/CSP_SECURITY.md             # This documentation
docs/changelog/CHANGELOG-FA.md   # Persian changelog
docs/changelog/CHANGELOG-EN.md   # English changelog
```

## References
- [MDN CSP Guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [Google CSP Guide](https://developers.google.com/web/fundamentals/security/csp)
- [Content-Security-Policy.com](https://content-security-policy.com/)

## Related Files
- `functions.php` - CSP implementation
- `header.php` - Removed old meta tag CSP
- `docs/NAJVA_OPTIMIZATION.md` - Najva integration docs
- `docs/SOURCE_MAPS_README.md` - Source maps docs

---

**Last Updated**: December 2025  
**Version**: 2.1.0
