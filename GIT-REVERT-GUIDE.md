# Git Revert Guide - راهنمای برگرداندن تغییرات

> **نسخه:** 1.5.1  
> **تاریخ:** 28 دسامبر 2025  
> **موضوع:** راهنمای برگرداندن commits در صورت بروز مشکل در deployment

---

## 📋 فهرست

1. [بررسی وضعیت](#check-status)
2. [روش‌های Revert](#revert-methods)
3. [GitHub Actions Error](#github-actions-error)
4. [Best Practices](#best-practices)

---

## 🔍 بررسی وضعیت {#check-status}

### مشاهده Commits اخیر

```bash
# نمایش 10 commit آخر
git log --oneline -10

# نمایش تغییرات در commit خاص
git show <commit-hash>

# نمایش فایل‌های تغییر یافته در commit
git diff-tree --no-commit-id --name-only -r <commit-hash>
```

### مشاهده وضعیت فعلی

```bash
# نمایش branch فعلی و تغییرات
git status

# نمایش تفاوت با آخرین commit
git diff HEAD

# نمایش تفاوت با commit خاص
git diff <commit-hash>
```

---

## 🔄 روش‌های Revert {#revert-methods}

### روش 1: Git Revert (پیشنهادی ✅)

**مزایا:**
- ✅ Safe - تاریخچه حفظ می‌شود
- ✅ قابل ردیابی - commit جدید ایجاد می‌شود
- ✅ قابل undo - می‌توان دوباره revert کرد
- ✅ مناسب برای کار تیمی

**دستورات:**

```bash
# Revert آخرین commit
git revert HEAD

# اگر conflict داشت، حل کنید و ادامه دهید
git add .
git revert --continue

# یا لغو revert
git revert --abort

# Push changes
git push origin master
```

**مثال:**
```bash
cd /path/to/xpay-theme

# نمایش آخرین commit
git log --oneline -1
# Output: abc1234 🐛 Fix W3C validation errors - Update 8

# Revert کردن
git revert HEAD
# Commit message پیش‌فرض: Revert "🐛 Fix W3C validation errors - Update 8"

# Push
git push origin master
```

---

### روش 2: Git Reset (⚠️ خطرناک)

**هشدار:**
- ⚠️ تاریخچه از بین می‌رود
- ⚠️ نیاز به force push
- ⚠️ باعث مشکل برای همکاران می‌شود
- ⚠️ فقط برای emergency استفاده کنید

**دستورات:**

```bash
# پیدا کردن commit مطلوب
git log --oneline -10

# Reset به commit خاص (⚠️ تغییرات working directory حفظ می‌شود)
git reset --soft <commit-hash>

# Reset به commit خاص (⚠️ تغییرات از بین می‌روند)
git reset --hard <commit-hash>

# Force push (⚠️ خطرناک!)
git push -f origin master
```

**مثال:**
```bash
# مشاهده commits
git log --oneline -5
# Output:
# abc1234 🐛 Fix W3C validation errors - Update 8
# def5678 🐛 Fix W3C validation errors - Update 7
# ghi9012 🐛 Fix W3C validation errors - Update 6
# jkl3456 ✨ Add performance optimization
# mno7890 📚 Update documentation

# Reset به commit قبل از Update 8
git reset --hard def5678

# Force push
git push -f origin master
```

---

### روش 3: Revert فایل‌های خاص (🎯 دقیق)

**مزایا:**
- ✅ فقط فایل‌های مشکل‌دار برمی‌گردند
- ✅ بقیه تغییرات حفظ می‌شوند
- ✅ Safe و قابل ردیابی

**دستورات:**

```bash
# Checkout فایل خاص از commit قبلی
git checkout HEAD~1 -- path/to/file.php

# یا از commit خاص
git checkout <commit-hash> -- path/to/file.php

# مشاهده تغییرات
git diff

# Commit
git commit -m "⏪ Revert specific files due to deployment error"

# Push
git push origin master
```

**مثال برای W3C fixes:**
```bash
# فایل‌هایی که تغییر کرده‌اند:
git checkout HEAD~1 -- wordpress/wp-content/themes/xpay_main_theme/header.php
git checkout HEAD~1 -- wordpress/wp-content/themes/xpay_main_theme/functions.php
git checkout HEAD~1 -- wordpress/wp-content/themes/xpay_main_theme/inc/Xpay_Mobile_Menu_Walker.php

# چک کردن تغییرات
git status
git diff

# Commit
git add .
git commit -m "⏪ Revert header.php, functions.php, Xpay_Mobile_Menu_Walker.php

Reason: GitHub Actions deployment error (line 231 syntax error)
Files causing issues with milanmk/actions-file-deployer
Will re-apply fixes incrementally after fixing deployment"

# Push
git push origin master
```

---

### روش 4: استفاده از GitHub UI (آسان‌ترین 🌟)

**مزایا:**
- ✅ نیاز به Git command line ندارد
- ✅ Automatic conflict handling
- ✅ Pull request برای review

**مراحل:**

1. **به GitHub Repository بروید**
   - URL: `https://github.com/<your-username>/<repo-name>`

2. **به صفحه Commits بروید**
   - کلیک روی تعداد commits (مثلاً "150 commits")

3. **پیدا کردن Commit مشکل‌دار**
   - Commit با پیام مثلاً "🐛 Fix W3C validation errors - Update 8"

4. **کلیک روی Commit**
   - صفحه جزئیات commit باز می‌شود

5. **کلیک روی دکمه "Revert"**
   - در سمت راست بالا
   - یک branch جدید ایجاد می‌شود
   - Pull request automatic ایجاد می‌شود

6. **Merge کردن Pull Request**
   - Review کردن تغییرات
   - کلیک روی "Merge pull request"
   - Confirm

7. **Pull در Local**
   ```bash
   git pull origin master
   ```

---

## 🚨 GitHub Actions Deployment Error {#github-actions-error}

### خطای فعلی

```bash
/home/runner/work/_temp/3bf9357e-4625-486f-b27a-34082abefd8f.sh: 
command substitution: line 231: syntax error near unexpected token `newline'
Error: Process completed with exit code 2.
```

### علت احتمالی

**1. Multiline Strings در PHP:**
```php
// functions.php - این ممکن است باعث مشکل شود:
add_action('wp_head', function() {
    echo '<style>
        .main-footer {
            content-visibility: auto;
        }
    </style>';
}, 100);
```

**2. Persian Characters (Farsi):**
```php
// header.php - کاراکترهای فارسی در attributes:
<a href="/" aria-label="ورود">ورود</a>
```

**3. UTF-8 BOM:**
- فایل‌های PHP با UTF-8 BOM encoding مشکل دارند

### راه‌حل‌های موقت

#### راه‌حل 1: جایگزینی Deployment Action

**فایل:** `.github/workflows/deploy.yml`

```yaml
# ❌ قبل - مشکل‌دار
- name: Deploy to Production FTP
  uses: milanmk/actions-file-deployer@master
  with:
    remote-protocol: "sftp"
    # ...

# ✅ بعد - بهتر
- name: Deploy via SFTP
  uses: SamKirkland/FTP-Deploy-Action@v4.3.4
  with:
    server: ${{ secrets.FTP_SERVER_PRODUCTION }}
    username: ${{ secrets.FTP_USERNAME_PRODUCTION }}
    password: ${{ secrets.FTP_PASSWORD_PRODUCTION }}
    protocol: sftp
    port: ${{ secrets.FTP_PORT_PRODUCTION }}
    server-dir: ${{ secrets.FTP_DIR_PRODUCTION }}
    exclude: |
      **/.git*
      **/.git*/**
      **/node_modules/**
      **/docs/**
      **/*.md
```

#### راه‌حل 2: Fix Encoding Issues

```bash
# چک کردن UTF-8 BOM
file -bi header.php
# Output: text/x-php; charset=utf-8

# حذف BOM اگر وجود دارد (Linux/Mac)
sed -i '1s/^\xEF\xBB\xBF//' header.php

# یا در PHP Storm / VS Code:
# File → Save with Encoding → UTF-8 (without BOM)
```

#### راه‌حل 3: Incremental Commits

```bash
# به جای یک commit بزرگ، چند commit کوچک:

# Commit 1: header.php changes only
git add wordpress/wp-content/themes/xpay_main_theme/header.php
git commit -m "🐛 Fix W3C: header.php attribute fixes"
git push origin master
# ✅ Test deployment

# Commit 2: functions.php changes only
git add wordpress/wp-content/themes/xpay_main_theme/functions.php
git commit -m "🐛 Fix W3C: functions.php global styles fix"
git push origin master
# ✅ Test deployment

# Commit 3: walker changes
git add wordpress/wp-content/themes/xpay_main_theme/inc/Xpay_Mobile_Menu_Walker.php
git commit -m "🐛 Fix W3C: walker duplicate li fix"
git push origin master
# ✅ Test deployment
```

---

## 💡 Best Practices {#best-practices}

### قبل از Revert

1. **✅ Backup بگیرید**
   ```bash
   # کپی فولدر theme
   cp -r wordpress/wp-content/themes/xpay_main_theme /backup/xpay_main_theme_$(date +%Y%m%d)
   
   # یا export git bundle
   git bundle create backup.bundle HEAD
   ```

2. **✅ با تیم هماهنگ کنید**
   - اطلاع دادن به همکاران
   - اطمینان از عدم push همزمان

3. **✅ Documentation بررسی کنید**
   - CHANGELOG.md
   - W3C-VALIDATION-FIXES-PART2.md

### حین Revert

1. **✅ مرحله به مرحله پیش بروید**
   ```bash
   # Dry run ابتدا
   git revert --no-commit HEAD
   git status
   # اگر مشکلی نبود:
   git revert --continue
   ```

2. **✅ Conflicts را دقیق حل کنید**
   ```bash
   # نمایش conflicted files
   git diff --name-only --diff-filter=U
   
   # حل conflict و ادامه
   git add <resolved-file>
   git revert --continue
   ```

3. **✅ Test قبل از Push**
   ```bash
   # Local test
   php -l header.php  # syntax check
   # Manual test در browser
   ```

### بعد از Revert

1. **✅ Deploy را Monitor کنید**
   - GitHub Actions → چک کردن status
   - وب‌سایت را بررسی کنید

2. **✅ Documentation بروزرسانی کنید**
   ```bash
   # CHANGELOG.md
   ## [نسخه 1.5.1] - 2025-12-28 (Rollback)
   
   ### ⏪ Rollback
   - Reverted commits due to deployment error
   - Will re-apply incrementally
   ```

3. **✅ Root Cause بررسی کنید**
   - چرا deployment fail شد؟
   - چگونه جلوگیری کنیم؟

---

## 🔧 دستورات کاربردی

### Alias های Git (اختیاری)

```bash
# افزودن به ~/.gitconfig
git config --global alias.undo 'reset --soft HEAD~1'
git config --global alias.uncommit 'reset --soft HEAD~1'
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual 'log --oneline --graph --all --decorate'

# استفاده:
git undo          # undo آخرین commit (حفظ changes)
git uncommit      # همان undo
git unstage file  # unstage فایل
git last          # نمایش آخرین commit
git visual        # نمایش tree
```

### Script خودکار Revert

```bash
#!/bin/bash
# revert-last-commit.sh

echo "🔄 Starting revert process..."

# Check git status
if [ -n "$(git status --porcelain)" ]; then
  echo "❌ Working directory is not clean. Commit or stash changes first."
  exit 1
fi

# Show last commit
echo "📋 Last commit:"
git log -1 --oneline

# Confirmation
read -p "⚠️  Are you sure you want to revert this commit? (y/N) " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "❌ Revert cancelled."
    exit 1
fi

# Revert
echo "🔄 Reverting..."
if git revert HEAD; then
    echo "✅ Revert successful!"
    echo "📤 Pushing to origin..."
    git push origin master
    echo "✅ Done!"
else
    echo "❌ Revert failed. Please resolve conflicts."
    exit 1
fi
```

**استفاده:**
```bash
chmod +x revert-last-commit.sh
./revert-last-commit.sh
```

---

## 📞 در صورت مشکل

### اگر Revert با مشکل مواجه شد:

1. **Abort کنید و دوباره تلاش کنید:**
   ```bash
   git revert --abort
   git reset --hard HEAD
   ```

2. **از GitHub UI استفاده کنید**
   - راحت‌ترین روش

3. **از Branch جدید استفاده کنید:**
   ```bash
   # ایجاد branch جدید از commit قدیمی
   git checkout -b hotfix-revert <good-commit-hash>
   git push origin hotfix-revert
   # در GitHub: create pull request → merge
   ```

4. **تماس با Team Lead**
   - برای کمک در حل conflicts پیچیده

---

## 🎯 خلاصه Quick Reference

```bash
# 🔍 بررسی
git log --oneline -10                    # نمایش commits
git status                               # وضعیت فعلی
git diff HEAD                            # تغییرات

# 🔄 Revert (Safe)
git revert HEAD                          # revert آخرین
git push origin master                   # push

# ⚠️ Reset (Dangerous)
git reset --hard <commit-hash>           # reset
git push -f origin master                # force push

# 🎯 Revert فایل خاص
git checkout HEAD~1 -- path/to/file.php  # revert file
git commit -m "Revert file"              # commit
git push origin master                   # push

# 🌟 GitHub UI
# GitHub → Commits → Select commit → Revert button
```

---

**آخرین بروزرسانی:** 28 دسامبر 2025  
**نگهداری توسط:** XPay Development Team
