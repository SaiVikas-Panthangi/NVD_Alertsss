# 🚀 Jenkins Setup Guide - Dashboard Alerts Monitor

This guide walks you through setting up the Dashboard Alerts Monitor job in Jenkins.

---

## 📋 What This Monitor Does

- ✅ Monitors internal API availability (HAPI, NVD Dashboard, CCMA)
- ✅ Checks every hour (09:00-23:00 IST daily)
- ✅ Sends Teams alerts ONLY when APIs fail
- ✅ Generates reports and state tracking

---

## 🔑 Step 1: Create Jenkins Credential

### Go To:
**Jenkins > Manage Jenkins > Manage Credentials > System > Global credentials (unrestricted)**

### Add New Credential:

```
Type:           Secret text
Secret:         [Your Teams Webhook URL]
ID:             dashboard-alerts-teams-webhook
Description:    Teams Webhook for Dashboard Alerts Monitor
```

**Teams Webhook URL Format:**
```
https://outlook.webhook.office.com/webhookb2/xxxx@xxxx/IncomingWebhook/yyyy
```

**Click: Create** ✅

---

## 📦 Step 2: Create Pipeline Job

### Go To:
**Jenkins > New Item**

### Fill the Form:

```
Item name:              Dashboard-Alerts-Monitor
Type:                   Pipeline
```

**Click: OK** ✅

---

## ⚙️ Step 3: Configure Pipeline

### Go To:
**Pipeline > Definition**

### Select:
```
Definition:     Pipeline script from SCM
```

### Fill SCM Section:

```
SCM:                    Git
Repository URL:         https://github.com/SaiVikas-Panthangi/NVD_Alertsss.git
Credentials:            (none - public repo)
Branch:                 */main
Script Path:            Jenkinsfile
```

### Go To:
**Build Triggers** (check these boxes)

```
☑ Poll SCM
Schedule: (leave empty for no polling)
```

**Note:** The Jenkinsfile has `cron('''TZ=Asia/Kolkata H 9-23 * * *''')` which runs hourly 09:00-23:00 IST ✅

---

## 💾 Step 4: Save & Test

### Click: **Save** (blue button at bottom)

### Test It:

1. Go to: **Dashboard-Alerts-Monitor job page**
2. Click: **Build Now** (left sidebar)
3. Wait for build to complete
4. Click: **Build #1** to view console output

### What To Look For:

```
✅ npm install completed
✅ Monitor started
✅ API checks completed
✅ Reports generated
✅ Teams notification sent (if APIs are down)
```

---

## 📊 Step 5: Verify Outputs

### Check Artifacts:

1. Go to: **Build #1**
2. Scroll down to: **Artifacts**
3. Should see:
   - `reports/api-monitor-last-run.json`
   - `reports/api-monitor-history-YYYY-MM-DD.csv`
   - `data/api-monitor-state.json`
   - `logs/` folder

### Check Teams Channel:

Look for alert message with format:
```
🔴 API DOWN: HAPI Inventory API
Status: 500 Server Error
Last seen: [timestamp]
```

---

## 📅 Step 6: Schedule (Optional)

The job already has a schedule in the Jenkinsfile:

```groovy
cron('''TZ=Asia/Kolkata
H 9-23 * * *''')
```

This means:
- **Runs automatically** every hour
- **Only between** 09:00 AM - 11:00 PM IST
- **No manual action needed!**

---

## 🧪 Testing the Monitor

### Manual Build:
```
Jenkins > Dashboard-Alerts-Monitor > Build Now
```

### View Logs:
```
Jenkins > Build #X > Console Output
```

### Check API Status:
```
Jenkins > Build #X > Artifacts > reports/api-monitor-last-run.json
```

---

## 🔧 What Was Configured

### **config/api-monitor-config.json:**

```json
{
  "staticApis": [
    "HAPI Inventory API" → https://inutilidcplp322.idc1.level3.com:8483/hapi/inventory,
    "NVD Dashboard API" → https://connect.lumen.com/enterprise/network/visibility,
    "CCMA Portal" → https://admin-controlcenter.lumen.com/admin/ccma/home
  ],
  "notifications": {
    "teamsWebhookUrl": "",  ← Jenkins credential fills this
    "bulkFailureAlert": {
      "enabled": true,
      "channels": ["teams"],
      "minDownCount": 1  ← Alert on first API failure
    }
  }
}
```

### **Jenkinsfile:**

```groovy
pipeline {
  agent any
  
  triggers {
    cron('''TZ=Asia/Kolkata
H 9-23 * * *''')  // Hourly 09:00-23:00 IST
  }
  
  environment {
    TEAMS_WEBHOOK_URL = credentials('dashboard-alerts-teams-webhook')
  }
  
  stages {
    stage('Run Monitor') {
      sh 'npm run monitor:run'
    }
  }
}
```

---

## ✅ Verification Checklist

Before declaring it "live":

- [ ] Created Jenkins credential: `dashboard-alerts-teams-webhook`
- [ ] Created Pipeline job: `Dashboard-Alerts-Monitor`
- [ ] Set SCM to Git with correct repository
- [ ] Built job manually - completed successfully
- [ ] Saw artifacts in build (reports, data)
- [ ] Teams received notification (if APIs down)
- [ ] Understood schedule: Hourly 09:00-23:00 IST

---

## 🎯 Next Steps

1. **Setup Complete!** Job is ready to run
2. **First build** may take 2-3 minutes (installing dependencies)
3. **Subsequent builds** take 30 seconds
4. **Automatic runs** start at next scheduled hour
5. **Monitor teams channel** for alerts

---

## 📞 Troubleshooting

| Issue | Solution |
|-------|----------|
| **Build fails with "npm not found"** | Jenkins agent needs Node.js installed |
| **No Teams notification received** | Check webhook URL is correct in credential |
| **"api-monitor-config.json not found"** | File must be in `config/` folder |
| **APIs showing as DOWN incorrectly** | Check timeout settings in config file |
| **Builds taking too long** | First run installs dependencies (~2min) |

---

## 📝 File Locations

```
GitHub Repo:
https://github.com/SaiVikas-Panthangi/NVD_Alertsss

Key Files:
├─ Jenkinsfile                    (Job configuration)
├─ config/api-monitor-config.json (APIs to monitor)
├─ src/api-inventory-monitor.js   (Main logic)
└─ package.json                   (Dependencies)
```

---

**✨ Your Dashboard Alerts Monitor is Ready!**

Start the first build and watch the magic happen! 🚀
