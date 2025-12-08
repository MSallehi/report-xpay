# Source Maps برای فایل‌های Webpack

این پوشه شامل راهنمای استفاده از source maps برای فایل‌های webpack است.

## چرا Source Maps؟

Source maps به PageSpeed Insights و browser DevTools کمک می‌کند که:
- کد minified شده را به کد اصلی map کنند
- debugging در production آسان‌تر شود
- Performance insights دقیق‌تری ارائه دهند

## ساختار فایل‌ها

```
assets/js/
├── app-vendor.js              # Bundle اصلی
├── app-vendor.js.map          # Source map اصلی
├── app-vendor.v5.5.32.js      # Bundle نسخه‌دار (برای production)
└── app-vendor.v5.5.32.js.map  # Source map نسخه‌دار
```

## نحوه استفاده

### 1. Build کردن webpack با source maps

```bash
npm run build
```

این دستور webpack را با `devtool: "source-map"` می‌سازد و فایل‌های زیر را ایجاد می‌کند:
- `app-vendor.js` + `app-vendor.js.map`
- `app-calculator.js` + `app-calculator.js.map`
- `app-chart.js` + `app-chart.js.map`
- `app-coins.js` + `app-coins.js.map`

### 2. ساخت فایل‌های نسخه‌دار برای production

بعد از build، فایل‌های versioned را بسازید:

```bash
# خودکار (نسخه از .env خوانده می‌شود)
npm run build:production

# یا دستی با PowerShell
.\create-versioned-sourcemaps.ps1 -Version "5.5.32"

# یا با Node.js
node create-versioned-sourcemaps.js
```

### 3. Deploy به production

فایل‌های زیر را به production منتقل کنید:
- همه فایل‌های `.js`
- همه فایل‌های `.js.map`

**مهم**: مطمئن شوید nginx یا Apache فایل‌های `.map` را serve می‌کنند!

## تنظیمات nginx

برای serve کردن source maps:

```nginx
location ~ \.map$ {
    add_header 'Access-Control-Allow-Origin' '*';
    add_header 'Content-Type' 'application/json';
    expires 1y;
}
```

## Troubleshooting

### خطای "Timed out fetching resource"

این خطا معمولاً به این دلایل رخ می‌دهد:

1. **فایل‌های versioned وجود ندارند**:
   - اگر `ASSET_VERSION` در production/staging تغییر کرده (مثلاً 5.5.34)
   - ولی فایل‌های versioned در سرور نیستند
   - **راه حل**: فایل‌های versioned را بسازید:
     ```bash
     # برای نسخه مشخص
     .\create-versioned-sourcemaps.ps1 -Version "5.5.34"
     
     # سپس commit و push کنید
     git add assets/js/*.v5.5.34.*
     git commit -m "Add source maps for version 5.5.34"
     git push
     ```

2. **Source map reference اشتباه**: 
   - فایل `app-vendor.v5.5.32.js` باید به `app-vendor.v5.5.32.js.map` اشاره کنه
   - نه به `app-vendor.js.map`

3. **فایل .map وجود ندارد در سرور**:
   - مطمئن شوید فایل‌های `.map` در production/staging هستند
   - بررسی کنید که `.gitignore` این فایل‌ها را ignore نمی‌کند
   - فایل‌های `.map` باید با فایل‌های `.js` در یک پوشه باشند

4. **MIME type اشتباه**:
   - فایل‌های `.map` باید با `Content-Type: application/json` serve شوند
   - nginx configuration را بررسی کنید

5. **Permission مشکل دارد**:
   - مطمئن شوید فایل‌های `.map` قابل خواندن هستند (chmod 644)

### بررسی وضعیت فایل‌ها

```bash
# بررسی که فایل‌های versioned برای نسخه فعلی وجود دارند
ls assets/js/*.v5.5.34.*

# بررسی نسخه در .env
grep ASSET_VERSION .env

# ساخت فایل‌های versioned اگر وجود ندارند
.\create-versioned-sourcemaps.ps1 -Version "5.5.34"
```

2. **فایل .map وجود ندارد**:
   - مطمئن شوید فایل‌های `.map` در production هستند

3. **MIME type اشتباه**:
   - فایل‌های `.map` باید با `Content-Type: application/json` serve شوند

### بررسی source map reference

```bash
# بررسی آخرین خط فایل JS
tail -n 1 app-vendor.v5.5.32.js
# باید چیزی شبیه این باشد:
# //# sourceMappingURL=app-vendor.v5.5.32.js.map
```

## فایل‌های مهم

- `webpack.config.js` - تنظیم `devtool: "source-map"`
- `create-versioned-sourcemaps.js` - اسکریپت Node.js برای ساخت نسخه‌دار
- `create-versioned-sourcemaps.ps1` - اسکریپت PowerShell برای Windows
- `package.json` - npm scripts

## یادداشت‌های مهم

1. فایل‌های `.map` حجم زیادی دارند (app-vendor.js.map ~3MB)
2. Source maps فقط در صورت نیاز load می‌شوند (DevTools باز باشد)
3. فایل‌های versioned باید هر بار با ASSET_VERSION جدید ساخته شوند
4. Git این فایل‌ها را ignore می‌کند (در `.gitignore`)
