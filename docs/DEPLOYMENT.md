# 🚀 راهنمای Deploy

این پروژه از GitHub Actions برای deploy خودکار به سرور استفاده می‌کند.

## 📋 محیط‌های Deploy

### 1️⃣ Staging (پیش‌نمایش)
- **Branch**: `develop`
- **آدرس**: `staging.xpay.co/wp-content/themes/xpay_main_theme/`
- **استفاده**: برای تست تغییرات قبل از production

### 2️⃣ Production (اصلی)
- **Branch**: `master`
- **آدرس**: `public_html/wp-content/themes/xpay_main_theme/`
- **استفاده**: سایت اصلی و زنده

---

## 🔄 Workflow معمولی توسعه

### مرحله 1: کار روی Feature جدید
```bash
# روی branch develop کار کنید
git checkout develop

# تغییرات خود را اعمال کنید
# ... edit files ...

# Commit و Push
git add .
git commit -m "feat: add new feature"
git push origin develop
```

✅ **بعد از push به develop**:
- GitHub Action اجرا میشه
- تغییرات به `staging.xpay.co` deploy میشن
- تست می‌کنید که همه چیز درست کار می‌کنه

---

### مرحله 2: انتقال به Production (دو روش)

#### 🅰️ روش اول: Merge (پیشنهادی ✨)

بدون نیاز به switch کردن branch، مستقیم develop رو merge کنید به master:

```bash
# مطمئن شوید روی develop هستید
git checkout develop

# آخرین تغییرات رو دریافت کنید
git pull origin develop

# merge به master بدون switch
git checkout master
git pull origin master
git merge develop

# push به master
git push origin master
```

یا اگر می‌خواهید بدون switch انجام بدید:

```bash
# از همون branch develop
git fetch origin master:master
git push origin develop:master
```

✅ **مزایای این روش**:
- تاریخچه کامل تغییرات حفظ میشه
- می‌تونید rollback کنید
- چند نفر می‌تونن روی develop کار کنن
- امن‌تر و حرفه‌ای‌تره

#### 🅱️ روش دوم: Switch & Push (ساده‌تر ولی کمتر توصیه میشه)

```bash
# switch به master
git checkout master

# merge از develop
git merge develop

# push
git push origin master

# برگشت به develop برای کار بعدی
git checkout develop
```

---

## 🎯 روش پیشنهادی: Pull Request

بهترین روش استفاده از Pull Request در GitHub است:

### قدم 1: Push به develop
```bash
git checkout develop
git add .
git commit -m "feat: new feature"
git push origin develop
```

### قدم 2: ساخت Pull Request در GitHub
1. برو به repository در GitHub
2. کلیک روی "Pull requests"
3. کلیک "New pull request"
4. Set:
   - **base**: `master` (مقصد)
   - **compare**: `develop` (منبع)
5. عنوان و توضیحات بنویس
6. کلیک "Create pull request"

### قدم 3: Review و Merge
1. تغییرات رو بررسی کن
2. اگر همه چیز OK بود، کلیک "Merge pull request"
3. Confirm merge

✅ **بعد از merge**:
- GitHub Action خودکار اجرا میشه
- تغییرات به Production deploy میشن

---

## 🔐 تنظیمات مورد نیاز (یکبار اول)

در GitHub Repository → Settings → Secrets and variables → Actions، این secrets رو اضافه کنید:

- `FTP_SERVER`: آدرس سرور FTP
- `FTP_USERNAME`: نام کاربری FTP
- `FTP_PASSWORD`: رمز عبور FTP

---

## 📊 مانیتورینگ Deploy

برای دیدن وضعیت deploy:

1. برو به repository در GitHub
2. کلیک روی تب "Actions"
3. آخرین workflow رو ببین
4. اگر ❌ قرمز شد = مشکل در deploy
5. اگر ✅ سبز شد = deploy موفق

---

## 🆘 عیب‌یابی

### Deploy موفق نشد چیکار کنم؟

1. **چک کردن Logs**:
   - برو به Actions → آخرین workflow
   - روی job کلیک کن و error رو ببین

2. **مشکلات رایج**:
   - ❌ FTP credentials اشتباه است → secrets رو چک کن
   - ❌ مسیر server-dir اشتباه است → مسیر رو در deploy.yml چک کن
   - ❌ دسترسی FTP محدود است → با هاستینگ تماس بگیر

3. **تست دستی FTP**:
   ```bash
   # از FileZilla یا WinSCP به FTP وصل شو و چک کن
   ```

---

## 💡 نکات مهم

1. ✅ **همیشه اول به staging deploy کنید**
2. ✅ **staging رو کامل تست کنید قبل از production**
3. ✅ **commit message های واضح بنویسید**
4. ✅ **قبل از merge، develop رو update کنید**
5. ❌ **مستقیم روی master push نکنید**
6. ❌ **بدون تست، merge نکنید**

---

## 📞 کمک بیشتر

اگر سوالی داشتید یا مشکلی پیش اومد، issues رو در GitHub چک کنید یا issue جدید بسازید.

---

**آخرین بروزرسانی**: نوامبر 2025
