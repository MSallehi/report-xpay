# 🚀 راهنمای سریع - قالب XPay

این راهنما یک مرجع سریع برای شروع کار با قالب XPay است.

---

## ⚡ نکات کلیدی

### ✅ معماری MVC (جدید - نسخه 2.0)
قالب از معماری **MVC مشابه Laravel** استفاده می‌کند:

- **Controllers:** منطق و داده‌ها - در `app/Controllers/`
- **Views:** نمایش HTML - در `views/`
- **Routes:** مسیریابی - در `routes.php`

### 🎯 نحوه افزودن Template جدید

#### 1. فایل Root بسازید
```php
// faq.php در root تم
<?php
/*
Template Name: faq
*/
// این فایل فقط برای شناسایی WordPress است
```

#### 2. متد Controller اضافه کنید
```php
// در app/Controllers/PageController.php
public function faq()
{
    $data = ['title' => get_the_title()];
    $this->render('pages.faq', $data);
}
```

#### 3. View بسازید
```php
// views/pages/faq.php
<h1><?php echo esc_html($title); ?></h1>
```

#### 4. Route ثبت کنید
```php
// routes.php
Template::register('faq', PageController::class, 'faq', 'pages.faq');
```

📖 **[راهنمای کامل افزودن Template](DEVELOPER-GUIDE.md)**

---

## 📋 چک‌لیست تست Template جدید

- [ ] فایل root با `Template Name` درست ساخته شد
- [ ] متد در Controller با extends از `Controller` نوشته شد
- [ ] View در مسیر صحیح قرار دارد
- [ ] Route در `routes.php` ثبت شده
- [ ] صفحه در وردپرس با template صحیح ذخیره شده
- [ ] صفحه بدون خطا نمایش داده می‌شود

---

## 🎨 RankMath SEO (خودکار)

### تنظیمات خودکار
این موارد **بدون نیاز به تغییر در پنل Rank Math** انجام می‌شود:

1. **نام "کوین‌ها" → "خرید ارز دیجیتال"** در Schema Breadcrumb
2. **Canonical URL** در صفحات صفحه‌بندی شده نویسنده
3. **noindex, nofollow** برای Author/News/Analysis archives
4. **H3 → span** در بلوک‌های FAQ

📖 **[مستندات کامل RankMath](rank-math-configuration.md)**

---

## 🐛 عیب‌یابی سریع

### صفحه سفید (White Screen)
```bash
1. بررسی: Template Name با route یکسان است؟
2. بررسی: فایل view وجود دارد؟
3. فعال کردن Debug: WP_DEBUG = true در wp-config.php
4. بررسی: wp-content/debug.log
```

### محتوا تکرار می‌شود
```bash
این مشکل در نسخه 2.0 حل شده است.
اگر هنوز وجود دارد:
- اطمینان حاصل کنید Controller از BaseController extend می‌کند
- متد render() را از $this->render() فراخوانی کنید
```

### View پیدا نمی‌شود
```bash
خطا: "View not found: pages.faq"
راه‌حل:
1. بررسی مسیر: views/pages/faq.php
2. بررسی نام در routes.php: 'pages.faq' (dot notation)
3. بررسی پسوند: فایل باید .php داشته باشد
```

---

## � ساختار کنترلرها

### Controllers موجود:

**PageController** (12 صفحه):
- home, help, calculator, contact, terms, wheel, referrals, job, levels, bug, app, about

**ArchiveController** (5 نوع):
- coin, blog, news, analysis, category

**SingleController** (4 نوع):
- coin, blog, news, post

---

## 🔍 دیباگ Routes

برای دیدن routes ثبت شده:
```php
// موقتاً در functions.php
add_action('wp_footer', function() {
    if (current_user_can('administrator')) {
        echo '<pre>';
        print_r(\XPayMain\Core\Template::getRoutes());
        echo '</pre>';
    }
});
```

---

## 📞 کجا کمک بگیریم؟

- 🎓 **[راهنمای توسعه‌دهندگان](DEVELOPER-GUIDE.md)** - افزودن template جدید
- 🏗️ **[معماری MVC](mvc-architecture.md)** - درک ساختار
- 🔧 **[Migration Guide](MIGRATION-GUIDE.md)** - رفع مشکلات
- 📖 **[RankMath Configuration](rank-math-configuration.md)** - تنظیمات SEO
- 🏠 **[README اصلی](../README.md)** - اطلاعات کلی

---

**نسخه:** 2.0.0  
**آخرین بروزرسانی:** نوامبر 2025
