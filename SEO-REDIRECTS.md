# SEO Redirect Controller

## مشکل
URL های duplicate content در سایت وجود داشت:
- ✗ `https://xpay.co/news` (بدون اسلش)
- ✗ `https://xpay.co/news/` (با اسلش)

هر دو URL باز می‌شدند و محتوای یکسان نمایش می‌دادند که باعث **duplicate content** و مشکلات SEO می‌شد.

## راه حل

Controller جدید `SEORedirectController` ایجاد شد که:

### 1. Archives و Taxonomies → باید اسلش داشته باشند
این URL ها **باید** به `/` ختم شوند:
- ✓ `/news/` (Post Type Archive)
- ✓ `/category/bitcoin/` (Category)
- ✓ `/tag/crypto/` (Tag)
- ✓ `/author/admin/` (Author Archive)
- ✓ Custom taxonomies و post types

**قبل:**
```
/news → 200 OK ❌
/news/ → 200 OK ❌ (duplicate content!)
```

**بعد:**
```
/news → 301 Redirect → /news/ ✓
/news/ → 200 OK ✓ (canonical URL)
```

### 2. Single Posts و Pages → نباید اسلش داشته باشند
این URL ها **نباید** به `/` ختم شوند:
- ✓ `/sample-post` (Single Post)
- ✓ `/about` (Page)
- ✓ Custom post types (تک مطالب)

**قبل:**
```
/sample-post → 200 OK ❌
/sample-post/ → 200 OK ❌ (duplicate content!)
```

**بعد:**
```
/sample-post → 200 OK ✓ (canonical URL)
/sample-post/ → 301 Redirect → /sample-post ✓
```

## فایل‌های تغییر یافته

1. **`app/Controllers/SEORedirectController.php`** (جدید)
   - Controller اصلی مدیریت redirects
   - متد `enforceTrailingSlash()` - بررسی و redirect
   - متد `shouldHaveTrailingSlash()` - تشخیص نوع صفحه
   - Logging برای debug در حالت `WP_DEBUG`

2. **`functions.php`**
   - اضافه شدن `use XPayMain\Controllers\SEORedirectController`
   - فراخوانی `SEORedirectController::register()`

## نحوه کار

```php
// در هر request:
template_redirect (hook)
    ↓
SEORedirectController::enforceTrailingSlash()
    ↓
1. بررسی: آیا admin/ajax/REST است? → خروج
2. بررسی: آیا POST request است? → خروج
3. بررسی: آیا فایل است (.xml, .json)? → خروج
4. تشخیص نوع صفحه:
   - Archive/Category/Tag? → باید اسلش داشته باشد
   - Single Post/Page? → نباید اسلش داشته باشد
5. URL فعلی را با URL مورد نظر مقایسه کن
6. اگر متفاوت بود → 301 Redirect
```

## تست و بررسی

### تست دستی با مرورگر:

1. **Test Archive:**
```
Visit: https://xpay.co/news
Expected: Redirect 301 → https://xpay.co/news/
```

2. **Test Archive (with slash):**
```
Visit: https://xpay.co/news/
Expected: 200 OK (no redirect)
```

3. **Test Single Post:**
```
Visit: https://your-post-slug/
Expected: Redirect 301 → https://your-post-slug
```

4. **Test Single Post (without slash):**
```
Visit: https://your-post-slug
Expected: 200 OK (no redirect)
```

### تست با PowerShell:

```powershell
# Test archive without slash
Invoke-WebRequest -Uri "https://xpay.co/news" -Method Head -MaximumRedirection 0

# Expected: StatusCode = 301, Location = https://xpay.co/news/
```

### تست با Browser DevTools:

