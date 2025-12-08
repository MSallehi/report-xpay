# 🚀 راهنمای استفاده از Git Functions

این فایل شامل چند function مفید PowerShell برای مدیریت Git workflow در پروژه XPay است.

---

## 📥 نصب و راه‌اندازی

### روش 1: استفاده موقت (توصیه میشه برای شروع)

هر بار که PowerShell رو باز می‌کنید، در پوشه theme این دستور رو بزنید:

```powershell
. .\git-functions.ps1
```

یا:

```powershell
cd C:\Docker\xpay\wordpress\wp-content\themes\xpay_main_theme
. .\git-functions.ps1
```

✅ **مزایا:**
- سریع و آسون
- نیازی به تغییر دائمی نیست
- می‌تونی هر موقع بخوای غیرفعالش کنی

---

### روش 2: اضافه کردن دائمی به PowerShell Profile

اگه می‌خوای همیشه این function ها در دسترس باشن:

#### قدم 1: باز کردن PowerShell Profile
```powershell
notepad $PROFILE
```

اگه خطا داد که فایل وجود نداره:
```powershell
New-Item -Path $PROFILE -ItemType File -Force
notepad $PROFILE
```

#### قدم 2: اضافه کردن این خط به آخر فایل
```powershell
# XPay Git Functions
. "C:\Docker\xpay\wordpress\wp-content\themes\xpay_main_theme\git-functions.ps1"
```

#### قدم 3: ذخیره و بستن Notepad

#### قدم 4: Reload کردن Profile
```powershell
. $PROFILE
```

یا PowerShell رو ببند و دوباره باز کن.

✅ حالا هر بار که PowerShell رو باز می‌کنی، این function ها خودکار load میشن!

---

## 🎯 دستورات موجود

### 1️⃣ `sync-develop` - همگام‌سازی Develop با Master

**کاربرد:** بعد از هر hotfix روی master، develop رو همگام کن

**استفاده:**
```powershell
sync-develop
```

**چیکار میکنه:**
1. ✅ Switch به branch develop
2. ✅ Pull آخرین تغییرات develop
3. ✅ Merge از master به develop
4. ✅ Push به develop
5. ✅ نمایش پیام موفقیت

**مثال:**
```powershell
PS> sync-develop

🔄 Starting sync from master to develop...
📍 Current branch: master
➡️  Switching to develop...
⬇️  Pulling latest develop...
🔀 Merging master into develop...
⬆️  Pushing to develop...
✅ Successfully synced develop with master!
```

---

### 2️⃣ `deploy-staging` - Deploy به Staging

**کاربرد:** Push سریع به develop برای deploy به staging

**استفاده:**
```powershell
# با پیام پیش‌فرض
deploy-staging

# با پیام سفارشی
deploy-staging "feat: add new feature"
```

**چیکار میکنه:**
1. ✅ چک می‌کنه روی develop هستی
2. ✅ git add . و commit
3. ✅ Push به develop
4. ✅ نمایش لینک GitHub Actions

---

### 3️⃣ `deploy-prod` - Deploy به Production

**کاربرد:** Deploy سریع به production

**استفاده:**
```powershell
deploy-prod
```

**چیکار میکنه:**
- اگه روی **develop** هستی:
  1. ✅ Switch به master
  2. ✅ Merge develop به master
  3. ✅ Push به master
  4. ⚠️ یادآوری sync-develop

- اگه روی **master** هستی:
  1. ✅ Push به master

---

### 4️⃣ `gst` - Git Status زیبا

**کاربرد:** نمایش وضعیت git با رنگ و emoji

**استفاده:**
```powershell
gst
```

**نمایش:**
```
📊 Git Status:
On branch develop
...

📝 Recent commits:
a1b2c3d feat: add new feature
e4f5g6h fix: bug fix
...
```

---

### 5️⃣ `gbr` - نمایش Branches

**کاربرد:** نمایش تمام branch های local و remote

**استفاده:**
```powershell
gbr
```

**نمایش:**
```
🌿 Local branches:
* develop
  master
  feature/new-feature

🌍 Remote branches:
  origin/develop
  origin/master
```

---

## 📚 سناریوهای کاربردی

### سناریو 1: کار روی Feature جدید

```powershell
# 1. مطمئن شو روی develop هستی
git checkout develop

# 2. Load کن functions رو
. .\git-functions.ps1

# 3. کار رو انجام بده و تست کن
# ... edit files ...

# 4. Deploy به staging
deploy-staging "feat: new payment method"

# 5. تست در staging.xpay.co

# 6. Deploy به production
deploy-prod
```

---

### سناریو 2: Hotfix فوری روی Production

```powershell
# 1. Switch به master
git checkout master

# 2. فیکس رو انجام بده
# ... fix the bug ...

# 3. Commit و Push
git add .
git commit -m "hotfix: fix critical bug"
git push origin master

# 4. همگام کن develop رو
sync-develop
```

---

### سناریو 3: چک کردن وضعیت پروژه

```powershell
# نمایش وضعیت
gst

# نمایش branches
gbr

# نمایش تغییرات uncommitted
git diff
```

---

## ⚙️ تنظیمات پیشرفته

### تغییر رنگ‌ها

فایل `git-functions.ps1` رو باز کن و رنگ‌ها رو تغییر بده:

```powershell
# رنگ‌های موجود:
# Black, DarkBlue, DarkGreen, DarkCyan, DarkRed, DarkMagenta
# DarkYellow, Gray, DarkGray, Blue, Green, Cyan, Red, Magenta
# Yellow, White

Write-Host "متن" -ForegroundColor Green
```

### اضافه کردن Function جدید

مثال: function برای pull سریع:

```powershell
function Quick-Pull {
    git pull origin $(git rev-parse --abbrev-ref HEAD)
}
Set-Alias -Name qpull -Value Quick-Pull
```

---

## 🆘 عیب‌یابی

### مشکل: Function ها کار نمی‌کنن

**راه حل:**
```powershell
# چک کن load شده باشن
. .\git-functions.ps1

# چک کن در پوشه درست هستی
cd C:\Docker\xpay\wordpress\wp-content\themes\xpay_main_theme
```

---

### مشکل: PowerShell اجازه اجرای script نمیده

**خطا:**
```
cannot be loaded because running scripts is disabled on this system
```

**راه حل:**
```powershell
# اجرا به عنوان Administrator
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

### مشکل: Conflict هنگام merge

اگه `sync-develop` conflict داد:

```powershell
# 1. Conflict ها رو حل کن
# ... edit conflicted files ...

# 2. بعد از حل:
git add .
git commit -m "chore: resolve merge conflicts"
git push origin develop
```

---

## 💡 نکات و Tips

1. ✅ **همیشه قبل از شروع کار، `git pull` کن**
2. ✅ **Commit message های واضح بنویس**
3. ✅ **قبل از deploy به production، staging رو تست کن**
4. ✅ **بعد از هر hotfix روی master، `sync-develop` رو بزن**
5. ✅ **از `gst` برای چک کردن وضعیت استفاده کن**

---

## 🔗 لینک‌های مفید

- [مستندات کامل Deploy](DEPLOYMENT.md)
- [راهنمای سریع Deploy](DEPLOY-QUICK.md)
- [GitHub Actions](https://github.com/Xpay-wp/xpay-wp/actions)

---

## 📞 کمک بیشتر

اگه سوالی داشتی یا مشکلی پیش اومد:

1. مستندات DEPLOYMENT.md رو بخون
2. GitHub Issues رو چک کن
3. با تیم development صحبت کن

---

**آخرین بروزرسانی:** نوامبر 2025
