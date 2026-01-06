# 🎯 XPay Price Manager - Quick Start

## چیه و چرا؟

**مشکل:** قیمت تتر در صفحه اصلی: ۷۲,۵۰۰ تومان، در صفحه ارز: ۷۲,۸۰۰ تومان ❌  
**راه‌حل:** یک منبع مرکزی که همه صفحات ازش استفاده می‌کنن ✅

## نصب خودکار ✓

سیستم **به صورت خودکار** در این صفحات فعاله:
- صفحه اصلی (home slider)
- لیست ارزها (price table)
- صفحه تک ارز (calculator)

**نیاز به کار اضافه نیست!**

## تست سریع

```javascript
// توی Console مرورگر:
window.XPayPriceManager.getPrice('USDT')
// Output: { buyPrice: 72500, sellPrice: 72300, ... }
```

## استفاده سفارشی

```javascript
// Subscribe به قیمت Bitcoin
window.XPayPriceManager.subscribe('BTC', function(data) {
  console.log('قیمت BTC:', data.buyPrice);
});
```

## بیشتر بخون

📚 [مستندات کامل](./PRICE-MANAGER.md)

---

**سؤال؟** کد کامل توی فایل‌های زیر:
- `assets/js/price-manager.js` - Core
- `assets/js/adapters/*.js` - Adapters
