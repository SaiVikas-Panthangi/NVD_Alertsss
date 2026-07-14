# Self-Hosted Runner Setup - DIY Guide with Explanations

This guide explains **WHAT**, **HOW**, and **WHY** for each step you need to do.

---

## 🎯 What We're Doing & Why

### **The Problem:**
- GitHub Actions cloud runners (`ubuntu-latest`, `windows-latest`) **CANNOT access** your internal APIs
- Dashboard Alerts Monitor needs to access internal APIs on `inutilidcplp322.idc1.level3.com:8483`
- Cloud runners can't reach your internal network

### **The Solution:**
- Install a **self-hosted runner** on your Windows machine
- Your machine IS on the internal network, so it CAN access internal APIs
- GitHub Actions sends jobs to your machine instead of cloud servers
- Your machine executes the workflow and reports back to GitHub

### **Why This Architecture:**
```
┌─────────────────────────────────────┐
│    GitHub.com (Cloud)               │
│  - Stores code & workflows           │
│  - Manages secrets securely          │
│  - Displays results & reports        │
└────────┬────────────────────────────┘
         │
         │ Sends workflow job to your runner
         │
         ▼
┌─────────────────────────────────────┐
│    YOUR WINDOWS MACHINE              │
│  - GitHub Actions Runner running     │
│  - Can access internal APIs          │
│  - Executes the dashboard alerts     │
│  - Reports results back to GitHub    │
└─────────────────────────────────────┘
         │
         │ Can reach internal network
         ▼
┌─────────────────────────────────────┐
│    Internal APIs                     │
│  - inutilidcplp322.idc1.level3.com   │
│  - Port 8483                         │
│  - Only accessible from internal net │
└─────────────────────────────────────┘
```

---

## 📋 Step 1: Create a Runner Directory

### **WHAT we're doing:**
Creating a folder where the runner software will live

### **WHY:**
- The runner needs a dedicated folder to store configuration, logs, and working files
- Keeps everything organized and separate from other programs
- Default location: `C:\GitHub\Actions-Runner` (you can change, but use same path throughout)

### **HOW TO DO IT:**

**Action 1: Open PowerShell as Administrator**
1. Click **Windows Start** button
2. Type: `PowerShell`
3. Right-click **Windows PowerShell**
4. Click **"Run as administrator"**
5. Click **"Yes"** when prompted

**Action 2: Create the folder**

Copy-paste this command into PowerShell:
```powershell
mkdir "C:\GitHub\Actions-Runner"
```

**Action 3: Enter the folder**
```powershell
cd "C:\GitHub\Actions-Runner"
```

**You should see:**
```
PS C:\GitHub\Actions-Runner>
```

✅ **Success**: You're now inside the runner directory

---

## 📥 Step 2: Get the Runner Download Link from GitHub

### **WHAT we're doing:**
Getting the unique download link + token for your specific GitHub repository

### **WHY:**
- GitHub generates a temporary token that proves YOU have permission to register a runner
- The token expires after ~1 hour for security
- We need to download the runner software that matches your Windows architecture

### **HOW TO DO IT:**

**Action 1: Go to GitHub Settings**
1. Open browser and go to: **https://github.com/SaiVikas-Panthangi/NVD**
2. Click **Settings** (top-right navigation)
3. In left sidebar, click **Actions** > **Runners**

**You should see:**
```
"There are no runners configured"
[New self-hosted runner] button
```

**Action 2: Click "New self-hosted runner"**
- Green button on the right side

**Action 3: Select Configuration**
1. **Runner OS:** Select **Windows** (click the button)
2. **Architecture:** Select **x64** (click the button)

**You'll see download instructions appear:**
```
Download
Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.xxx.x/actions-runner-win-x64-2.xxx.x.zip -OutFile actions-runner-win-x64-2.xxx.x.zip

Configure
.\config.cmd --url https://github.com/SaiVikas-Panthangi/NVD --token ABC123XYZ...
```

**Action 4: COPY the two commands**
- Copy the **Invoke-WebRequest** command (the long download one)
- Copy the **config.cmd** command (the registration one)
- Save them somewhere temporary (Notepad is fine)

✅ **Success**: You have the download link and your unique token

---

## 📦 Step 3: Download the Runner Software

### **WHAT we're doing:**
Downloading ~100MB of runner software from GitHub's servers

### **WHY:**
- This is the GitHub Actions Runner application that will listen for jobs
- We're downloading the Windows x64 version that matches your machine
- The zip file contains all the necessary files to run workflows

### **HOW TO DO IT:**

**Action 1: Paste the download command into PowerShell**

