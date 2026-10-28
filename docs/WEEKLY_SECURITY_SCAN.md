# 📅 راهنمای اجرای بررسی امنیتی هفتگی

این مستندات راهنمای کامل برای اجرای بررسی امنیتی هفتگی در پروژه i-Drill است.

---

## 🎯 بررسی امنیتی هفتگی چیست؟

بررسی امنیتی هفتگی شامل اجرای کامل تمام security checks برای شناسایی آسیب‌پذیری‌ها در:
- کد Python (Bandit)
- وابستگی‌ها (pip-audit, Safety)
- Secrets و credentials (TruffleHog)
- Dockerfile (Hadolint)
- Container و File System (Trivy)
- Static Analysis (Semgrep)

---

## ⏰ زمان اجرای خودکار

بررسی امنیتی هفتگی به صورت خودکار اجرا می‌شود:

**زمان**: هر یکشنبه ساعت 2:00 AM UTC

**Workflow**: `.github/workflows/security.yml`

**Cron Expression**: `0 2 * * 0`

### تبدیل به زمان محلی

| منطقه زمانی | زمان اجرا |
|-------------|-----------|
| UTC | یکشنبه 02:00 |
| تهران (IRST) | یکشنبه 05:30 |
| نیویورک (EST) | شنبه 21:00 |
| لندن (GMT) | یکشنبه 02:00 |
| توکیو (JST) | یکشنبه 11:00 |

---

## 🔄 روش‌های اجرا

### 1. اجرای خودکار (Scheduled)

بررسی به صورت خودکار در زمان تعیین شده اجرا می‌شود.

**بررسی وضعیت**:
1. بروید به: `https://github.com/[owner]/[repo]/actions`
2. workflow `Security Checks` را پیدا کنید
3. بررسی کنید که scheduled runs به درستی اجرا می‌شوند

---

### 2. اجرای دستی (Manual Dispatch)

#### از GitHub Actions UI:

1. بروید به: `https://github.com/[owner]/[repo]/actions`
2. workflow `Security Checks` را انتخاب کنید
3. روی `Run workflow` کلیک کنید
4. Branch را انتخاب کنید (معمولاً `main`)
5. روی `Run workflow` کلیک کنید

#### از GitHub CLI:

```bash
# اجرای workflow
gh workflow run security.yml

# بررسی وضعیت
gh run list --workflow=security.yml

# مشاهده نتایج آخرین اجرا
gh run view --web
```

---

### 3. اجرای محلی (Local)

برای اجرای بررسی امنیتی به صورت محلی:

#### نصب ابزارها

```bash
# فعال کردن virtual environment
source venv/bin/activate  # Linux/Mac
.\venv\Scripts\activate   # Windows

# نصب security tools
pip install -r requirements/dev.txt
```

#### اجرای Bandit

```bash
cd i-drill
bandit -r src/ \
  -f screen \
  -ll \
  --exclude src/backend/tests,src/tests
```

#### اجرای pip-audit

```bash
cd i-drill

# بررسی backend dependencies
pip-audit --requirement requirements/backend.txt --desc

# بررسی ML dependencies
pip-audit --requirement requirements/ml.txt --desc

# بررسی dev dependencies
pip-audit --requirement requirements/dev.txt --desc
```

#### اجرای Safety

```bash
cd i-drill
safety check \
  --file requirements/backend.txt \
  --file requirements/ml.txt \
  --file requirements/dev.txt
```

---

## 📊 بررسی نتایج

### 1. GitHub Actions UI

**مسیر**: 
- `Actions` → `Security Checks` → [Latest run]

**اطلاعات موجود**:
- وضعیت هر job (✅ موفق / ❌ ناموفق)
- Logs هر step
- Artifacts
- Workflow summary

---

### 2. GitHub Security Tab

**مسیر**: 
- `Security` → `Code scanning`

**نتایج**:
- Trivy findings (file system و Docker image)
- Semgrep findings
- CodeQL findings

**فیلترها**:
- Severity (Critical, High, Medium, Low)
- Tool (Trivy, Semgrep, CodeQL)
- Status (Open, Closed, Dismissed)

---

### 3. Artifacts

**مسیر**: 
- `Actions` → [Workflow run] → `Artifacts`

**Artifacts موجود**:
- `bandit-report.json` - Bandit findings
- `pip-audit-*.json` - Dependency vulnerabilities
- `safety-report.json` - Safety check results
- `trufflehog-results.json` - Secret scanning results
- `hadolint-report.json` - Dockerfile issues
- `trivy-*-results.sarif` - Trivy scan results
- `semgrep.sarif` - Semgrep analysis

---

## 🔔 دریافت اعلان‌ها

### 1. GitHub Notifications

**تنظیمات**:
1. بروید به: `Settings` → `Notifications`
2. فعال کردن:
   - ✅ Actions (workflow runs)
   - ✅ Security alerts

### 2. Email Notifications

برای دریافت email برای:
- Security alerts
- Workflow failures
- New vulnerabilities

**تنظیمات**:
1. `Settings` → `Notifications` → `Email`
2. انتخاب: `Security alerts`, `Actions`

### 3. Slack/Teams Integration

برای یکپارچه‌سازی با Slack یا Microsoft Teams:

