# مستندات تنظیمات Rank Math در قالب XPay

## مقدمه

این سند توضیحاتی درباره پیکربندی‌ها و سفارشی‌سازی‌های انجام شده برای افزونه **Rank Math SEO** در قالب XPay ارائه می‌دهد. تمامی تنظیمات مربوط به Rank Math در یک Controller مجزا به نام `RankMathController` مدیریت می‌شود.

---

## ساختار و معماری

### موقعیت فایل
```
wp-content/themes/xpay_main_theme/app/Controllers/RankMathController.php
```

### ثبت Controller
در فایل `functions.php`:
```php
use XPayMain\Controllers\RankMathController;

// Register the Rank Math controller (manages all Rank Math SEO customizations)
RankMathController::register();
```

---

## قابلیت‌های پیاده‌سازی شده

### 1. تغییر Schema Breadcrumb
**هدف:** تغییر خودکار نام "کوین‌ها" به "خرید ارز دیجیتال" در Schema Breadcrumb

#### توضیحات
- این تغییر به صورت خودکار روی تمام schema های تولید شده توسط Rank Math اعمال می‌شود
- تیم SEO می‌تواند از پنل مدیریت Rank Math سایر تنظیمات schema را تغییر دهد
- این تغییر فقط روی breadcrumb اعمال می‌شود و سایر قسمت‌های schema دست نخورده باقی می‌ماند

#### متد مسئول
```php
public function modifySchema($data, $jsonld)
```

#### فیلتر استفاده شده
```php
add_filter('rank_math/json_ld', [$this, 'modifySchema'], 99, 2);
```

#### مثال Schema قبل از تغییر
```json
{
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": "2",
      "item": {
        "@id": "https://xpay.co/coin/",
        "name": "کوین‌ها"
      }
    }
  ]
}
```

#### مثال Schema بعد از تغییر
```json
{
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": "2",
      "item": {
        "@id": "https://xpay.co/coin/",
        "name": "خرید ارز دیجیتال"
      }
    }
  ]
}
```

---

### 2. تنظیم Canonical URL

**هدف:** اصلاح canonical URL برای صفحات صفحه‌بندی شده نویسنده

#### توضیحات
- در صفحات آرشیو نویسنده که صفحه‌بندی دارند، canonical URL به صفحه اول اشاره می‌کند
- این کار از ایجاد مشکلات duplicate content جلوگیری می‌کند

#### متد مسئول
```php
public function customizeCanonicalUrl($canonical)
```

#### فیلتر استفاده شده
```php
add_filter('rank_math/canonical_url', [$this, 'customizeCanonicalUrl']);
```

#### مثال
- صفحه: `https://xpay.co/author/username/page/2/`
- Canonical: `https://xpay.co/author/username/`

---

### 3. تنظیم Robots Meta Tag

**هدف:** مدیریت نمایه‌سازی صفحات خاص در موتورهای جستجو

#### توضیحات
صفحات زیر از نمایه‌سازی موتورهای جستجو خارج می‌شوند:
- صفحات آرشیو نویسنده (Author Archives)
- صفحات آرشیو اخبار (News Archive)
- صفحات آرشیو تحلیل‌ها (Analysis Archive)

#### متد مسئول
```php
public function customizeRobotsMeta($robots)
```

#### فیلتر استفاده شده
```php
add_filter('rank_math/frontend/robots', [$this, 'customizeRobotsMeta']);
```

#### خروجی
```html
<meta name="robots" content="noindex, nofollow">
```

---

### 4. تبدیل H3 به Span در FAQ Block

**هدف:** جلوگیری از ایجاد تگ‌های H3 متعدد در بلوک‌های FAQ

#### توضیحات
- بلوک‌های FAQ در Rank Math به صورت پیش‌فرض از تگ H3 استفاده می‌کنند
- برای بهبود ساختار سئو، این تگ‌ها به `<span>` تبدیل می‌شوند
- استایل‌های CSS حفظ می‌شود

