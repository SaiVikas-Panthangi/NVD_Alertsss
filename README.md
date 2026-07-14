# Dashboard Alerts API Monitor

[![Dashboard Alerts Monitor](https://github.com/SaiVikas-Panthangi/NVD/actions/workflows/monitor-hourly.yml/badge.svg)](https://github.com/SaiVikas-Panthangi/NVD/actions/workflows/monitor-hourly.yml)
[![Playwright Tests](https://github.com/SaiVikas-Panthangi/NVD/actions/workflows/playwright-tests.yml/badge.svg)](https://github.com/SaiVikas-Panthangi/NVD/actions/workflows/playwright-tests.yml)

This folder contains a standalone monitor that:
- Reads API URLs from your inventory page and/or static API list.
- Calls each API and checks status code + response time.
- Sends alert notifications only on state transitions (UP -> DOWN, DOWN -> UP).
- Writes run summary and CSV history for reporting.

## Required Fields to Fill

Update `config/api-monitor-config.json`:

1. `inventory.inventoryPageUrl`
- Set the inventory page URL if you want auto extraction of links.

2. `inventory.extractMode`
- `fetch`: for public HTML inventory pages.
- `playwright`: for authenticated/dynamic inventory pages.

3. `inventory.urlPattern`
- Regex filter to keep only API URLs from inventory links.
- Example: `/api/|apigee|nvd`

4. `staticApis[].url`
- Add known API URLs directly (recommended for initial setup).

5. `notifications.teamsWebhookUrl`
- Teams Incoming Webhook URL.

6. `notifications.email`
- `enabled`: true/false
- `from`: verified sender in SendGrid
- `recipients`: list of alert recipients
- Also set environment variable: `SENDGRID_API_KEY`

7. `apiCheck`
- `timeoutMs`, `slowThresholdMs`, `successStatusCodes`, optional headers.

## Run

```bash
npm install
npm run monitor:run
```

## Scheduler Ownership

Multiple schedulers available to avoid single point of failure:

- **Jenkins** (PRIMARY): Runs hourly from 09:00 IST to 23:00 IST
- **GitHub Actions** (SECONDARY): Runs hourly via scheduled workflow on the same schedule
- **Windows Task Scheduler** (OPTIONAL): `\DashboardAlerts_Monitor_Hourly` runs `scripts/run-monitor-scheduled.ps1`

### To prevent duplicate alerts, keep only ONE active scheduler:

**Option 1: Use Jenkins only** (recommended for existing setup)
```powershell
# Disable Windows Task Scheduler
schtasks /Change /TN "\DashboardAlerts_Monitor_Hourly" /Disable

# Disable GitHub Actions by removing schedule trigger from .github/workflows/monitor-hourly.yml
```

**Option 2: Use GitHub Actions** (cloud-native, no self-hosted Jenkins needed)
```powershell
# Disable Windows Task Scheduler
schtasks /Change /TN "\DashboardAlerts_Monitor_Hourly" /Disable

# Keep GitHub Actions enabled
# Disable Jenkins job or remove it
```

## GitHub Actions Setup

GitHub Actions is now configured with two workflows:

### 1. Dashboard Alerts Monitor (`monitor-hourly.yml`)
- **Trigger**: Hourly schedule + Manual trigger (`workflow_dispatch`)
- **Schedule**: 09:00-23:00 IST (03:30-17:30 UTC)
- **Runner**: Self-hosted (requires internal API access)
- **Secrets Required**: `TEAMS_WEBHOOK_URL`

### 2. NVD Playwright Tests (`playwright-tests.yml`)
- **Trigger**: Push/PR on main/develop + Daily schedule + Manual trigger
- **Schedule**: Daily at 08:00 UTC (13:30 IST)
- **Runner**: Ubuntu latest
- **Secrets Required**: `CCMA_USERNAME`, `CCMA_PASSWORD`, `TEST_ENVIRONMENT_URL`, `TEAMS_WEBHOOK_URL`

### Required GitHub Secrets

Add these in **GitHub Repo > Settings > Secrets and variables > Actions**:

```
# Dashboard Alerts Monitor
TEAMS_WEBHOOK_URL              # Required for both workflows

# NVD Playwright Tests (if using playwright-tests.yml)
CCMA_USERNAME                  # CCMA login username
CCMA_PASSWORD                  # CCMA login password
TEST_ENVIRONMENT_URL           # Test environment URL
SENDGRID_API_KEY              # Optional, for email reports
```

### How to Add Secrets:
1. Go to: **GitHub Repo > Settings > Secrets and variables > Actions**
2. Click **New repository secret**
3. Enter secret name and value
4. Click **Add secret**

### View Workflow Runs:
- Go to: **GitHub Repo > Actions**
- Click workflow name to see execution history
- Click specific run to see logs and artifacts

## Jenkins Setup

The included [Jenkinsfile](Jenkinsfile) is configured to:
- run hourly from 09:00 IST to 23:00 IST
- install Node dependencies with `npm ci`
- install the Playwright Chromium browser required by `checkMode: "playwright"`
- run `npm run monitor:run`
- archive `reports`, `logs`, and `data` artifacts after each run

Jenkins credentials expected:

- `dashboard-alerts-teams-webhook`: required
- `dashboard-alerts-sendgrid-api-key`: optional, only if email alerts are enabled

## Outputs

- `data/api-monitor-state.json`: state tracking for no-spam alerts.
- `reports/api-monitor-last-run.json`: latest run summary.
- `reports/api-monitor-history-YYYY-MM-DD.csv`: daily historical checks.
