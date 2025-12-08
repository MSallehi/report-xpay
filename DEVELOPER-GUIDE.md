# راهنمای توسعه‌دهندگان - تم XPay

این راهنما برای توسعه‌دهندگانی است که می‌خواهند template های جدید به تم اضافه کنند یا تغییرات عمده‌ای در ساختار تم ایجاد کنند.

## فهرست مطالب

1. [افزودن Page Template جدید](#افزودن-page-template-جدید)
2. [افزودن Archive Template جدید](#افزودن-archive-template-جدید)
3. [افزودن Single Template جدید](#افزودن-single-template-جدید)
4. [ساخت Controller جدید](#ساخت-controller-جدید)
5. [کار با Partial Views](#کار-با-partial-views)
6. [دیباگ و رفع مشکل](#دیباگ-و-رفع-مشکل)

---

## افزودن Page Template جدید

برای اضافه کردن یک صفحه سفارشی جدید (مثلاً صفحه "پرسش و پاسخ"):

### مرحله 1: ساخت فایل Root Template

در root تم، فایل `faq.php` بسازید:

```php
<?php
/*
Template Name: faq
*/

/**
 * This file is handled by MVC routing system
 * Content is rendered from views/
 * 
 * @see routes.php
 */
```

**نکته مهم:** نام در `Template Name` باید با نامی که در route ثبت می‌کنید یکسان باشد.

### مرحله 2: افزودن متد به Controller

فایل `app/Controllers/PageController.php` را باز کرده و متد جدید اضافه کنید:

```php
/**
 * FAQ page
 */
public function faq()
{
    $data = [
        'page_title' => get_the_title(),
        'faqs' => get_field('faq_items'), // مثال: دریافت از ACF
        // اضافه کردن داده‌های مورد نیاز
    ];

    $this->render('pages.faq', $data);
}
```

### مرحله 3: ساخت View

فایل view را در `views/pages/faq.php` بسازید:

```php
<?php
/**
 * FAQ Page Template
 * 
 * @var string $page_title
 * @var array $faqs
 */

defined('ABSPATH') || exit;
?>

<div class="faq-page">
    <h1><?php echo esc_html($page_title); ?></h1>
    
    <?php if (!empty($faqs)): ?>
        <div class="faq-list">
            <?php foreach ($faqs as $faq): ?>
                <div class="faq-item">
                    <h3><?php echo esc_html($faq['question']); ?></h3>
                    <p><?php echo wp_kses_post($faq['answer']); ?></p>
                </div>
            <?php endforeach; ?>
        </div>
    <?php endif; ?>
</div>
```

### مرحله 4: ثبت Route

فایل `routes.php` را باز کرده و route جدید اضافه کنید:

```php
Template::register('faq', PageController::class, 'faq', 'pages.faq');
```

**فرمت:**
```php
Template::register(
    'template-name',      // نام template (باید با Template Name در فایل root یکسان باشد)
    Controller::class,    // کلاس کنترلر
    'methodName',         // نام متد در کنترلر
    'view.path'          // مسیر view با dot notation
);
```

### تست

1. در وردپرس، یک صفحه جدید بسازید
2. از قسمت "Page Attributes" → "Template" گزینه "faq" را انتخاب کنید
3. صفحه را ذخیره و مشاهده کنید

---

## افزودن Archive Template جدید

برای اضافه کردن آرشیو برای post type جدید (مثلاً "video"):

### مرحله 1: ساخت Root Template

فایل `archive-video.php` در root تم بسازید:

```php
<?php
/*
Template Name: archive-video
*/

/**
 * This file is handled by MVC routing system
 * Content is rendered from views/
 * 
 * @see routes.php
 */
```

### مرحله 2: افزودن متد به ArchiveController

```php
/**
 * Video archive
 */
public function video()
{
    $data = [
        'posts' => new \WP_Query([
            'post_type' => 'video',
            'posts_per_page' => get_option('posts_per_page'),
            'paged' => get_query_var('paged') ? get_query_var('paged') : 1,
        ]),
        'archive_title' => get_field('video_archive_title', 'option'),
    ];

    $this->render('archives.video', $data);
}
```

### مرحله 3: ساخت View

فایل `views/archives/video.php` بسازید:

```php
<?php
/**
 * Video Archive Template
 * 
 * @var WP_Query $posts
 * @var string $archive_title
 */

defined('ABSPATH') || exit;
?>

<div class="video-archive">
    <h1><?php echo esc_html($archive_title); ?></h1>
    
    <?php if ($posts->have_posts()): ?>
        <div class="video-list">
            <?php while ($posts->have_posts()): $posts->the_post(); ?>
                <article class="video-item">
                    <h2><?php the_title(); ?></h2>
                    <?php the_excerpt(); ?>
                </article>
            <?php endwhile; ?>
        </div>
        
        <?php
        // Pagination
        the_posts_pagination();
        wp_reset_postdata();
        ?>
    <?php else: ?>
        <p>ویدیویی یافت نشد.</p>
    <?php endif; ?>
</div>
```

### مرحله 4: ثبت Route

```php
Template::register('archive-video', ArchiveController::class, 'video', 'archives.video');
```

---

## افزودن Single Template جدید

برای اضافه کردن template برای نمایش تک post از نوع سفارشی:

### مرحله 1: ساخت Root Template

فایل `single-video.php` بسازید:

```php
<?php
/*
Template Name: single-video
*/

/**
 * This file is handled by MVC routing system
 * Content is rendered from views/
 * 
 * @see routes.php
 */
```

### مرحله 2: افزودن متد به SingleController

```php
/**
 * Single video page
 */
public function video()
{
    global $post;

    $data = [
        'post' => $post,
        'video_url' => get_field('video_url'),
        'duration' => get_field('video_duration'),
        'related_videos' => $this->getRelatedVideos($post->ID),
    ];

    $this->render('singles.video', $data);
}

/**
 * Get related videos
 */
private function getRelatedVideos($post_id)
{
    return new \WP_Query([
        'post_type' => 'video',
        'posts_per_page' => 4,
        'post__not_in' => [$post_id],
        'orderby' => 'rand',
    ]);
}
```

### مرحله 3: ساخت View

```php
<?php
/**
 * Single Video Template
 * 
 * @var WP_Post $post
 * @var string $video_url
 * @var string $duration
 * @var WP_Query $related_videos
 */

defined('ABSPATH') || exit;
?>

<article class="single-video">
    <h1><?php echo esc_html($post->post_title); ?></h1>
    
    <div class="video-player">
        <iframe src="<?php echo esc_url($video_url); ?>"></iframe>
    </div>
    
    <div class="video-content">
        <?php echo wp_kses_post($post->post_content); ?>
    </div>
    
    <?php if ($related_videos->have_posts()): ?>
        <div class="related-videos">
            <h2>ویدیوهای مرتبط</h2>
            <!-- نمایش ویدیوهای مرتبط -->
        </div>
    <?php endif; ?>
</article>
```

### مرحله 4: ثبت Route

```php
Template::register('single-video', SingleController::class, 'video', 'singles.video');
```

---

## ساخت Controller جدید

اگر نیاز به Controller مجزا دارید (مثلاً برای API یا Ajax):

### ساخت فایل Controller

فایل `app/Controllers/VideoController.php` بسازید:

```php
<?php

namespace XPayMain\Controllers;

defined('ABSPATH') || exit;

use XPayMain\Core\Controller;

/**
 * VideoController
 * 
 * Handles video-related functionality
 * 
 * @package XPayMain\Controllers
 * @since 2.0.0
 */
class VideoController extends Controller
{
    /**
     * Display video gallery
     */
    public function gallery()
    {
        $data = [
            'videos' => $this->getAllVideos(),
            'categories' => $this->getVideoCategories(),
        ];

        $this->render('video.gallery', $data);
    }

    /**
     * Get all videos
     */
    private function getAllVideos()
    {
        return new \WP_Query([
            'post_type' => 'video',
            'posts_per_page' => -1,
        ]);
    }

    /**
     * Get video categories
     */
    private function getVideoCategories()
    {
        return get_terms([
            'taxonomy' => 'video_category',
            'hide_empty' => true,
        ]);
    }
}
```

### Import در functions.php

```php
use XPayMain\Controllers\VideoController;
```

### ثبت Route

```php
Template::register('video-gallery', VideoController::class, 'gallery', 'video.gallery');
```

---

## کار با Partial Views

برای استفاده مجدد از بخش‌های HTML در چندین template:

### ساخت Partial

فایل `views/partials/video-card.php` بسازید:

```php
<?php
/**
 * Video Card Partial
 * 
 * @var WP_Post $video
 */

defined('ABSPATH') || exit;
?>

<div class="video-card">
    <?php if (has_post_thumbnail($video->ID)): ?>
        <div class="video-thumbnail">
            <?php echo get_the_post_thumbnail($video->ID, 'medium'); ?>
        </div>
    <?php endif; ?>
    
    <h3><?php echo esc_html($video->post_title); ?></h3>
    <p><?php echo wp_trim_words($video->post_excerpt, 20); ?></p>
    
    <a href="<?php echo get_permalink($video->ID); ?>">
        مشاهده ویدیو
    </a>
</div>
```

### استفاده از Partial در View

```php
<?php
// در فایل view اصلی
foreach ($videos as $video): 
    include get_template_directory() . '/views/partials/video-card.php';
endforeach;
?>
```

یا با استفاده از متد `renderPartial`:

```php
// در کنترلر
public function gallery()
{
    $videos = $this->getAllVideos();
    
    $videoCards = '';
    foreach ($videos as $video) {
        $videoCards .= $this->renderToString('partials.video-card', ['video' => $video]);
    }
    
    $this->render('video.gallery', ['video_cards' => $videoCards]);
}
```

---

## دیباگ و رفع مشکل

### صفحه سفید (White Screen)

**علل احتمالی:**

1. **Route ثبت نشده:** بررسی کنید که route در `routes.php` اضافه شده باشد
2. **Template Name مطابقت ندارد:** نام در `Template Name` باید با پارامتر اول `Template::register()` یکسان باشد
3. **View فایل وجود ندارد:** بررسی کنید فایل view در مسیر صحیح ساخته شده باشد
4. **خطای PHP:** Debug mode را فعال کنید:

```php
// در wp-config.php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', true);
```

### بررسی Routes ثبت شده

برای دیباگ، می‌توانید لیست routes را ببینید:

```php
// در functions.php موقتاً اضافه کنید
add_action('init', function() {
    if (current_user_can('administrator')) {
        echo '<pre>';
        print_r(\XPayMain\Core\Template::getRoutes());
        echo '</pre>';
    }
});
```

### بررسی Template Type

برای دیدن اینکه WordPress چه template type ای تشخیص میده:

```php
// در Template.php موقتاً اضافه کنید
public static function handleTemplateRedirect()
{
    $templateType = self::getTemplateType();
    
    // Debug
    if (current_user_can('administrator')) {
        echo "<!-- Template Type: {$templateType} -->";
    }
    
    // ... ادامه کد
}
```

### خطای "View not found"

- بررسی کنید مسیر view در `routes.php` با dot notation صحیح باشد
- نام فایل باید دقیقاً با نام در مسیر مطابقت داشته باشد
- پسوند `.php` را در مسیر view نیاورید (سیستم خودکار اضافه می‌کند)

**مثال صحیح:**
```php
Template::register('faq', PageController::class, 'faq', 'pages.faq');
// فایل باید در: views/pages/faq.php
```

**مثال نادرست:**
```php
Template::register('faq', PageController::class, 'faq', 'pages/faq.php'); // ❌
```

### Controller Method پیدا نمی‌شود

- اطمینان حاصل کنید متد public است
- نام متد در `routes.php` با نام متد واقعی در کنترلر یکسان باشد
- کلاس Controller در `functions.php` import شده باشد

---

## بهترین روش‌ها (Best Practices)

### 1. نام‌گذاری

- از kebab-case برای نام template ها استفاده کنید: `user-profile`, `contact-us`
- نام متدهای کنترلر را camelCase بنویسید: `userProfile()`, `contactUs()`
- نام فایل‌های view را snake_case یا kebab-case بنویسید

### 2. ساختار داده

همیشه داده‌ها را به صورت آرایه associative از کنترلر به view بفرستید:

```php
// ✅ درست
$data = [
    'title' => 'عنوان صفحه',
    'items' => $items,
];

// ❌ نادرست
$data = $items; // نه آرایه associative
```

### 3. Validation و Security

همیشه output را escape کنید:

```php
// برای متن ساده
<?php echo esc_html($text); ?>

// برای URL
<?php echo esc_url($url); ?>

// برای attribute
<?php echo esc_attr($attribute); ?>

// برای HTML (فقط با کنترل شده)
<?php echo wp_kses_post($content); ?>
```

### 4. Performance

- از caching استفاده کنید برای query های سنگین
- از transient API وردپرس برای ذخیره موقت داده‌ها استفاده کنید
- query های غیرضروری را از view به کنترلر منتقل کنید

### 5. Documentation

همیشه در ابتدای view فایل، متغیرهای مورد انتظار را document کنید:

```php
<?php
/**
 * Page Title
 * 
 * Description of what this view does
 * 
 * @var string $title Page title
 * @var array $items List of items to display
 * @var WP_Post $post Current post object
 */
?>
```

---

## منابع بیشتر

- [مستندات MVC Architecture](mvc-architecture.md)
- [راهنمای Migration](MIGRATION-GUIDE.md)
- [تنظیمات Rank Math](rank-math-configuration.md)
- [Quick Start Guide](QUICK-START.md)

---

**آخرین بروزرسانی:** نوامبر 2025  
**نسخه تم:** 2.0.0
