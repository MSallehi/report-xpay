# رفع خطاهای Accessibility برای PageSpeed

## 📊 خلاصه تغییرات

این مستند شامل تغییرات انجام شده برای رفع خطاهای accessibility گزارش شده توسط PageSpeed در صفحه `/coin/*` است.

**تاریخ:** 2 دسامبر 2025

---

## 🔗 1. Links do not have a discernible name

**مشکل:**
لینک‌های خالی (بدون متن یا aria-label) برای screen reader ها قابل تشخیص نیستند.

**فایل تغییر یافته:**
`templates/coins/growth-coin.php`

**قبل:**
```php
<a href="<?php echo esc_url(home_url('/') . 'coin/' . $coin['link'] . '/'); ?>"></a>
```

**بعد:**
```php
<a href="<?php echo esc_url(home_url('/') . 'coin/' . $coin['link'] . '/'); ?>" 
   aria-label="<?php echo sprintf(esc_attr__('مشاهده قیمت %s', 'xpay'), $coin['name']); ?>"></a>
```

**تغییرات اضافی:**
- بهبود `alt` تصاویر از `"currency-img"` به نام واقعی ارز:
```php
// قبل:
alt="currency-img"

// بعد:
alt="<?php echo esc_attr($coin['name']); ?>"
```

**تعداد لینک‌های اصلاح شده:** 3 عدد در این فایل

---

## 📋 2. Lists do not contain only `<li>` elements

**مشکل:**
در بخش کامنت‌ها، المان‌های `<div>` داخل `<ul class="children">` قرار داشت که از نظر HTML نامعتبر است.

**فایل تغییر یافته:**
`comments.php`

**قبل:**
```php
<div <?php comment_class('comment-box flex column'); ?> id="comment-<?php comment_ID(); ?>">
    <!-- محتوای کامنت -->
</div>
```

**بعد:**
```php
<li <?php comment_class('comment-box flex column cm-list-none'); ?> id="comment-<?php comment_ID(); ?>">
    <!-- محتوای کامنت -->
</li>
```

**تغییرات ساختاری:**

1. **تغییر container از `<div>` به `<li>`:**
```php
// قبل:
<div <?php comment_class('comment-box flex column'); ?>>

// بعد:
<li <?php comment_class('comment-box flex column cm-list-none'); ?>>
```

2. **اضافه کردن `end-callback` به `wp_list_comments`:**
```php
// قبل:
wp_list_comments([
    'callback' => 'custom_comment_callback',
    'style' => 'ul',
    // ...
]);

// بعد:
wp_list_comments([
    'callback' => 'custom_comment_callback',
    'end-callback' => 'custom_comment_end_callback',
    'style' => 'ul',
    // ...
]);
```

3. **ایجاد تابع `custom_comment_end_callback`:**
```php
function custom_comment_end_callback($comment, $args, $depth)
{
    echo '</li>';
}
```

4. **اضافه کردن CSS inline:**
```css
.comment-list,
.comment-list .children {
    list-style: none;
    padding: 0;
    margin: 0;
}

.cm-list-none {
    list-style: none;
}
```

---

## 👆 3. Touch targets do not have sufficient size or spacing

**مشکل:**
دکمه‌های pagination در Swiper (8x8px) کوچکتر از حداقل استاندارد 48x48px هستند.

**فایل‌های تغییر یافته:**
- `assets/scss/main.scss`
- `assets/css/main.css`

**قبل:**
```scss
.swiper-pagination-bullet {
    width: 8px;
    height: 8px;
    background: #ccc;
}
```

**بعد:**
```scss
.swiper-pagination-bullet {
    width: 8px;
    height: 8px;
    background: #ccc;
    /* Accessibility: Increase touch target to 48x48px while keeping visual size */
    padding: 20px;
    margin: 0 4px;
    background-clip: content-box;
    box-sizing: content-box;
}
```

**توضیح راه‌حل:**
- `padding: 20px` → ابعاد کلی را به 48x48px می‌رساند (8 + 20*2 = 48)
- `background-clip: content-box` → background فقط در 8x8px مرکزی نمایش داده می‌شود
- `box-sizing: content-box` → padding خارج از width/height محاسبه می‌شود
- نتیجه: ظاهر بصری همان 8px باقی می‌ماند ولی ناحیه قابل کلیک 48x48px است

---

## ✅ خلاصه فایل‌های تغییر یافته

| فایل | خط(های) تغییر یافته | نوع تغییر |
|------|---------------------|-----------|
| `templates/coins/growth-coin.php` | ~50, 70, 90 | aria-label + بهبود alt |
| `comments.php` | callback, end-callback | ساختار HTML |
| `assets/scss/main.scss` | ~116 | touch target CSS |
| `assets/css/main.css` | ~126 | touch target CSS |

---

## 🧪 تست

برای تست این تغییرات:

1. **PageSpeed Insights:**
   - برو به https://pagespeed.web.dev/
   - URL صفحه کوین را وارد کن: `https://xpay.co/coin/tron`
   - بخش Accessibility را چک کن

2. **دستی:**
   - با Tab key روی لینک‌ها حرکت کن و ببین screen reader چه می‌خواند
   - روی bullet های pagination کلیک کن (باید راحت‌تر باشد)

---

## 📚 منابع

- [WCAG 2.1 - Target Size](https://www.w3.org/WAI/WCAG21/Understanding/target-size.html)
- [WCAG 2.1 - Link Purpose](https://www.w3.org/WAI/WCAG21/Understanding/link-purpose-in-context.html)
- [HTML5 - List Elements](https://html.spec.whatwg.org/multipage/grouping-content.html#the-li-element)

---

**نسخه:** 4.9.10  
**نویسنده:** XPay Development Team
