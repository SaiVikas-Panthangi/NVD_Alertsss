# GitHub Actions Setup Guide

This guide walks you through setting up GitHub Actions for the Dashboard Alerts and NVD Playwright tests.

## ✅ What Has Been Done

- ✅ Enabled hourly schedule trigger in `monitor-hourly.yml` (09:00-23:00 IST)
- ✅ Added failure notifications via Teams webhook
- ✅ Created new `playwright-tests.yml` workflow for NVD tests
- ✅ Updated README with documentation and status badges
- ✅ Pushed all changes to GitHub

## 🔑 Step 1: Configure GitHub Secrets

GitHub Actions need secrets to authenticate and send notifications. Follow these steps:

### Access GitHub Secrets

1. Go to your GitHub repository: [https://github.com/SaiVikas-Panthangi/NVD](https://github.com/SaiVikas-Panthangi/NVD)
2. Click **Settings** (top-right, or go to: Repo > Settings)
3. In the left sidebar, click **Secrets and variables > Actions**
4. Click **New repository secret** (green button)

### Add Required Secrets

**For Dashboard Alerts Monitor (Required):**

#### `TEAMS_WEBHOOK_URL`
- **Where to get it**: From `config/api-monitor-config.json`, find `notifications.teamsWebhookUrl`
- **Value**: `https://default72b17115991542c09f1b4f98e5a4bc.d2.environment.api.powerplatform.com:443/powerautomate/...`
- **Steps**:
  1. Click **New repository secret**
  2. **Name**: `TEAMS_WEBHOOK_URL`
  3. **Secret**: Paste the webhook URL from config
  4. Click **Add secret**

---

**For NVD Playwright Tests (if running):**

#### `CCMA_USERNAME`
- **Where to get it**: Your CCMA login username
- **Steps**:
  1. Click **New repository secret**
  2. **Name**: `CCMA_USERNAME`
  3. **Secret**: Your username
  4. Click **Add secret**

#### `CCMA_PASSWORD`
- **Where to get it**: Your CCMA login password
- **⚠️ IMPORTANT**: Never commit this to git, use GitHub Secrets only
- **Steps**:
  1. Click **New repository secret**
  2. **Name**: `CCMA_PASSWORD`
  3. **Secret**: Your password
  4. Click **Add secret**

#### `TEST_ENVIRONMENT_URL`
- **Where to get it**: The URL of your test environment
- **Example**: `https://inutilidcplp322.idc1.level3.com:8483`
- **Steps**:
  1. Click **New repository secret**
  2. **Name**: `TEST_ENVIRONMENT_URL`
  3. **Secret**: Your test environment URL
  4. Click **Add secret**

#### `SENDGRID_API_KEY` (Optional)
- **Where to get it**: SendGrid API key (if you want email reports)
- **Steps**:
  1. Click **New repository secret**
  2. **Name**: `SENDGRID_API_KEY`
  3. **Secret**: Your SendGrid API key
  4. Click **Add secret**

---

## 🤖 Step 2: Verify Workflows Are Running

### Check Workflow Status

1. Go to your GitHub repository
2. Click **Actions** (top-right navigation)
3. You should see two workflows:
   - ✅ **Dashboard Alerts Hourly Monitor**
   - ✅ **NVD Playwright Tests**

### Trigger Manual Test Run

**Dashboard Alerts Monitor:**
1. Click **Dashboard Alerts Hourly Monitor**
2. Click **Run workflow** (right side)
3. Select **Branch**: main
4. Click **Run workflow**

**NVD Playwright Tests:**
1. Click **NVD Playwright Tests**
2. Click **Run workflow** (right side)
3. Select **Environment**: staging (or production)
4. Click **Run workflow**

---

## 📅 Step 3: Verify Scheduling

The workflows are configured to run on schedule:

### Dashboard Alerts Monitor
- **Schedule**: Every hour from 09:00 IST to 23:00 IST (03:30-17:30 UTC)
- **Cron**: `30 3-17 * * *`
- **Manual Trigger**: Available via workflow_dispatch

### NVD Playwright Tests
- **Schedule**: Daily at 08:00 UTC (13:30 IST)
- **Cron**: `0 8 * * *`
- **Push/PR Trigger**: Runs on push/PR to main or develop branches
- **Manual Trigger**: Available via workflow_dispatch

---

## 🚀 Step 4: Understanding the Workflows

### Dashboard Alerts Monitor (`monitor-hourly.yml`)

**What it does:**
- Runs the API inventory monitor
- Checks API health and response times
- Sends alerts to Teams on transitions (UP → DOWN, DOWN → UP)
- Uploads reports and logs as artifacts

**When it runs:**
- Hourly schedule (09:00-23:00 IST)
- Manual trigger via GitHub Actions UI

**Runner:**
- `self-hosted` (requires self-hosted runner configured)

**Artifacts:**
- `reports/` - Test reports and history
- `data/` - State tracking JSON files
- `logs/` - Execution logs

---

### NVD Playwright Tests (`playwright-tests.yml`)

**What it does:**
- Runs Playwright E2E tests from NVD-Narnia
- Tests CCMA login and dashboard functionality
- Generates HTML reports and screenshots
- Sends Teams notifications on failure or scheduled success

**When it runs:**
- Daily at 08:00 UTC (scheduled)
- On push to main/develop branches
- On pull requests to main/develop
- Manual trigger via GitHub Actions UI

**Runner:**
- `ubuntu-latest` (cloud-hosted GitHub runner)

**Artifacts:**
- `playwright-report/` - HTML test report
- `test-results/` - JSON test results
- `screenshots/` (on failure only) - Test screenshots

---

## 🔧 Step 5: Troubleshooting

### Workflow Not Running on Schedule?

**Possible causes:**
1. GitHub Actions might be disabled for the repository
2. Secrets are not configured
3. Runner is offline (for self-hosted workflows)

**Solutions:**
1. Check **Settings > Actions > General** - ensure "Actions Permissions" are enabled
2. Verify secrets are added (see Step 1)
3. For self-hosted workflows, verify runner is registered and active

### "Missing secret" Error?

**Solution:**
1. Go to **Settings > Secrets and variables > Actions**
2. Verify all required secrets are added
3. Check secret names match exactly (case-sensitive)
4. Re-run the workflow

### Self-Hosted Runner Issues?

**Check runner status:**
1. Go to **Settings > Actions > Runners**
2. Look for your runner and verify status is "idle"
3. If offline, restart the runner service on your machine

**Register a self-hosted runner:**
1. Go to **Settings > Actions > Runners**
2. Click **New self-hosted runner**
3. Follow GitHub's instructions to download and configure
4. Restart the runner service

---

## 📊 Step 6: Monitor Workflow Performance

### View Workflow Run History

1. Go to **Actions** tab
2. Click a workflow name (e.g., "Dashboard Alerts Hourly Monitor")
3. See all past runs with status, duration, and results

### Download Artifacts

1. Click a successful run
2. Scroll to **Artifacts** section
3. Click to download reports, screenshots, or logs

### Check Logs

1. Click a workflow run
2. Click a job (e.g., "run-monitor")
3. Expand step logs to see detailed output

---

## ✅ Verification Checklist

Before declaring success, verify:

- [ ] All GitHub Secrets are configured (TEAMS_WEBHOOK_URL, CCMA_USERNAME, CCMA_PASSWORD, etc.)
- [ ] Dashboard Alerts workflow appears in Actions tab
- [ ] NVD Playwright Tests workflow appears in Actions tab
- [ ] Manual trigger test runs successfully
- [ ] Teams notifications are received when manually triggered
- [ ] Artifacts are being uploaded after each run
- [ ] Scheduled workflows will run at expected times (check next 24-48 hours)
- [ ] Status badges in README show green ✅ (may take first successful run)

---

## 📞 Support & Troubleshooting

For more info on GitHub Actions:
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax Reference](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Scheduled Workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule)

For issues specific to your setup:
1. Check workflow run logs in GitHub Actions UI
2. Review error messages in the run logs
3. Verify secrets and configuration
4. Check self-hosted runner status if applicable

---

## 🎯 Next Steps

After GitHub Actions is set up:

1. **Monitor the first scheduled run** (wait for next scheduled time or trigger manually)
2. **Review test results** in GitHub Actions UI
3. **Check Teams notifications** to ensure alerts are being sent
4. **Download artifacts** to inspect reports
5. **Adjust configuration** as needed (e.g., environment URLs, test parameters)

---

**Status:** ✅ GitHub Actions workflows are now live and ready!