You should still be in:
```
PS C:\GitHub\Actions-Runner>
```

Paste the **Invoke-WebRequest** command you copied from GitHub:
```powershell
Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.316.0/actions-runner-win-x64-2.316.0.zip -OutFile actions-runner-win-x64-2.316.0.zip
```

**Action 2: Wait for download**
- You'll see progress: `Downloaded X MB of Y MB`
- Should take 1-5 minutes depending on internet speed
- **DO NOT CLOSE PowerShell**

**You'll know it's done when:**
```
PS C:\GitHub\Actions-Runner>
```
(The prompt returns)

✅ **Success**: Download complete

---

## 🗜️ Step 4: Extract (Unzip) the Runner

### **WHAT we're doing:**
Extracting the zip file into individual files

### **WHY:**
- The runner came as a compressed zip file
- We need to extract it so we can run the configuration script
- Windows uses .NET to extract - that's what this command does

### **HOW TO DO IT:**

**Action 1: Run extraction command**

Paste this into PowerShell:
```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::ExtractToDirectory("$PWD\actions-runner-win-x64-2.316.0.zip", "$PWD")
```

**What this does:**
- `Add-Type -AssemblyName System.IO.Compression.FileSystem` = Tell PowerShell how to extract zip files
- `[System.IO.Compression.ZipFile]::ExtractToDirectory` = Extract the zip
- `"$PWD\actions-runner-win-x64-2.316.0.zip"` = From this file
- `"$PWD"` = Into this folder

**Action 2: Verify extraction**

Type this command:
```powershell
dir
```

**You should see files like:**
```
config.cmd
config.sh
run.cmd
run.ps1
bin/
lib/
...
```

✅ **Success**: Files are extracted

---

## 🔐 Step 5: Register the Runner with GitHub

### **WHAT we're doing:**
Telling GitHub that this machine is now a valid runner for your repository

### **WHY:**
- GitHub needs to verify that YOU authorized this machine to run workflows
- The token proves you have permission
- This registration stores the configuration locally on your machine
- After this, GitHub will know to send jobs to this machine

### **HOW TO DO IT:**

**Action 1: Run the configuration command**

Paste the **config.cmd** command you copied from GitHub earlier:
```powershell
.\config.cmd --url https://github.com/SaiVikas-Panthangi/NVD --token ABC123XYZ...
```

⚠️ **Use your actual token from GitHub** (not ABC123XYZ)

**Action 2: Answer the questions**

The runner will ask:

```
Enter the name of the runner group to add this runner to? (default)
```
Answer: Just press **Enter** (use default)

```
Enter the name of the runner to configure as? (DESKTOP-ABC123)
```
Answer: Press **Enter** (or type a name like `dashboard-alerts-runner`)

```
Enter the work directory? (_work)
```
Answer: Press **Enter** (use default `_work`)

```
Enter additional labels (optional):
```
Answer: Press **Enter** (you can add labels later if needed)

```
Enter name of work folder? (_work)
```
Answer: Press **Enter**

**Action 3: Look for success message**

You should see:
```
√ Connected to GitHub
√ Settings Saved.
```

✅ **Success**: Runner is registered with GitHub

---

## 🚀 Step 6: Install Runner as Windows Service

### **WHAT we're doing:**
Installing the runner as a Windows Service so it starts automatically

### **WHY:**
- Without this, the runner only runs while PowerShell is open
- As a Windows Service, it runs in the background 24/7
- If your machine reboots, the service automatically restarts
- This is critical for scheduled workflows to run reliably

### **HOW TO DO IT:**

**Action 1: Run the service installation command**

Paste this command:
```powershell
.\config.cmd --url https://github.com/SaiVikas-Panthangi/NVD --token ABC123XYZ... --svc --svc-user NT AUTHORITY\SYSTEM
```

⚠️ **Replace your actual token** (same token as Step 5)

**What this does:**
- `--svc` = Install as Windows Service
- `--svc-user NT AUTHORITY\SYSTEM` = Run with System privileges (needed for internal API access)

**Action 2: Look for success**

You should see:
```
Service installed successfully
```

**Action 3: Verify the service exists**

Type this:
```powershell
Get-Service | Where-Object {$_.Name -like "*GitHub*"}
```

**You should see:**
```
Status   Name                      DisplayName
------   ----                      -----------
Running  GitHubActionsRunner...    GitHub Actions Runner
```

✅ **Success**: Service is installed and running

---

## ✅ Step 7: Verify Runner in GitHub

### **WHAT we're doing:**
Checking that GitHub can see your runner

