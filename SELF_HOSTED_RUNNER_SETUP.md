# Self-Hosted Runner Setup Guide

This guide will walk you through setting up a **GitHub self-hosted runner** on your Windows machine to run Dashboard Alerts workflows.

## 🎯 Why Self-Hosted Runner?

The Dashboard Alerts Monitor workflow needs to:
- Access internal/private APIs on your network
- Maintain scheduled hourly execution on your local infrastructure
- Access local configuration and resources

**GitHub's cloud runners (ubuntu-latest, windows-latest) cannot access internal resources**, so we need a self-hosted runner on your local machine.

---

## 📋 Prerequisites

✅ Verify you have:
- Windows 10/11 machine with Administrator access
- GitHub account with repository access
- Git installed on your machine
- Node.js installed (v18+)
- `.NET Framework 4.6.2+` (for runner service)

**Check your setup:**
```powershell
# Check Windows version
[System.Environment]::OSVersion

# Check .NET Framework
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP" /s
```

---

## 🚀 Step 1: Download the Runner

### **1.1 Access Runner Configuration Page**

1. Go to your GitHub repository: **https://github.com/SaiVikas-Panthangi/NVD**
2. Click **Settings** (top-right)
3. In the left sidebar, click **Actions > Runners**
4. Click the green **"New self-hosted runner"** button

### **1.2 Select Runner Configuration**

You'll see options:
- **Runner OS**: Select **Windows**
- **Architecture**: Select **x64** (for most machines)

Click these to display the download commands.

---

## 🔧 Step 2: Download Runner Software

### **2.1 Create Runner Directory**

On your Windows machine, create a directory for the runner:

```powershell
# Open PowerShell as Administrator
# Create runner directory
mkdir "C:\GitHub\Actions-Runner"
cd "C:\GitHub\Actions-Runner"
```

### **2.2 Download Runner Package**

GitHub will show you a download command. It looks like:

```powershell
Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.xxx.x/actions-runner-win-x64-2.xxx.x.zip -OutFile actions-runner-win-x64-2.xxx.x.zip
```

**Copy and paste the exact command from GitHub Settings page into PowerShell:**

```powershell
# Example (use the exact URL from GitHub):
Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.316.0/actions-runner-win-x64-2.316.0.zip -OutFile actions-runner-win-x64-2.316.0.zip
```

**Wait for download to complete** (~100-200 MB).

### **2.3 Extract the Archive**

```powershell
# Extract the zip file
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::ExtractToDirectory("$PWD\actions-runner-win-x64-2.316.0.zip", "$PWD")

# Verify extraction
dir
```

You should see files like: `config.cmd`, `run.cmd`, `run.ps1`, etc.

---

## 🔐 Step 3: Register the Runner

### **3.1 Generate Registration Token**

Back in GitHub Settings page, you'll see:

**On Ubuntu / Linux:**
```bash
./config.sh --url <REPOSITORY_URL> --token <TOKEN>
```

**For Windows, it will be:**
```powershell
.\config.cmd --url <REPOSITORY_URL> --token <TOKEN>
```

**Copy the exact command from GitHub** (the token is temporary and unique to your setup).

### **3.2 Run Configuration Command**

In PowerShell (still in `C:\GitHub\Actions-Runner` directory):

```powershell
# Run the exact command from GitHub (example):
.\config.cmd --url https://github.com/SaiVikas-Panthangi/NVD --token ABCDEFGHIJKLMNOP1234567890
```

### **3.3 Answer Configuration Questions**

The runner will ask you several questions:

```
Enter the name of the runner group to add this runner to? (default)
[Press Enter for default]

Enter the name of the runner to configure as? (DESKTOP-ABC123)
[Recommended: Keep default or use meaningful name like "Dashboard-Alerts-Runner"]

Enter the work directory? (_work)
[Press Enter for default]

Enter additional labels (optional):
[Press Enter or add labels like: windows,dashboard-alerts,internal-api]

Enter name of work folder? (_work)
[Press Enter for default]
```

**Expected output after completion:**
```
√ Connected to GitHub

Current runner version: 2.316.0
√ Settings Saved.
```

---

## ✅ Step 4: Install & Start Runner Service

