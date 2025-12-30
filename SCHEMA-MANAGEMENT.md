# 🔷 مدیریت Schema در قالب XPay
# Schema Management Guide

> **فایل اصلی:** `app/Controllers/RankMathController.php`  
> **آخرین بروزرسانی:** دسامبر 2024  
> **نسخه:** 1.0.0

---

## 📋 فهرست مطالب

1. [معرفی](#معرفی)
2. [ساختار فایل](#ساختار-فایل)
3. [Schema های فعال](#schema-های-فعال)
4. [نحوه اضافه کردن Schema جدید](#نحوه-اضافه-کردن-schema-جدید)
5. [نحوه حذف Schema](#نحوه-حذف-schema)
6. [تغییر Schema موجود](#تغییر-schema-موجود)
7. [مثال‌های کاربردی](#مثالهای-کاربردی)
8. [تست و اعتبارسنجی](#تست-و-اعتبارسنجی)
9. [نکات مهم](#نکات-مهم)

---

## معرفی

فایل `RankMathController.php` مسئول مدیریت تمام Schema های JSON-LD در قالب XPay است. این کنترلر از فیلتر `rank_math/json_ld` برای تغییر schema های Rank Math و از `wp_head` برای اضافه کردن schema های سفارشی استفاده می‌کند.

### چرا Schema مهم است؟
- **SEO بهتر**: گوگل از Schema برای نمایش Rich Snippets استفاده می‌کند
- **CTR بالاتر**: نتایج جستجو با Rich Snippets کلیک بیشتری دارند
- **درک بهتر محتوا**: موتورهای جستجو محتوای سایت را بهتر می‌فهمند

---

## ساختار فایل

```
app/Controllers/RankMathController.php
│
├── register()              # ثبت هوک‌ها
├── init()                  # Initialize فیلترها و اکشن‌ها
│
├── modifySchema()          # فیلتر اصلی Rank Math JSON-LD
├── modifyBreadcrumbList()  # تغییر Breadcrumb
├── createCoinBreadcrumb()  # ساخت Breadcrumb سفارشی برای صفحات coin
│
├── outputCoinFaqSchema()   # خروجی FAQ Schema در wp_head
├── generateCoinFaqSchema() # تولید FAQ Schema از ACF
│
├── outputCoinVideoSchema() # خروجی VideoObject Schema در wp_head
├── generateCoinVideoSchema() # تولید VideoObject Schema
│
└── ... سایر متدها
```

---

## Schema های فعال

### صفحات Coin (ارزها)

| Schema | وضعیت | توضیحات |
|--------|-------|---------|
| BreadcrumbList | ✅ فعال | مسیر: خانه > خرید ارز دیجیتال > خرید {نام ارز} |
| FAQPage | ✅ فعال | از فیلد ACF `coin_faq` خوانده می‌شود |
| VideoObject | ✅ فعال | ویدیو آپارات از فیلد ACF `coin_video` |
| CreativeWorkSeason | ✅ فعال | (اگر Rank Math تولید کند) |

### Schema های حذف شده از صفحات Coin
- WebPage
- WebSite  
- Organization
- Article

---

## نحوه اضافه کردن Schema جدید

### روش 1: اضافه کردن به فیلتر Rank Math

اگر می‌خواهید schema به `@graph` اصلی Rank Math اضافه شود:

```php
// در متد modifySchema() پس از فیلتر کردن graph:
if ($is_coin_page) {
    $my_schema = $this->generateMyNewSchema();
    if ($my_schema) {
        $filtered_graph[] = $my_schema;
    }
}
```

### روش 2: خروجی مستقیم در wp_head (توصیه شده)

این روش مطمئن‌تر است و مستقل از Rank Math کار می‌کند:

#### مرحله 1: ثبت اکشن در init()

```php
private function init()
{
    // ... کدهای موجود ...
    
    // اضافه کردن schema جدید
    add_action('wp_head', [$this, 'outputMyNewSchema'], 7);
}
```

#### مرحله 2: ایجاد متد خروجی

```php
/**
 * Output MyNew schema in head
 */
public function outputMyNewSchema()
{
    // فقط در صفحات مورد نظر
    if (!is_singular('my_post_type')) {
        return;
    }

    $schema = $this->generateMyNewSchema();

    if (!$schema) {
        return;
    }

    echo '<script type="application/ld+json">' . wp_json_encode($schema, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES) . '</script>' . "\n";
}
```

#### مرحله 3: ایجاد متد تولید Schema

```php
/**
 * Generate MyNew schema
 * 
 * @return array|null Schema or null
 */
private function generateMyNewSchema()
{
    $post_id = get_queried_object_id();
    if (!$post_id) {
        $post_id = get_the_ID();
    }

    if (!$post_id) {
        return null;
    }

    // ساخت schema
    return [
        '@context' => 'https://schema.org',
        '@type' => 'MySchemaType',
        'name' => get_the_title($post_id),
        // ... سایر فیلدها
    ];
}
```

---

## نحوه حذف Schema

### حذف از لیست مجاز صفحات Coin

در متد `modifySchema()`:

```php
// Schema های مجاز برای صفحات coin
$allowed_coin_schemas = [
    'BreadcrumbList',
    'FAQPage',
    'CreativeWorkSeason',
    'Question',
    'Answer',
    // 'VideoObject',  // کامنت کنید برای حذف
];
```

### غیرفعال کردن کامل یک Schema

در متد `init()` اکشن مربوطه را کامنت کنید:

```php
private function init()
{
    // ...
    
    // غیرفعال کردن FAQ schema
    // add_action('wp_head', [$this, 'outputCoinFaqSchema'], 5);
    
    // غیرفعال کردن Video schema
    // add_action('wp_head', [$this, 'outputCoinVideoSchema'], 6);
}
```

---

## تغییر Schema موجود

### تغییر Breadcrumb

برای تغییر ساختار Breadcrumb صفحات coin، متد `createCoinBreadcrumb()` را ویرایش کنید:

```php
private function createCoinBreadcrumb()
{
    // ...
    
    return [
        '@type' => 'BreadcrumbList',
        'itemListElement' => [
            [
                '@type' => 'ListItem',
                'position' => 1,
                'name' => 'نام جدید برای خانه',  // تغییر نام
                'item' => [
                    '@type' => 'Thing',
                    '@id' => $home_url,
                ],
            ],
            // ... سایر آیتم‌ها
        ],
    ];
}
```

### تغییر VideoObject

متد `generateCoinVideoSchema()` را ویرایش کنید:

```php
private function generateCoinVideoSchema()
{
    // ...
    
    return [
        '@context' => 'https://schema.org',
        '@type' => 'VideoObject',
        'name' => sprintf('عنوان جدید برای %s', $coin_name),  // تغییر فرمت نام
        'duration' => 'PT5M0S',  // تغییر مدت زمان پیش‌فرض
        // ...
    ];
}
```

---

## مثال‌های کاربردی

### مثال 1: اضافه کردن Product Schema برای صفحات محصول

```php
// در init()
add_action('wp_head', [$this, 'outputProductSchema'], 8);

// متد جدید
public function outputProductSchema()
{
    if (!is_singular('product')) {
        return;
    }

    $post_id = get_queried_object_id();
    $price = get_field('product_price', $post_id);
    
    $schema = [
        '@context' => 'https://schema.org',
        '@type' => 'Product',
        'name' => get_the_title($post_id),
        'description' => get_the_excerpt($post_id),
        'offers' => [
            '@type' => 'Offer',
            'price' => $price,
            'priceCurrency' => 'IRR',
            'availability' => 'https://schema.org/InStock',
        ],
    ];

    echo '<script type="application/ld+json">' . wp_json_encode($schema, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES) . '</script>' . "\n";
}
```

### مثال 2: اضافه کردن Article Schema برای اخبار

```php
// در init()
add_action('wp_head', [$this, 'outputNewsArticleSchema'], 9);

// متد جدید
public function outputNewsArticleSchema()
{
    if (!is_singular('news')) {
        return;
    }

    $post_id = get_queried_object_id();
    
    $schema = [
        '@context' => 'https://schema.org',
        '@type' => 'NewsArticle',
        'headline' => get_the_title($post_id),
        'datePublished' => get_the_date('c', $post_id),
        'dateModified' => get_the_modified_date('c', $post_id),
        'author' => [
            '@type' => 'Person',
            'name' => get_the_author_meta('display_name'),
        ],
        'publisher' => [
            '@type' => 'Organization',
            'name' => 'ایکس پی',
            'logo' => [
                '@type' => 'ImageObject',
                'url' => TEMPLATE_DIR . '/assets/img/logo.png',
            ],
        ],
    ];

    echo '<script type="application/ld+json">' . wp_json_encode($schema, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES) . '</script>' . "\n";
}
```

### مثال 3: اضافه کردن HowTo Schema

```php
public function outputHowToSchema()
{
    if (!is_singular('tutorial')) {
        return;
    }

    $post_id = get_queried_object_id();
    $steps = get_field('tutorial_steps', $post_id); // ACF Repeater
    
    $step_items = [];
    $position = 1;
    
    foreach ($steps as $step) {
        $step_items[] = [
            '@type' => 'HowToStep',
            'position' => $position,
            'name' => $step['step_title'],
            'text' => $step['step_content'],
        ];
        $position++;
    }

    $schema = [
        '@context' => 'https://schema.org',
        '@type' => 'HowTo',
        'name' => get_the_title($post_id),
        'step' => $step_items,
    ];

    echo '<script type="application/ld+json">' . wp_json_encode($schema, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES) . '</script>' . "\n";
}
```

---

## تست و اعتبارسنجی

### ابزارهای تست

1. **Google Rich Results Test**
   - https://search.google.com/test/rich-results

2. **Schema.org Validator**
   - https://validator.schema.org/

3. **View Page Source**
   - در مرورگر: `Ctrl+U` یا `View Page Source`
   - جستجو کنید: `application/ld+json`

### مراحل تست

1. تغییرات را ذخیره کنید
2. کش سایت را پاک کنید (اگر دارید)
3. صفحه مورد نظر را باز کنید
4. View Source کنید و `ld+json` را جستجو کنید
5. Schema را در validator.schema.org تست کنید

---

## نکات مهم

### ⚠️ نکات ضروری

1. **همیشه `JSON_UNESCAPED_UNICODE` استفاده کنید** - برای نمایش صحیح کاراکترهای فارسی

2. **از `get_queried_object_id()` استفاده کنید** - در فیلترها `get_the_ID()` ممکن است کار نکند

3. **Schema های تکراری نسازید** - قبل از اضافه کردن، چک کنید Rank Math قبلاً نساخته باشد

4. **تست در Production** - گوگل ممکن است در localhost تست نکند

### 📌 Best Practices

```php
// ✅ صحیح - چک کردن نوع صفحه
if (!is_singular('coin')) {
    return;
}

// ✅ صحیح - گرفتن post_id به روش مطمئن
$post_id = get_queried_object_id();
if (!$post_id) {
    $post_id = get_the_ID();
}

// ✅ صحیح - چک کردن وجود ACF
if (!function_exists('get_field')) {
    return null;
}

// ✅ صحیح - encode با فلگ‌های صحیح
echo '<script type="application/ld+json">' . wp_json_encode($schema, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES) . '</script>';
```

### 🚫 اشتباهات رایج

```php
// ❌ غلط - استفاده از json_encode بدون فلگ
echo json_encode($schema);

// ❌ غلط - عدم چک نوع صفحه
public function outputSchema() {
    // بدون چک is_singular
    $schema = $this->generateSchema();
}

// ❌ غلط - استفاده از have_rows در فیلتر
while (have_rows('field')) { // ممکن است کار نکند
    // ...
}

// ✅ صحیح - از get_field استفاده کنید
$data = get_field('field', $post_id);
foreach ($data as $item) {
    // ...
}
```

---

## لیست Schema های Schema.org

برای اطلاع از انواع Schema:
- https://schema.org/docs/full.html

Schema های پرکاربرد:
- `Article` / `NewsArticle` / `BlogPosting`
- `Product` / `Offer`
- `FAQPage` / `Question` / `Answer`
- `HowTo` / `HowToStep`
- `VideoObject`
- `BreadcrumbList` / `ListItem`
- `Organization` / `LocalBusiness`
- `Person`
- `Event`
- `Review` / `AggregateRating`

---

## پشتیبانی

در صورت بروز مشکل:
1. لاگ‌های PHP را چک کنید
2. Schema را در validator تست کنید
3. View Source صفحه را بررسی کنید

---

**نویسنده:** تیم توسعه XPay  
**تاریخ:** دسامبر 2024