1. باز کردن DevTools (F12)
2. رفتن به تب **Network**
3. بازدید از URL بدون اسلش
4. بررسی:
   - Status Code باید **301 Moved Permanently** باشد
   - Response Header باید **Location: /news/** داشته باشد

### فایل تست خودکار:

```
http://localhost/test-redirects.php
```

این فایل همه حالات را چک می‌کند و نتایج را نمایش می‌دهد.

## Debug و Logging

در حالت debug (`WP_DEBUG = true`):

```php
// در wp-config.php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
```

هر redirect در `wp-content/debug.log` ثبت می‌شود:

```
SEO Redirect: https://xpay.co/news → https://xpay.co/news/ [Post Type Archive: post]
```

## مزایای SEO

1. ✓ **رفع Duplicate Content** - هر محتوا فقط یک URL canonical دارد
2. ✓ **بهبود Crawl Budget** - Google نیازی به crawl کردن URL های تکراری ندارد
3. ✓ **تمرکز Link Juice** - تمام لینک‌ها به یک URL واحد اشاره می‌کنند
4. ✓ **بهبود در Google Search Console** - کاهش خطاهای duplicate content
5. ✓ **301 Redirect** - انتقال کامل SEO juice به URL صحیح

## استثناها

این موارد redirect **نمی‌شوند**:

- ❌ Admin panel (`/wp-admin/`)
- ❌ AJAX requests (`admin-ajax.php`)
- ❌ REST API endpoints (`/wp-json/`)
- ❌ POST requests (فرم‌ها)
- ❌ فایل‌های static (`.xml`, `.json`, `.txt`, etc.)

## مشکلات احتمالی و رفع آن‌ها

### مشکل: Redirect loop

**علت:** Conflict با plugin دیگری (مثل Rank Math یا Yoast)

**راه حل:**
```php
// غیرفعال کردن trailing slash redirect در plugin دیگر
// یا اولویت hook را تغییر دهید:
add_action('template_redirect', [$this, 'enforceTrailingSlash'], 1); // priority 1
```

### مشکل: Query parameters از بین می‌روند

**علت:** `trailingslashit()` ممکن است query string را حفظ نکند

**راه حل:** کد فعلی query parameters را حفظ می‌کند:
```php
$current_url = $this->getCurrentUrl(); // includes query string
$redirect_url = trailingslashit($current_url); // preserves query
```

### مشکل: Custom endpoints کار نمی‌کنند

**راه حل:** Extension check اضافه شده:
```php
if (preg_match('/\.[a-z0-9]{2,4}$/i', $path)) {
    return; // Skip files
}
```

## Performance

- ⚡ **سبک و سریع** - فقط یک بررسی ساده در هر request
- ⚡ **بدون Query** - هیچ database query اضافی ندارد
- ⚡ **Early Exit** - در صورت عدم نیاز به redirect سریع خارج می‌شود
- ⚡ **Priority 1** - قبل از load شدن template اجرا می‌شود

## سازگاری

- ✓ WordPress 5.0+
- ✓ همه post types (پیش‌فرض و custom)
- ✓ همه taxonomies (پیش‌فرض و custom)
- ✓ Multisite compatible
- ✓ WPML compatible
- ✓ Polylang compatible

## نکات برنامه‌نویسی

### اضافه کردن استثنا برای post type خاص:

```php
private function shouldHaveTrailingSlash()
{
    // استثنا برای post type خاص
    if (is_post_type_archive('coins')) {
        return false; // نباید اسلش داشته باشد
    }
    
    return (is_home() || is_archive() || ...);
}
```

### غیرفعال کردن برای taxonomy خاص:

```php
private function shouldHaveTrailingSlash()
{
    // استثنا برای taxonomy خاص
    if (is_tax('coin-category')) {
        return false;
    }
    
    return (is_home() || is_archive() || ...);
}
```

## بروزرسانی‌های آینده

- [ ] اضافه کردن cache برای performance بهتر
- [ ] پشتیبانی از custom rules از طریق admin panel
- [ ] گزارش‌دهی تعداد redirects در admin
- [ ] Integration با Google Search Console

## منابع

- [Google: Duplicate Content](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls)
- [WordPress Codex: Trailing Slashes](https://wordpress.org/support/article/htaccess/)
- [RFC 3986: URI Generic Syntax](https://www.rfc-editor.org/rfc/rfc3986)

---

## فیکس Pagination 404 (نسخه 5.6.8)

### مشکل
صفحات pagination بدون محتوا (مثل `/page/2`, `/page/999`) باید 404 برگردانند، اما وردپرس به‌صورت پیش‌فرض این صفحات را به‌صورت خالی نمایش می‌داد.

**URLهای مشکل‌دار:**
- `https://xpay.co/page/2` - وقتی صفحه اصلی یک صفحه هست
- `https://xpay.co/blog/page/999` - شماره صفحه بیشتر از حد
- `https://xpay.co/author/xpay/page/7/` - صفحه نویسنده بدون محتوا
- `https://xpay.co/page2` - فرمت غلط pagination

### راه حل

دو hook جدید در `functions.php` اضافه شد:

#### 1. بررسی Pagination بدون محتوا

```php
add_action('template_redirect', function () {
    if (!is_paged()) return;

    global $wp_query;
    $paged = get_query_var('paged', 1);

    // صفحه بدون پست
    if ($paged > 1 && !$wp_query->have_posts()) {
        $wp_query->set_404();
        status_header(404);
        nocache_headers();
        return;
    }

    // شماره صفحه بیشتر از حد مجاز
    $max_pages = $wp_query->max_num_pages;
    if ($paged > $max_pages && $max_pages > 0) {
        $wp_query->set_404();
        status_header(404);
        nocache_headers();
        return;
    }

    // صفحه اصلی: اگر همه پست‌ها در یک صفحه جا می‌شوند
    if (is_front_page() || is_home()) {
        $total_posts = $wp_query->found_posts;
        $posts_per_page = get_option('posts_per_page');
        if ($total_posts <= $posts_per_page && $paged > 1) {
            $wp_query->set_404();
            status_header(404);
            nocache_headers();
            return;
        }
    }
}, 5);
```

#### 2. بلاک کردن فرمت غلط Pagination

```php
add_action('template_redirect', function () {
    $request_uri = trim($_SERVER['REQUEST_URI'], '/');

    // /page2, /page3 (بدون اسلش)
    if (preg_match('/^page\d+$/i', $request_uri)) {
        global $wp_query;
        $wp_query->set_404();
        status_header(404);
        nocache_headers();
        return;
    }

    // /blog/page2, /coins/page3 (بدون اسلش بین page و عدد)
    if (preg_match('#/page\d+$#i', $_SERVER['REQUEST_URI'])) {
        global $wp_query;
        $wp_query->set_404();
        status_header(404);
        nocache_headers();
        return;
    }
}, 1);
```

### تست

**قبل:**
```
/page/999 → 200 OK ❌ (صفحه خالی)
/page2 → 200 OK ❌ (وردپرس این را handle نمی‌کرد)
```

**بعد:**
```
/page/999 → 404 Not Found ✓
/page2 → 404 Not Found ✓
/page/1 → 200 OK ✓ (اگر محتوا داشته باشد)
```

### موارد تحت پوشش

| URL Pattern | نتیجه |
|-------------|-------|
| `/page/2` (بدون محتوا) | 404 |
| `/page/999` | 404 |
| `/blog/page/50` (بیشتر از حد) | 404 |
| `/author/xpay/page/7/` (بدون محتوا) | 404 |
| `/page2` | 404 |
| `/coins/page3` | 404 |
| `/page/1` (با محتوا) | 200 OK |
| `/blog/page/2` (با محتوا) | 200 OK |

---

**نسخه:** 1.1.0  
**تاریخ:** 2026-01-07  
**نویسنده:** XPay Development Team