#### متدهای مسئول
```php
public function convertFaqHeadings($block_content, $block)
public function convertFaqHeadingsInContent($content)
```

#### فیلترهای استفاده شده
```php
add_filter('render_block', [$this, 'convertFaqHeadings'], 10, 2);
add_filter('the_content', [$this, 'convertFaqHeadingsInContent'], 20);
```

#### مثال تبدیل
**قبل:**
```html
<h3 class="rank-math-question">سوال چیست؟</h3>
```

**بعد:**
```html
<span class="rank-math-question">سوال چیست؟</span>
```

---

## نحوه استفاده

### فعال‌سازی تنظیمات
تنظیمات به صورت خودکار با فعال بودن قالب اعمال می‌شوند. نیازی به تنظیمات دستی نیست.

### غیرفعال کردن موقت یک قابلیت
اگر نیاز به غیرفعال کردن موقت یک قابلیت دارید، می‌توانید خط مربوط به آن فیلتر را در متد `init()` کامنت کنید:

```php
private function init()
{
    // Schema modifications
    add_filter('rank_math/json_ld', [$this, 'modifySchema'], 99, 2);

    // Canonical URL customizations
    // add_filter('rank_math/canonical_url', [$this, 'customizeCanonicalUrl']);

    // ... rest of filters
}
```

---

## تغییرات آینده

### اضافه کردن فیلتر جدید
برای اضافه کردن یک فیلتر یا hook جدید Rank Math:

1. متد عمومی جدید در کلاس ایجاد کنید:
```php
public function myNewCustomization($param)
{
    // Your code here
    return $param;
}
```

2. فیلتر را در متد `init()` ثبت کنید:
```php
private function init()
{
    // ... existing filters
    add_filter('rank_math/filter_name', [$this, 'myNewCustomization']);
}
```

### تغییر اولویت فیلتر
اولویت فیلترها را می‌توانید از طریق پارامتر سوم تغییر دهید:
```php
add_filter('rank_math/json_ld', [$this, 'modifySchema'], 99, 2);
//                                                          ^^ اولویت (عدد بالاتر = اجرای دیرتر)
```

---

## نکات مهم برای تیم SEO

### ✅ آنچه تیم SEO می‌تواند تغییر دهد:
- تمامی تنظیمات Schema از پنل Rank Math
- Meta Title و Meta Description
- Focus Keyword و تنظیمات آنالیز محتوا
- Snippet Preview
- تنظیمات Sitemap
- تنظیمات Redirects
- تنظیمات Social Media

### ⚠️ آنچه به صورت خودکار اعمال می‌شود:
- نام "کوین‌ها" همیشه به "خرید ارز دیجیتال" در breadcrumb تبدیل می‌شود
- صفحات Author/News/Analysis همیشه noindex هستند
- تگ‌های H3 در FAQ همیشه به span تبدیل می‌شوند

---

## پشتیبانی و سوالات

برای سوالات فنی و مشکلات مربوط به RankMathController با تیم توسعه تماس بگیرید.

### اطلاعات فنی
- **نسخه:** 1.0.0
- **تاریخ ایجاد:** نوامبر 2025
- **سازگاری:** Rank Math 1.0.100+
- **معماری:** MVC Pattern

---

## تاریخچه تغییرات

### نسخه 1.0.0 (نوامبر 2025)
- پیاده‌سازی اولیه RankMathController
- اضافه شدن تغییر خودکار نام breadcrumb
- اضافه شدن تنظیمات canonical URL
- اضافه شدن تنظیمات robots meta
- اضافه شدن تبدیل FAQ headings

---

## منابع مفید

- [Rank Math Filters Documentation](https://rankmath.com/kb/filters-hooks-api-developer/)
- [Rank Math JSON-LD Schema](https://rankmath.com/kb/rich-snippets/)
- [Schema.org Breadcrumb](https://schema.org/BreadcrumbList)
