# Quick Deploy Guide

## 🚀 Deploy سریع

### Deploy به Staging
```bash
git checkout develop
git add .
git commit -m "your message"
git push origin develop
```
→ Deploy میشه به: `staging.xpay.co`

---

### Deploy به Production (روش پیشنهادی)

**از روی develop بدون switch:**
```bash
# مطمئن شو develop آخرین تغییرات رو داره
git checkout develop
git pull origin develop

# Merge به master
git checkout master
git pull origin master
git merge develop
git push origin master

# برگشت به develop
git checkout develop
```

→ Deploy میشه به: `public_html/xpay.co`

---

**یا اگر می‌خواهی از همون develop بدون switch:**
```bash
git push origin develop:master
```

---

### Deploy به Production (با Pull Request - بهترین روش)

1. Push به develop
2. برو GitHub → Pull requests → New PR
3. Base: `master`, Compare: `develop`
4. Create PR → Review → Merge
5. ✅ خودکار به production deploy میشه!

---

## 📋 چک‌لیست قبل از Production

- [ ] Staging تست شده؟
- [ ] همه features کار می‌کنن؟
- [ ] Bug نداره؟
- [ ] Commit message واضحه؟

---

## 🆘 مشکل داری؟

- چک کن: GitHub → Actions → آخرین workflow
- اگر ❌ قرمز شد → logs رو بخون
- اگر ✅ سبز شد → موفقیت آمیز بوده!

---

📚 برای جزئیات بیشتر: `DEPLOYMENT.md` رو بخون
