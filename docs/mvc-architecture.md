# 🎨 معماری MVC و سیستم Routing قالب XPay

## مقدمه

قالب XPay با الهام از Laravel به یک معماری MVC کامل تبدیل شده است. این سند نحوه کار با سیستم جدید را توضیح می‌دهد.

> 💡 **برای افزودن Template جدید:** به [راهنمای توسعه‌دهندگان](DEVELOPER-GUIDE.md) مراجعه کنید.

---

## 🏗️ ساختار پوشه‌ها

```
xpay_main_theme/
├── app/
│   ├── Controllers/          # کنترلرها
│   │   ├── PageController.php       # 12 صفحه سفارشی
│   │   ├── ArchiveController.php    # 5 نوع آرشیو
│   │   ├── SingleController.php     # 4 نوع پست تکی
│   │   ├── RankMathController.php   # SEO
│   │   └── ...
│   ├── Core/                 # هسته سیستم MVC
│   │   ├── Controller.php           # Base Controller (extends شود)
│   │   ├── View.php                 # سیستم View
│   │   └── Template.php             # سیستم Routing
│   ├── Models/               # مدل‌های دیتا
│   ├── Services/             # سرویس‌ها
│   └── Support/              # کلاس‌های کمکی
├── views/                    # فایل‌های View (23 فایل)
│   ├── pages/                # 12 صفحه
│   ├── archives/             # 5 آرشیو
│   ├── singles/              # 4 پست تکی
│   ├── layouts/              # قالب‌های کلی
│   └── partials/             # بخش‌های کوچک قابل استفاده مجدد
├── routes.php                # تعریف 16+ روت
└── functions.php             # ثبت کنترلرها و روت‌ها
```

---

## 🚀 نحوه کار سیستم

### 1. فلوی درخواست (Request Flow)

```
WordPress Request
    ↓
functions.php (Template::init())
    ↓
template_redirect hook
    ↓
Template::handleTemplateRedirect()
    ↓
Template::getTemplateType() (تشخیص نوع صفحه)
    ↓
Controller instantiation
    ↓
Controller method execution
    ↓
$this->render() (در Controller)
    ↓
get_header() + View::render() + get_footer()
    ↓
exit (جلوگیری از لود مجدد template توسط WordPress)
```

### 2. مثال عملی

#### ✅ قبل (روش قدیمی):
```php
// help.php
<?php
/*
Template Name: help
*/
get_header();
?>
<div class="help-page">
    <!-- کل HTML اینجا -->
</div>
<?php get_footer(); ?>
```

#### ✨ بعد (روش جدید MVC):

**1. تعریف Route (`routes.php`):**
```php
Template::register('help', PageController::class, 'help', 'pages.help');
```

**2. Controller (`app/Controllers/PageController.php`):**
```php
public function help()
{
    $data = [
        'page_title' => 'مرکز راهنما',
        // داده‌های دیگر
    ];
    
    View::render('pages.help', $data);
}
```

**3. View (`views/pages/help.php`):**
```php
<?php get_header(); ?>
<div class="help-page">
    <h1><?php echo esc_html($page_title); ?></h1>
    <!-- HTML -->
</div>
<?php get_footer(); ?>
```

---

## 📚 راهنمای استفاده

### ساخت صفحه جدید

#### مرحله 1: ایجاد View
```php
// views/pages/my-page.php
<?php
defined('ABSPATH') || exit;
get_header();
?>

<div class="my-page">
    <h1><?php echo esc_html($title); ?></h1>
    <p><?php echo esc_html($description); ?></p>
</div>

<?php get_footer(); ?>
```

#### مرحله 2: اضافه کردن متد در Controller
```php
// app/Controllers/PageController.php
public function myPage()
{
    $data = [
        'title' => get_the_title(),
        'description' => get_field('description'),
    ];
    
    View::render('pages.my-page', $data);
}
```

#### مرحله 3: ثبت Route
```php
// routes.php
Template::register('my-page-template', PageController::class, 'myPage', 'pages.my-page');
```

#### مرحله 4: ایجاد Template در WordPress
```php
// my-page-template.php (در root قالب)
<?php
/*
Template Name: my-page-template
*/
// این فایل خالی می‌ماند، سیستم routing خودکار کار می‌کند
?>
```

---

## 🎯 کار با View System

### Render کردن View

```php
// روش ساده
View::render('pages.home', $data);

// با استفاده از make (alias)
View::make('pages.contact', $data);

// Return کردن output
$output = View::render('partials.header', $data, true);
```

### Dot Notation

```php
// views/pages/help.php
View::render('pages.help');

// views/singles/coin.php
View::render('singles.coin');

// views/partials/help/faq-account.php  
View::render('partials.help.faq-account');
```

### Share کردن داده با همه View ها

```php
// در functions.php یا controller
View::share('site_name', 'ایکس پی');
View::share(['key1' => 'value1', 'key2' => 'value2']);

// حالا در هر view
echo $site_name; // ایکس پی
```

### بررسی وجود View

```php
if (View::exists('pages.custom')) {
    View::render('pages.custom');
}
```

---

## 🗺️ سیستم Routing

### انواع Template ها

```php
// صفحات سفارشی (Custom Page Templates)
Template::register('template-name', PageController::class, 'methodName', 'view.name');

// آرشیوها
Template::register('archive-coin', ArchiveController::class, 'coin', 'archives.coin');

// پست‌های تکی
Template::register('single-blog', SingleController::class, 'blog', 'singles.blog');

// صفحه اصلی
Template::register('home', PageController::class, 'home', 'pages.home');

// 404
Template::register('404', ErrorController::class, 'notFound', 'errors.404');
```