### **4.1 Install as Windows Service**

**Run PowerShell as Administrator** and execute:

```powershell
cd "C:\GitHub\Actions-Runner"

# Install as service (creates "GitHub Actions Runner" service)
.\config.cmd --url https://github.com/SaiVikas-Panthangi/NVD --token ABCDEFGHIJKLMNOP1234567890 --svc --svc-user NT AUTHORITY\SYSTEM
```

⚠️ **Note:** Use your actual token from GitHub Settings page.

### **4.2 Verify Service Installation**

```powershell
# Check if service is installed
Get-Service | Where-Object {$_.Name -like "*GitHub*"}

# Should return something like:
# Status   Name                DisplayName
# ------   ----                -----------
# Running  GitHubActionsRunnerSvc  GitHub Actions Runner (...)
```

### **4.3 Start the Service**

```powershell
# Start the runner service
Start-Service -Name "GitHub Actions Runner (DESKTOP-ABC123)"

# Or if you don't know the exact name:
Get-Service | Where-Object {$_.Name -like "*GitHubActionsRunner*"} | Start-Service
```

### **4.4 Verify Runner is Active**

Wait 10-15 seconds, then check:

```powershell
# Check service status
Get-Service | Where-Object {$_.Name -like "*GitHub*"}

# Should show: Running
```

---

## 🔍 Step 5: Verify in GitHub

### **5.1 Check Runner Status in GitHub**

1. Go to GitHub Repo > **Settings > Actions > Runners**
2. Look for your runner in the list
3. **Status should be: 🟢 Idle** (green dot)

### **5.2 Check Runner Details**

Click your runner name to see:
- ✅ Name
- ✅ Status (Idle / Busy / Offline)
- ✅ Last Activity timestamp
- ✅ Workflow runs history

---

## 🧪 Step 6: Test the Runner

### **6.1 Trigger Manual Workflow**

1. Go to GitHub Repo > **Actions**
2. Click **"Dashboard Alerts Hourly Monitor"**
3. Click **"Run workflow"** (top-right)
4. Select **Branch: main**
5. Click **"Run workflow"**

### **6.2 Monitor Execution**

The workflow should start immediately and you should see:
1. Status changes to 🟡 "in progress"
2. In GitHub: A new run appears with status "In progress"
3. The runner status in Settings should change to **Busy**

### **6.3 Check Logs**

1. Click the running workflow
2. Click the **"run-monitor"** job
3. Expand each step to see logs
4. Look for `npm run monitor:run` execution

### **6.4 Verify Completion**

✅ Success indicators:
- Workflow status: ✅ **Passed** (green checkmark)
- Runner status: 🟢 **Idle** (back to idle)
- Artifacts uploaded: `monitor-reports` available
- Teams notification received (if webhook working)

---

## 🛠️ Step 7: Manage the Runner Service

### **Start the Runner**
```powershell
Get-Service | Where-Object {$_.Name -like "*GitHubActionsRunner*"} | Start-Service
```

### **Stop the Runner**
```powershell
Get-Service | Where-Object {$_.Name -like "*GitHubActionsRunner*"} | Stop-Service
```

### **Restart the Runner**
```powershell
Get-Service | Where-Object {$_.Name -like "*GitHubActionsRunner*"} | Restart-Service
```

### **Check Service Status**
```powershell
Get-Service | Where-Object {$_.Name -like "*GitHubActionsRunner*"}
```

### **View Service Logs**
```powershell
# Open Event Viewer and look for "GitHub Actions"
eventvwr
```

---

## 📊 Troubleshooting

### **Runner Shows "Offline" in GitHub**

**Problem:** Runner status is 🔴 offline

**Solutions:**
1. **Check if service is running:**
   ```powershell
   Get-Service | Where-Object {$_.Name -like "*GitHubActionsRunner*"}
   # If Status is "Stopped", restart it:
   Start-Service -Name "GitHub Actions Runner (*)"
   ```

2. **Check network connectivity:**
   ```powershell
   Test-NetConnection github.com -Port 443
   # Should show: TcpTestSucceeded: True
   ```