**GitHub Apps**:
- [Slack App for GitHub](https://github.com/integrations/slack)
- [Microsoft Teams App](https://github.com/integrations/microsoft-teams)

---

## 📝 پیگیری و رفع آسیب‌پذیری‌ها

### 1. اولویت‌بندی

**اولویت‌ها**:
1. 🔴 **Critical** - رفع فوری
2. 🟠 **High** - رفع در 1 هفته
3. 🟡 **Medium** - رفع در 1 ماه
4. 🟢 **Low** - رفع در زمان مناسب

### 2. ایجاد Issue

برای هر آسیب‌پذیری مهم:

```markdown
## Security Vulnerability: [Title]

**Severity**: Critical/High/Medium/Low
**Tool**: Bandit/pip-audit/Trivy/...
**Location**: `path/to/file.py:line`

### Description
[توضیحات آسیب‌پذیری]

### Impact
[تأثیر آسیب‌پذیری]

### Solution
[راه‌حل پیشنهادی]
```

### 3. رفع آسیب‌پذیری

1. ایجاد branch جدید:
   ```bash
   git checkout -b fix/security-[issue-number]
   ```

2. رفع آسیب‌پذیری

3. تست:
   ```bash
   # اجرای security checks محلی
   bandit -r src/
   pip-audit --requirement requirements/backend.txt
   ```

4. ایجاد Pull Request:
   - Title: `fix(security): [description]`
   - Label: `security`
   - Reviewer: Security team

---

## 📅 تقویم بررسی امنیتی

### بررسی هفتگی (Scheduled)

- **زمان**: هر یکشنبه ساعت 2:00 AM UTC
- **Workflow**: `security.yml`
- **مدت زمان**: ~15-30 دقیقه
- **هزینه**: رایگان (در GitHub Actions free tier)

### بررسی مداوم

- **در هر Push**: Quick security scan (در `ci.yml`)
- **در هر PR**: Full security checks
- **Manual**: هر زمان که نیاز باشد

---

## 🔧 تنظیمات Scheduled Workflow

### تغییر زمان اجرا

اگر می‌خواهید زمان اجرای scheduled workflow را تغییر دهید:

**فایل**: `.github/workflows/security.yml`

```yaml
schedule:
  # Format: minute hour day month weekday
  - cron: '0 2 * * 0'  # یکشنبه 2 صبح UTC
```

**مثال‌ها**:
- `0 2 * * 0` - یکشنبه 2 صبح UTC
- `0 3 * * 1` - دوشنبه 3 صبح UTC
- `0 0 * * 0` - یکشنبه نیمه شب UTC
- `0 6 * * 0` - یکشنبه 6 صبح UTC (09:30 IRST)

### غیرفعال کردن Scheduled Run

اگر می‌خواهید scheduled run را موقتاً غیرفعال کنید:

```yaml
on:
  # schedule:
  #   - cron: '0 2 * * 0'
  # Comment out the schedule section
  push:
    branches: [ main, develop, master ]
  pull_request:
    branches: [ main, develop, master ]
  workflow_dispatch:
```

---

## ✅ چک‌لیست بررسی هفتگی

هر هفته این موارد را بررسی کنید:

- [ ] Workflow `Security Checks` به درستی اجرا شده است
- [ ] هیچ Critical یا High severity vulnerability جدیدی وجود ندارد
- [ ] نتایج Trivy بررسی شده است
- [ ] نتایج Semgrep بررسی شده است
- [ ] نتایج Bandit بررسی شده است
- [ ] Dependency vulnerabilities بررسی شده است
- [ ] Secrets scanning بررسی شده است
- [ ] Dockerfile issues بررسی شده است
- [ ] Issues برای vulnerabilities ایجاد شده است
- [ ] Vulnerabilities قبلی رفع شده‌اند

---

## 📊 گزارش هفتگی

برای تهیه گزارش هفتگی:

### از GitHub CLI:

```bash
# دریافت آخرین workflow run
gh run list --workflow=security.yml --limit 1

# دریافت جزئیات
gh run view [run-id] --web

# دانلود artifacts
gh run download [run-id]
```

### Template گزارش:

```markdown
# Security Scan Report - Week of [Date]

## Summary
- **Date**: [Date]
- **Status**: ✅ Pass / ⚠️ Warnings / ❌ Failures
- **Total Findings**: [Number]

## Findings by Severity
- 🔴 Critical: [Number]
- 🟠 High: [Number]
- 🟡 Medium: [Number]
- 🟢 Low: [Number]

## Tool Results
- **Bandit**: [Issues found]
- **pip-audit**: [Vulnerabilities found]
- **Safety**: [Issues found]
- **Trivy**: [Vulnerabilities found]
- **Semgrep**: [Issues found]

## Actions Taken
- [ ] Vulnerabilities fixed
- [ ] Dependencies updated
- [ ] Issues created
- [ ] Next review scheduled
```

---

## 🆘 عیب‌یابی

### مشکل: Scheduled workflow اجرا نمی‌شود

**راه‌حل‌ها**:
1. بررسی کنید که repository در GitHub است (نه فقط local)
2. بررسی کنید که workflow file در `.github/workflows/` موجود است
3. بررسی کنید که syntax YAML صحیح است
4. بررسی کنید که scheduled workflows در repository فعال هستند

### مشکل: Workflow خیلی دیر اجرا می‌شود

**توضیح**: GitHub Actions scheduled workflows ممکن است با تأخیر اجرا شوند (تا 15 دقیقه)

**راه‌حل**: استفاده از workflow_dispatch برای اجرای فوری

### مشکل: نتایج قدیمی هستند

**راه‌حل**: اجرای manual workflow برای دریافت آخرین نتایج

```bash
gh workflow run security.yml
```

---

## 📚 منابع بیشتر

- [GitHub Actions Scheduled Events](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule)
- [Cron Expression Format](https://crontab.guru/)
- [Security Checks Documentation](./SECURITY_CI_CD.md)
- [GitHub Security Features](https://docs.github.com/en/code-security)

---

**آخرین به‌روزرسانی**: نوامبر 2025  
**وضعیت**: ✅ فعال - اجرای هفتگی هر یکشنبه 2:00 AM UTC