### بررسی Route ها

```php
// بررسی وجود route
if (Template::hasRoute('help')) {
    // ...
}

// دریافت همه route ها
$routes = Template::getRoutes();
```

---

## 📦 کار با Partials

Partial ها برای بخش‌های کوچک و قابل استفاده مجدد هستند.

### ساختار

```
views/partials/
├── header/
│   ├── nav.php
│   └── mobile-menu.php
├── footer/
│   └── newsletter.php
└── help/
    ├── faq-account.php
    └── contact-section.php
```

### استفاده

```php
// در view اصلی
<?php include get_template_directory() . '/views/partials/help/faq-account.php'; ?>

// یا با View system
<?php View::render('partials.help.faq-account'); ?>
```

---

## 🎨 Best Practices

### 1. جدا کردن Logic از View

❌ **بد:**
```php
// در view
<?php
$posts = new WP_Query(['post_type' => 'blog']);
while ($posts->have_posts()) : $posts->the_post();
    // HTML
endwhile;
?>
```

✅ **خوب:**
```php
// در Controller
public function blog()
{
    $posts = new WP_Query(['post_type' => 'blog']);
    View::render('archives.blog', ['posts' => $posts]);
}

// در View
<?php while ($posts->have_posts()) : $posts->the_post(); ?>
    <!-- HTML -->
<?php endwhile; ?>
```

### 2. نام‌گذاری

```php
// Controllers: PascalCase + Controller suffix
PageController, ArchiveController

// Views: kebab-case
pages/my-page.php, singles/blog-post.php

// Routes: kebab-case
'my-page-template', 'archive-coin'
```

### 3. Data Validation

```php
// همیشه داده‌ها را escape کنید
<?php echo esc_html($title); ?>
<?php echo esc_url($link); ?>
<?php echo esc_attr($attribute); ?>
<?php echo wp_kses_post($content); ?>
```

---

## 🔧 Troubleshooting

### View پیدا نمی‌شود

```php
// بررسی مسیر
echo View::path('pages.help');
// Output: /path/to/theme/views/pages/help.php

// بررسی وجود
var_dump(View::exists('pages.help'));
```

### Route کار نمی‌کند

1. بررسی کنید `Template::init()` فراخوانی شده
2. بررسی کنید `routes.php` include شده
3. نام route باید دقیقاً با template name مطابقت داشته باشد

### داده به View نمی‌رسد

```php
// در Controller
public function test()
{
    $data = ['title' => 'Test'];
    View::render('pages.test', $data);
}

// در View - استفاده از متغیر
<?php echo isset($title) ? esc_html($title) : 'No title'; ?>
```

---

## 📊 مقایسه قبل و بعد

| ویژگی | قبل | بعد |
|------|-----|-----|
| **ساختار** | Flat files | MVC Pattern |
| **کد HTML** | در root قالب | در پوشه views |
| **Logic** | در template ها | در Controllers |
| **قابلیت استفاده مجدد** | کم | زیاد (با Partials) |
| **نگهداری** | سخت | آسان |
| **تست** | دشوار | ساده‌تر |
| **Laravel-like** | ❌ | ✅ |

---

## 🎓 مثال کامل

### صفحه About Us

**1. Route:**
```php
// routes.php
Template::register('about-us', PageController::class, 'about', 'pages.about');
```

**2. Controller:**
```php
// app/Controllers/PageController.php
public function about()
{
    $data = [
        'team_members' => get_field('team_members'),
        'company_info' => get_field('company_info'),
        'stats' => [
            'users' => '10000+',
            'transactions' => '50000+',
        ],
    ];
    
    View::render('pages.about', $data);
}
```

**3. View:**
```php
// views/pages/about.php
<?php
defined('ABSPATH') || exit;
get_header();
?>

<div class="about-page">
    <h1><?php echo esc_html(get_the_title()); ?></h1>
    
    <?php if (!empty($company_info)): ?>
        <div class="company-info">
            <?php echo wp_kses_post($company_info); ?>
        </div>
    <?php endif; ?>
    
    <div class="stats">
        <div class="stat">
            <span><?php echo esc_html($stats['users']); ?></span>
            <p>کاربر</p>
        </div>
        <div class="stat">
            <span><?php echo esc_html($stats['transactions']); ?></span>
            <p>تراکنش</p>
        </div>
    </div>
    
    <?php include get_template_directory() . '/views/partials/team-members.php'; ?>
</div>

<?php get_footer(); ?>
```

**4. Partial:**
```php
// views/partials/team-members.php
<?php defined('ABSPATH') || exit; ?>

<?php if (!empty($team_members)): ?>
    <div class="team-members">
        <?php foreach ($team_members as $member): ?>
            <div class="member">
                <img src="<?php echo esc_url($member['image']); ?>" alt="<?php echo esc_attr($member['name']); ?>">
                <h3><?php echo esc_html($member['name']); ?></h3>
                <p><?php echo esc_html($member['role']); ?></p>
            </div>
        <?php endforeach; ?>
    </div>
<?php endif; ?>
```

---

## 📖 منابع بیشتر

- [Laravel Views Documentation](https://laravel.com/docs/views)
- [WordPress Template Hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/)
- [PHP Namespaces](https://www.php.net/manual/en/language.namespaces.php)

---

**نسخه:** 2.0.0  
**تاریخ:** نوامبر 2025  
**وضعیت:** ✅ فعال و آماده استفاده
