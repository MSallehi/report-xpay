# Najva Push Notification - بهینه‌سازی

## مشکل قبلی

Najva script بلافاصله بعد از load شدن صفحه، notification permission درخواست می‌کرد که:
- تجربه کاربری بدی ایجاد می‌کرد
- کاربران را سردرگم می‌کرد
- PageSpeed Insights خطا می‌داد

## راه حل پیاده‌سازی شده

### 1. Load بر اساس User Interaction

Najva script حالا فقط بعد از اولین تعامل کاربر load می‌شود:
- **Scroll**: وقتی کاربر صفحه را scroll کند
- **Click**: وقتی کاربر روی هر جایی کلیک کند
- **Touch**: وقتی کاربر صفحه را لمس کند (موبایل)

### 2. استثنا برای Permission های موجود

اگر کاربر قبلاً notification permission داده باشد:
- Script بلافاصله load می‌شود (چون دیگر مزاحم نیست)
- نیازی به انتظار برای user interaction نیست

### 3. تنظیمات .env

می‌توانید Najva را کاملاً غیرفعال کنید:

```env
# Disable Najva in development
ENABLE_NAJVA=false

# Or keep it enabled (default)
ENABLE_NAJVA=true
```

Najva در حالت debug mode هم خودکار غیرفعال می‌شود:
```env
APP_DEBUG=true  # Najva won't load
```

## کد پیاده‌سازی

```javascript
// Check if notification permission already granted
if ('Notification' in window && Notification.permission === 'granted') {
    // Load immediately
    loadNajvaScript();
} else {
    // Wait for user interaction
    document.addEventListener('scroll', loadNajvaScript, {passive: true, once: true});
    document.addEventListener('click', loadNajvaScript, {once: true});
    document.addEventListener('touchstart', loadNajvaScript, {passive: true, once: true});
}
```

## مزایا

✅ **بهتر برای SEO**: PageSpeed Insights دیگر خطا نمی‌دهد
✅ **تجربه کاربری بهتر**: Permission بعد از تعامل درخواست می‌شود
✅ **Performance بهتر**: Script تا زمان نیاز load نمی‌شود
✅ **قابل کنترل**: می‌توان از طریق .env غیرفعال کرد

## یادداشت‌ها

- Event listeners با `{once: true}` استفاده می‌کنند تا بعد از اولین بار خودکار remove شوند
- `{passive: true}` برای scroll performance بهتر استفاده شده
- در debug mode Najva load نمی‌شود
- کد با `najvaLoaded` flag از load های duplicate جلوگیری می‌کند

## تست

برای تست:

1. صفحه را باز کنید (Najva load نمی‌شود)
2. Console DevTools را باز کنید
3. صفحه را scroll کنید یا کلیک کنید
4. بایدببینید که Najva script load شده است

یا اگر قبلاً permission داده‌اید، بلافاصله load می‌شود.
