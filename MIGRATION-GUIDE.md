# 🚀 راهنمای سریع - Migration به MVC

این راهنما برای انتقال فایل‌های template قدیمی به ساختار MVC جدید است.

---

## ✅ چک‌لیست Migration

- [x] ساختار Core (View, Template) ایجاد شد
- [x] Controllers ایجاد شدند
- [x] پوشه views و ساختار آن آماده است
- [x] routes.php ایجاد شد
- [x] functions.php به‌روز شد
- [ ] تست صفحه help انجام شود
- [ ] سایر صفحات منتقل شوند

---

## 📋 مراحل Migration یک Template

### مرحله 1: شناسایی Template

فایل قدیمی:
```php
// help.php (در root قالب)
<?php
/*
Template Name: help
*/
get_header();
?>
<div class="help-page">
    <!-- HTML content -->
</div>
<?php get_footer(); ?>
```

### مرحله 2: ایجاد View

```php
// views/pages/help.php
<?php
defined('ABSPATH') || exit;
get_header();
?>
<div class="help-page">
    <h1><?php echo esc_html($page_title); ?></h1>
    <!-- HTML content -->
</div>
<?php get_footer(); ?>
```

### مرحله 3: اضافه کردن Controller Method

```php
// app/Controllers/PageController.php
public function help()
{
    $data = [
        'page_title' => 'مرکز راهنما',
    ];
    
    View::render('pages.help', $data);
}
```

### مرحله 4: ثبت Route

```php
// routes.php
Template::register('help', PageController::class, 'help', 'pages.help');
```

### مرحله 5: تست

1. فایل قدیمی را تغییر نام دهید: `help.php` → `help.php.old`
2. صفحه را در مرورگر باز کنید
3. اگر کار کرد، فایل قدیمی را حذف کنید

---

## 🎯 Templates باقی‌مانده

### صفحات (Pages)
- [ ] calculator.php → views/pages/calculator.php
- [ ] contact.php → views/pages/contact.php
- [ ] terms.php → views/pages/terms.php
- [ ] wheel.php → views/pages/wheel.php
- [ ] referrals.php → views/pages/referrals.php
- [ ] job.php → views/pages/job.php
- [ ] levels.php → views/pages/levels.php
- [ ] bug.php → views/pages/bug.php
- [ ] app.php → views/pages/app.php
- [ ] about.php → views/pages/about.php
- [ ] home.php → views/pages/home.php
- [ ] home-current.php → views/pages/home-current.php

### آرشیوها (Archives)
- [ ] archive-coin.php → views/archives/coin.php
- [ ] archive-blog.php → views/archives/blog.php
- [ ] archive-news.php → views/archives/news.php
- [ ] archive-analysis.php → views/archives/analysis.php
- [ ] category.php → views/archives/category.php

### پست‌های تکی (Singles)
- [ ] single-coin.php → views/singles/coin.php
- [ ] single-blog.php → views/singles/blog.php
- [ ] single-news.php → views/singles/news.php
- [ ] single.php → views/singles/post.php

### سایر
- [ ] author.php → views/author.php (نیاز به AuthorController)
- [ ] 404.php → views/errors/404.php (نیاز به ErrorController)

---

## 🛠️ دستورات مفید

### کپی فایل template
```bash
# PowerShell
Copy-Item "help.php" "views/pages/help.php"
```

### تغییر نام فایل قدیمی
```bash
# PowerShell
Rename-Item "help.php" "help.php.old"
```

### حذف فایل قدیمی
```bash
# PowerShell
Remove-Item "help.php.old"
```

---

## ⚠️ نکات مهم

### 1. فایل template در root باید خالی باشد

```php
// help.php (در root - فقط برای WordPress)
<?php
/*
Template Name: help
*/
// این فایل خالی می‌ماند
// سیستم routing خودکار view را load می‌کند
?>
```

### 2. داده‌ها را از Controller بگیرید

❌ **در view ننویسید:**
```php
$title = get_field('title');
```

✅ **در controller بنویسید:**
```php
public function help()
{
    $data = ['title' => get_field('title')];
    View::render('pages.help', $data);
}
```

### 3. Escape کردن داده‌ها

```php
<?php echo esc_html($title); ?>
<?php echo esc_url($link); ?>
<?php echo esc_attr($attr); ?>
<?php echo wp_kses_post($content); ?>
```

---

## 🐛 مشکلات متداول

### صفحه سفید نشان می‌دهد

1. خطاها را بررسی کنید: `wp-content/debug.log`
2. بررسی کنید view وجود دارد: `View::exists('pages.help')`
3. بررسی کنید route ثبت شده: `Template::hasRoute('help')`

### View پیدا نمی‌شود

```php
// مسیر را چک کنید
echo View::path('pages.help');
```

### متغیر در view undefined است

```php
// در view
<?php echo isset($title) ? esc_html($title) : 'Default'; ?>
```

---

## 📞 کمک

- 📖 [مستندات کامل MVC](mvc-architecture.md)
- 🏠 [README اصلی](../README.md)
- 💬 تیم فنی

---

**نکته:** این فرآیند را برای هر template تکرار کنید. بعد از اطمینان از کارکرد صحیح، فایل‌های قدیمی را حذف کنید.