3. **Restart the runner:**
   ```powershell
   Stop-Service -Name "GitHub Actions Runner (*)"
   Start-Sleep -Seconds 5
   Start-Service -Name "GitHub Actions Runner (*)"
   ```

4. **Reinstall if persistent:**
   - Remove service: `.\config.cmd remove`
   - Reconfigure: `.\config.cmd --url ... --token ...`

---

### **Workflow Stays "Queued"**

**Problem:** Workflow is queued but not running

**Cause:** No idle runners available

**Solution:**
1. Verify runner is Idle (not Busy)
2. Check runner has correct labels if specified in workflow
3. Restart runner service

---

### **"Cannot connect to GitHub" Error**

**Problem:** Runner fails to register

**Solutions:**
1. **Check GitHub token is valid** (tokens expire after ~1 hour)
   - Get new token from GitHub Settings > Actions > Runners
   - Re-run config command with new token

2. **Check internet connection:**
   ```powershell
   Test-NetConnection github.com -Port 443
   ```

3. **Check firewall:**
   - Allow `github.com` through Windows Firewall
   - Allow PowerShell to connect to network

4. **Verify GitHub URL:**
   - Must be: `https://github.com/SaiVikas-Panthangi/NVD`
   - Check for typos

---

### **Permission Denied Error**

**Problem:** "Access Denied" when running commands

**Solution:**
- Run PowerShell **as Administrator**
- Right-click PowerShell > "Run as administrator"

---

### **Service Won't Start**

**Problem:** Service fails to start

**Solutions:**
1. **Check service status:**
   ```powershell
   Get-Service -Name "GitHub Actions Runner (*)" | Select-Object Status, StartType
   ```

2. **Set to Auto-start:**
   ```powershell
   Set-Service -Name "GitHub Actions Runner (*)" -StartupType Automatic
   ```

3. **Check logs:**
   ```powershell
   eventvwr  # Look for errors under Windows Logs > Application
   ```

4. **Reinstall service:**
   ```powershell
   # Remove old service
   cd "C:\GitHub\Actions-Runner"
   .\config.cmd remove
   
   # Reinstall
   .\config.cmd --url https://github.com/SaiVikas-Panthangi/NVD --token <NEW_TOKEN> --svc --svc-user NT AUTHORITY\SYSTEM
   ```

---

## 📋 Verification Checklist

Before declaring the runner ready, verify:

- [ ] Runner directory created: `C:\GitHub\Actions-Runner`
- [ ] Runner software downloaded and extracted
- [ ] Runner registered with GitHub (token used successfully)
- [ ] Service installed (`Get-Service` shows "GitHub Actions Runner")
- [ ] Service is running (Status = "Running")
- [ ] GitHub shows runner as 🟢 **Idle**
- [ ] Manual workflow trigger completes successfully
- [ ] Artifacts are uploaded after workflow run
- [ ] Teams notifications work (if configured)
- [ ] Runner status updates correctly (Idle → Busy → Idle)

---

## 🔄 Schedule Check

Verify the runner will start on machine reboot:

```powershell
# Set runner service to Auto-start
Set-Service -Name "GitHub Actions Runner (*)" -StartupType Automatic

# Verify
Get-Service | Where-Object {$_.Name -like "*GitHubActionsRunner*"} | Select-Object Status, StartType
# Should show: StartType = Automatic
```

**This ensures the runner restarts automatically if your machine reboots.**

---

## ✅ Success!

When you see:
- ✅ Runner status: **Idle** (green) in GitHub Settings
- ✅ Service running on your Windows machine
- ✅ Successful test workflow run
- ✅ Artifacts uploaded to GitHub

**Your self-hosted runner is ready for Dashboard Alerts workflows!** 🚀

---

## 🎓 Next Steps

1. **Verify runner is working** (follow Step 5 & 6)
2. **Configure GitHub Secrets** (see `GITHUB_ACTIONS_SETUP.md`)
3. **Wait for scheduled run** (or trigger manually to test)
4. **Monitor workflow execution** in GitHub Actions tab

---

## 📞 Reference Links

- [GitHub Actions Self-Hosted Runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
- [Adding Self-Hosted Runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners)
- [Runner Configuration](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/configuring-the-self-hosted-runner-application-as-a-service)