### **WHY:**
- Confirms the registration worked
- Verifies your runner is "online" and ready
- If status is green/idle, everything is working

### **HOW TO DO IT:**

**Action 1: Go back to GitHub Settings**
1. Open browser: **https://github.com/SaiVikas-Panthangi/NVD**
2. Click **Settings > Actions > Runners**

**Action 2: Look for your runner**

You should see:
```
🟢 DESKTOP-ABC123 (or your custom name)
Status: idle
Last Activity: Just now
```

🟢 **Green dot = GOOD!** (means it's online and ready)

If you see:
- 🔴 Red dot = Offline (restart the service)
- ⏳ Yellow dot = Busy (a workflow is running)

✅ **Success**: Runner is online and idle

---

## 🧪 Step 8: Test the Runner

### **WHAT we're doing:**
Running a test workflow to make sure everything works

### **WHY:**
- Proves the runner is actually connected and can execute jobs
- Verifies artifacts are uploaded correctly
- Confirms notifications work (if configured)
- Early detection of any configuration issues

### **HOW TO DO IT:**

**Action 1: Go to GitHub Actions**
1. Go to: **https://github.com/SaiVikas-Panthangi/NVD/actions**

**Action 2: Click the workflow**
- Click **"Dashboard Alerts Hourly Monitor"**

**Action 3: Click "Run workflow"**
- Top right, find the **"Run workflow"** button (gray/white)
- Click the dropdown arrow next to it
- Select **Branch: main**
- Click **"Run workflow"** button

**Action 4: Watch it run**
- The page should show a new run appearing
- Status will be 🟡 **In progress** (yellow)
- Your runner status in Settings should change to **Busy**

**Action 5: Wait for completion**
- Usually takes 1-2 minutes
- Status will change to ✅ **Passed** (green) or ❌ **Failed** (red)

**Action 6: Check results**
1. Click the completed workflow run
2. Click the **"run-monitor"** job
3. Expand steps to see logs
4. Scroll to bottom of **"Run monitor"** step - look for success message

✅ **Success**: Workflow completed successfully!

---

## 📊 Summary: What We Did & Why

| Step | What | Why |
|------|------|-----|
| 1 | Created runner directory | Organized place to store runner files |
| 2 | Got GitHub token | Proves we have permission to register runner |
| 3 | Downloaded runner software | The actual runner application |
| 4 | Extracted zip file | Unpacked the runner files |
| 5 | Registered with GitHub | Told GitHub this machine is a valid runner |
| 6 | Installed as Windows Service | Runs 24/7 in background, auto-restarts |
| 7 | Verified in GitHub | Confirmed runner is online and ready |
| 8 | Tested with workflow | Proved everything works correctly |

---

## 🎯 You Are Now Done With Runner Setup!

✅ Self-hosted runner is installed and running
✅ GitHub can communicate with your machine
✅ Workflows can execute on your machine
✅ Internal APIs are now accessible from workflows

---

## 🔄 What Happens Next

1. **Every hour** (09:00-23:00 IST), GitHub will automatically send the Dashboard Alerts workflow to your runner
2. Your runner executes the `npm run monitor:run` command
3. The monitor checks internal APIs
4. Results are sent to GitHub and Teams webhook
5. Artifacts (reports, logs) are uploaded to GitHub

---

## ⚠️ Important Notes

### **If You Need to Stop/Restart the Runner**

Stop it:
```powershell
Stop-Service -Name "GitHub Actions Runner (*)"
```

Start it:
```powershell
Start-Service -Name "GitHub Actions Runner (*)"
```

Restart it:
```powershell
Restart-Service -Name "GitHub Actions Runner (*)"
```

### **Auto-Start on Machine Reboot**

Make sure it auto-starts:
```powershell
Set-Service -Name "GitHub Actions Runner (*)" -StartupType Automatic
```

### **If Something Goes Wrong**

1. Check runner status: `Get-Service | Where-Object {$_.Name -like "*GitHub*"}`
2. Check GitHub Settings > Actions > Runners > Status
3. See detailed troubleshooting in: `SELF_HOSTED_RUNNER_SETUP.md`

---

## ✅ Verification Checklist

Before you declare success:

- [ ] PowerShell ran without errors
- [ ] Files extracted successfully (`dir` showed config.cmd, run.cmd, etc.)
- [ ] GitHub showed success messages during registration
- [ ] Service is installed and running
- [ ] GitHub Settings shows runner as 🟢 Idle
- [ ] Manual workflow test completed successfully
- [ ] Service is set to auto-start

---

**You've completed the self-hosted runner setup! 🚀**
