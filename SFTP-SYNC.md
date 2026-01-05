# راهنمای SFTP Sync Scripts

دو اسکریپت PowerShell برای دانلود فایل‌های تغییر یافته از سرور SFTP

---

## 📋 فهرست

1. [نصب WinSCP](#نصب-winscp) (روش توصیه شده)
2. [نصب Posh-SSH](#نصب-posh-ssh) (روش جایگزین)
3. [پیکربندی](#پیکربندی)
4. [استفاده](#استفاده)
5. [عیب‌یابی](#عیبیابی)

---

## 1. نصب WinSCP (روش توصیه شده)

### مزایا:
- ✅ سریع‌تر و پایدارتر
- ✅ رابط گرافیکی دارد
- ✅ پشتیبانی کامل از SFTP/SCP/FTP

### نصب:

1. دانلود از: https://winscp.net/eng/download.php
2. نصب در مسیر پیش‌فرض: `C:\Program Files (x86)\WinSCP\`
3. اجرای اسکریپت: `.\Sync-ServerFiles.ps1`

---

## 2. نصب Posh-SSH (روش جایگزین)

### مزایا:
- ✅ نیازی به نصب نرم‌افزار اضافی ندارد
- ✅ فقط PowerShell Module هست
- ✅ سبک‌تر از WinSCP

### نصب:

```powershell
# نصب Posh-SSH Module
Install-Module -Name Posh-SSH -Force -Scope CurrentUser

# اجرای اسکریپت
.\Sync-ServerFiles-PoshSSH.ps1
```

---

## 3. پیکربندی

### مرحله 1: ویرایش اطلاعات سرور

فایل مورد نظر را باز کنید:
- `Sync-ServerFiles.ps1` (WinSCP)
- `Sync-ServerFiles-PoshSSH.ps1` (Posh-SSH)

### مرحله 2: تنظیم متغیرها

```powershell
# اطلاعات سرور
$SftpHost = "YOUR_SERVER_IP_OR_DOMAIN"     # مثال: "staging.xpay.co"
$SftpPort = 22                              # پورت SFTP (معمولاً 22)
$SftpUsername = "YOUR_USERNAME"             # نام کاربری
$SftpPassword = "YOUR_PASSWORD"             # رمز عبور

# مسیرها
$RemotePath = "/path/to/wordpress/wp-content/themes/xpay_main_theme"
$LocalPath = "C:\Docker\xpay\wordpress\wp-content\themes\xpay_main_theme"
```

### مثال واقعی:

```powershell
# Production Server
$SftpHost = "staging.xpay.co"
$SftpPort = 22
$SftpUsername = "xpay_user"
$SftpPassword = "MySecurePassword123!"

$RemotePath = "/home/xpay/public_html/wp-content/themes/xpay_main_theme"
$LocalPath = "C:\Docker\xpay\wordpress\wp-content\themes\xpay_main_theme"
```

### استفاده از SSH Key (امن‌تر):

اگر می‌خواهید از SSH Key استفاده کنید:

```powershell
# در WinSCP
$sessionOptions.SshPrivateKeyPath = "C:\Users\YourUser\.ssh\id_rsa.ppk"
$sessionOptions.Password = ""  # خالی بگذارید

# در Posh-SSH
$session = New-SFTPSession -ComputerName $SftpHost -Port $SftpPort `
    -Credential $credential -KeyFile "C:\Users\YourUser\.ssh\id_rsa" -AcceptKey
```

---

## 4. استفاده

### حالت‌های مختلف

#### حالت 1: دانلود فایل‌های تغییر یافته در X روز اخیر (پیش‌فرض)

```powershell
# دانلود فایل‌هایی که در 7 روز اخیر روی سرور تغییر کرده‌اند
.\Sync-ServerFiles.ps1

# با تعداد روز دلخواه
.\Sync-ServerFiles.ps1 -Days 3   # فقط 3 روز اخیر
.\Sync-ServerFiles.ps1 -Days 14  # 2 هفته اخیر
```

⚠️ **هشدار:** این روش فایل‌هایی را دانلود می‌کند که روی سرور touch شده‌اند، حتی اگر محتوای آن‌ها تغییر نکرده باشد.

---

#### حالت 2: مقایسه با Local و دانلود فقط فایل‌های جدیدتر (توصیه می‌شود ✅)

```powershell
# دانلود فقط فایل‌هایی که روی سرور جدیدتر از local هستند
.\Sync-ServerFiles.ps1 -CompareWithLocal
```

**مزایا:**
- ✅ فقط فایل‌هایی که واقعاً روی سرور تغییر کرده‌اند را دانلود می‌کند
- ✅ مقایسه timestamp بین سرور و local
- ✅ مقایسه size فایل‌ها
- ✅ نمایش دقیق تفاوت‌ها

**خروجی نمونه:**

```
=== XPay SFTP Sync Tool ===
Mode: Compare with Local (download only newer files)

Connecting to 45.82.137.127...
Connected successfully!

Scanning remote directory...
Found 152 files on server

Comparing with local files...

Files to download:

  functions.php
    Remote: 2026-01-05 14:30:15 | Size: 35.2 KB
    Local:  2026-01-04 10:15:00
    Time Diff: 1d 4h newer on server
    Reason: Remote is newer

  app/Controllers/RankMathController.php
    Remote: 2026-01-05 08:20:00 | Size: 18.7 KB
    Local:  2026-01-03 15:30:00
    Time Diff: 1d 16h newer on server
    Reason: Remote is newer

  header.php
    Remote: 2026-01-04 22:45:30 | Size: 22.1 KB
    Local Size: 22.5 KB
    Reason: Size differs

Download these 3 files? (Y/N):
```

---

#### حالت 3: دانلود فایل‌های خاص

```powershell
# دانلود فقط فایل‌های مشخص شده (بدون توجه به تاریخ)
.\Sync-ServerFiles.ps1 -SpecificFiles "functions.php,header.php"

# چند فایل با مسیر
.\Sync-ServerFiles.ps1 -SpecificFiles "functions.php,app/Controllers/RankMathController.php,header.php"

# فایل‌های CSS/JS
.\Sync-ServerFiles.ps1 -SpecificFiles "style.css,assets/js/main.js"
```

**مزایا:**
- ✅ دانلود دقیق فایل‌های مورد نظر
- ✅ نیازی به مقایسه تاریخ نیست
- ✅ سریع برای چند فایل خاص

**خروجی نمونه:**

```
=== XPay SFTP Sync Tool ===
Mode: Download Specific Files

Connecting to 45.82.137.127...
Connected successfully!

Scanning remote directory...
Found 152 files on server

Filtering for specific files...

Files to download:

  functions.php
    Remote: 2026-01-05 14:30:15 | Size: 35.2 KB
    Reason: Specific file requested

  header.php
    Remote: 2026-01-04 22:45:30 | Size: 22.1 KB
    Reason: Specific file requested

Download these 2 files? (Y/N):
```

---

### انتخاب بهترین روش

| وضعیت | روش توصیه شده |
|------|---------------|
| همکاران شما روی سرور فایل‌هایی را ویرایش کرده‌اند | `-CompareWithLocal` ✅ |
| می‌دانید دقیقاً چه فایل‌هایی تغییر کرده | `-SpecificFiles "file1.php,file2.php"` |
| می‌خواهید همه تغییرات اخیر را ببینید | `-Days 7` (پیش‌فرض) |
| فقط امروز را بررسی کنید | `-Days 1` |

---

### اجرای اسکریپت

```powershell
# مسیر scripts را باز کنید
cd C:\Docker\xpay\wordpress\wp-content\themes\xpay_main_theme\scripts

# روش 1: WinSCP (توصیه می‌شود)
.\Sync-ServerFiles.ps1 -CompareWithLocal

# روش 2: Posh-SSH
.\Sync-ServerFiles-PoshSSH.ps1
```

### خروجی نمونه (حالت پیش‌فرض):

```
=== XPay SFTP Sync Tool ===
Mode: Download files modified in last 7 days

Connecting to staging.xpay.co...
Connected successfully!

Looking for files modified after: 2026-01-01 12:00:00

Scanning remote directory...
  Searching for: *.php
  Searching for: *.js
  Searching for: *.css
Found 152 files on server

Files to download:

  functions.php
    Remote: 2026-01-05 10:30:15 | Size: 35.2 KB
    Reason: Modified in last 7 days

  app/Controllers/RankMathController.php
    Remote: 2026-01-04 22:15:00 | Size: 18.7 KB
    Reason: Modified in last 7 days

  header.php
    Remote: 2026-01-04 21:45:30 | Size: 22.1 KB
    Reason: Modified in last 7 days

Download these 3 files? (Y/N): Y

Downloading files...
  Downloading: functions.php... OK
  Downloading: app/Controllers/RankMathController.php... OK
  Downloading: header.php... OK

=== Download Summary ===
Total files: 3
Downloaded: 3
Errors: 0

Files synced successfully!

Next steps:
1. Review changes: git status
2. See differences: git diff
3. Add changes: git add -A
4. Commit: git commit -m 'Sync server changes'
5. Push: git push origin develop
```

### بعد از دانلود

```powershell
# 1. بررسی تغییرات
git status

# 2. مشاهده دقیق تفاوت‌ها
git diff

# 3. اضافه کردن همه فایل‌های تغییر یافته
git add -A

# 4. Commit
git commit -m "Sync server changes - functions.php, RankMathController, header.php"

# 5. Push به branch مورد نظر
git push origin develop
```

---

## 5. عیب‌یابی

### مشکل: "فایل‌هایی که دانلود شد درست نیستند"

**علت:** حالت پیش‌فرض فقط تاریخ تغییر روی سرور را چک می‌کند، نه محتوا.

**راه‌حل:** از حالت CompareWithLocal استفاده کنید:

```powershell
.\Sync-ServerFiles.ps1 -CompareWithLocal
```

این روش:
- ✅ فایل‌های local و remote را مقایسه می‌کند
- ✅ فقط فایل‌هایی که timestamp جدیدتری دارند را دانلود می‌کند
- ✅ فایل‌هایی که size متفاوت است را نیز دانلود می‌کند
- ✅ فایل‌هایی که local وجود ندارند را دانلود می‌کند

---

### خطا: "WinSCP not found"

```powershell
# راه‌حل: نصب WinSCP
# دانلود از: https://winscp.net/eng/download.php
```

### خطا: "Posh-SSH module not found"

```powershell
# راه‌حل: نصب Posh-SSH
Install-Module -Name Posh-SSH -Force -Scope CurrentUser
```

### خطا: "Authentication failed"

```powershell
# راه‌حل: چک کردن username و password
# اطمینان از صحت اطلاعات در اسکریپت
```

### خطا: "Connection timeout"

```powershell
# راه‌حل 1: چک کردن Firewall
# راه‌حل 2: چک کردن آدرس و پورت سرور
# راه‌حل 3: تست اتصال با WinSCP رابط گرافیکی
```

### خطا: "Permission denied"

```powershell
# راه‌حل: مطمئن شوید کاربر دسترسی خواندن به فایل‌ها دارد
# اجرای دستور روی سرور:
chmod -R 755 /path/to/theme/
```

---

## نکات امنیتی

⚠️ **هرگز رمز عبور را در Git commit نکنید!**

### راه‌حل 1: استفاده از فایل Config جداگانه

```powershell
# ایجاد فایل: scripts/sftp-config.ps1
$SftpHost = "staging.xpay.co"
$SftpUsername = "xpay_user"
$SftpPassword = "MyPassword"

# در .gitignore اضافه کنید:
scripts/sftp-config.ps1

# در اسکریپت اصلی:
. .\sftp-config.ps1
```

### راه‌حل 2: استفاده از Environment Variables

```powershell
# Set environment variable
$env:SFTP_PASSWORD = "MyPassword"

# در اسکریپت:
$SftpPassword = $env:SFTP_PASSWORD
```

### راه‌حل 3: استفاده از SSH Key (بهترین روش)

```powershell
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Copy public key to server
ssh-copy-id user@server

# استفاده در اسکریپت (بدون نیاز به password)
```

---

## گزینه‌های پیشرفته

### فیلتر کردن فایل‌های خاص

```powershell
# فقط فایل‌های PHP
$FilePatterns = @("*.php")

# فقط Controllers
$RemotePath = "/path/to/theme/app/Controllers"

# چندین پسوند
$FilePatterns = @("*.php", "*.js", "*.css", "*.json")
```

### تغییر محدوده زمانی

```powershell
# فقط امروز
.\Sync-ServerFiles.ps1 -Days 0

# یک هفته اخیر (پیش‌فرض)
.\Sync-ServerFiles.ps1 -Days 7

# یک ماه اخیر
.\Sync-ServerFiles.ps1 -Days 30
```

---

## پشتیبانی

در صورت بروز مشکل:
1. Log فایل‌ها را چک کنید
2. با WinSCP GUI تست کنید
3. دسترسی‌های SFTP سرور را بررسی کنید

---

**نویسنده:** تیم توسعه XPay  
**تاریخ:** ژانویه 2026  
**نسخه:** 1.0.0
