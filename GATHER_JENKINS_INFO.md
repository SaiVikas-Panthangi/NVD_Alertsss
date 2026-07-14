# Gather Jenkins Server Information for GitHub Actions Setup

You have Jenkins login access! This guide will show you exactly what to look for to get the server details.

---

## 🎯 What Information We Need

I need to know:
1. **Jenkins Server Address** (URL, hostname, IP)
2. **Operating System** (Linux or Windows)
3. **Server Location** (On-premises or Cloud)
4. **How to Access** (SSH for Linux, RDP for Windows)

---

## 📋 Step 1: Find Jenkins Server Address

### **Action 1: Login to Jenkins**
1. Go to your Jenkins URL in browser
2. Enter your credentials
3. You should see Jenkins dashboard

### **Action 2: Find "About Jenkins"**

**Method A (Recommended):**
1. Click **Jenkins icon** (top-left) to go to main dashboard
2. Look for **"Manage Jenkins"** (left sidebar)
3. Click it
4. Look for **"System Information"** or **"About Jenkins"** link

**Method B (Alternative):**
1. Hover over **Jenkins icon** (top-left)
2. Click **"Jenkins"** when dropdown appears
3. Scroll down to find system info

### **Action 3: Write Down**

Look for:
```
Jenkins Version: [write this down]
System Properties section shows:
  - java.version: [write this down]
  - os.name: [write this down] ← THIS IS IMPORTANT
  - os.arch: [write this down]
  - java.io.tmpdir: [write this down]
```

**What you're looking for:**
- If `os.name` says **"Linux"** → Server is Linux
- If `os.name` says **"Windows"** → Server is Windows

### **Action 4: Get Server Hostname/IP**

In same "System Information" page, look for:
```
System Properties:
  - hostname: [e.g., jenkins-prod-server]
  - java.net.preferIPv4Stack: [shows network info]
```

⚠️ If not visible, go to terminal/command prompt on Jenkins machine (see Step 2).

---

## 🌐 Step 2: Find Jenkins Server Address from URL

**Easiest method:**

Look at your browser URL bar:
```
https://jenkins.yourcompany.com:8080/
        ^^^^^^^^^^^^^^^^^^^^^^^^^^
        This is your Jenkins server hostname/address
```

Or it might be:
```
https://192.168.1.100:8080/    ← IP address
https://jenkins-prod:8080/     ← Hostname
```

**Write down:** What appears between `https://` and `:8080` (or whatever port)

---

## 💻 Step 3: Determine Operating System

### **Method A: From Jenkins UI**

From System Information page, look for:
```
os.name = Windows 10           ✓ Windows Server
os.name = Linux               ✓ Linux Server  
os.name = Mac OS X            ✓ Mac Server
```

### **Method B: Check Jenkins Configuration**

1. Go to **Manage Jenkins > Configure System**
2. Look for shell configuration or script paths
3. **If you see batch files** (`.bat`) → Windows
4. **If you see shell scripts** (`.sh`) → Linux

### **Method C: Look at Jenkinsfile in Your Repo**

Open the Jenkinsfile in Dashboard-Alerts:
```groovy
stages {
  stage('Install Dependencies') {
    steps {
      script {
        if (isUnix()) {           ← Detects if Linux
          sh 'npm ci'             ← Linux command
        } else {
          bat 'npm ci'            ← Windows command
        }
      }
    }
  }
}
```

This means Jenkins supports **both Linux and Windows**. Check which one is actually running the jobs.

---

## 🔍 Step 4: Check Current Jenkins Jobs/Agents

### **To See Where Jobs Run:**

1. **Dashboard Alerts Jenkins Job** (if it exists)
   - Find the job in Jenkins UI
   - Click **"Configure"**
   - Look for **"Restrict where this project can be run"**
   - It will show: agent name, or `any`, or specific label
   - This tells you what server the job runs on

2. **Check Agent/Node Info**
   - Go to **Manage Jenkins > Nodes and Cloud**
   - You'll see list of agents/nodes
   - Click each one to see:
     - **Name**
     - **IP/Hostname**
     - **OS** (Windows or Linux)
     - **Labels** (e.g., "linux", "windows", "api-monitor")
     - **Status** (Online/Offline)

---

## 📊 Step 5: Collect All Information

Create a checklist and fill it out:

```
JENKINS SERVER INFORMATION
==========================================

Jenkins URL: 
  https://____________ or http://____________

Jenkins Version:
  ________

Server Hostname/IP Address:
  ________

Operating System:
  ☐ Linux (Ubuntu / CentOS / RHEL)
  ☐ Windows Server
  ☐ Docker Container
  ☐ Cloud VM (AWS / Azure / Google Cloud)
  ☐ Other: ________

OS Details (from System Information):
  - os.name: ________
  - os.arch: ________
  - java.version: ________

Can you SSH to it? (Linux)
  ☐ Yes, using: ________ (e.g., ssh ubuntu@192.168.1.100)
  ☐ No, explain: ________

Can you RDP to it? (Windows)
  ☐ Yes, using: ________ (e.g., 192.168.1.100)
  ☐ No, explain: ________

Jenkins Jobs that run Dashboard Alerts:
  - Job Name: ________
  - Runs on Node/Agent: ________
  - Status: ________

Current Scheduled Jobs:
  - Dashboard Alerts Monitor runs at: ________
  - Jenkinsfile cron schedule: ________

Access Method:
  ☐ Direct SSH/RDP access available
  ☐ Through VPN
  ☐ Through Bastion Host/Jump Server
  ☐ Cloud console (AWS/Azure)
  ☐ Other: ________
```

---

## 🖼️ What Pages to Look At

### **Jenkins Dashboard - Key Areas:**

```
JENKINS HOME
├── Manage Jenkins (Click here)
│   ├── System Information    ← Shows OS details ✓
│   ├── Configure System      ← Shows agent configs ✓
│   ├── Nodes and Clouds      ← Shows where jobs run ✓
│   └── Logs                  ← Shows server details
│
├── New Job / Existing Jobs
│   └── Dashboard Alerts Monitor (if exists)
│       └── Configure         ← Shows which agent it runs on ✓
│
└── Build History
    └── Recent builds         ← Shows execution details ✓
```

---

## 🎯 Once You Have This Info

Share the checklist with me, and I'll tell you:

1. **Exactly what server setup is needed**
   - Linux → Provide Linux installation steps
   - Windows → Provide Windows installation steps
   - Docker → Provide Docker containerization steps
   - Cloud → Provide cloud-specific setup

2. **How to SSH/RDP to the server**

3. **How to install GitHub Actions runner on THAT server**

4. **How to configure it with your GitHub repo**

---

## 🚀 Quick Summary

**Just login to Jenkins and:**

1. ✅ Note down the Jenkins URL (from address bar)
2. ✅ Go to **Manage Jenkins > System Information**
3. ✅ Write down `os.name` (Windows or Linux)
4. ✅ Check **Nodes and Clouds** to see where jobs run
5. ✅ Find hostname/IP of that server
6. ✅ Share this info with me

**That's all I need!** Then I can give you exact server setup instructions. 💼

---

## 📌 Example Responses

**Example 1 - Linux Server:**
```
Jenkins URL: https://jenkins.lumen.com:8080
OS: Linux (Ubuntu 20.04)
Hostname: jenkins-prod-server
IP: 192.168.1.50
Access: SSH to ubuntu@jenkins-prod-server
```
→ I'll give you **Linux installation steps**

**Example 2 - Windows Server:**
```
Jenkins URL: https://jenkins-windows.lumen.com:8080
OS: Windows Server 2019
Hostname: JENKINS-PROD-01
IP: 192.168.1.100
Access: RDP to JENKINS-PROD-01
```
→ I'll give you **Windows Server installation steps**

**Example 3 - Cloud (AWS):**
```
Jenkins URL: https://jenkins-prod.aws.lumen.com
OS: Linux (Amazon Linux 2)
Instance ID: i-0abc123def456
Region: us-east-1
Access: SSH key pair + AWS console
```
→ I'll give you **AWS EC2 installation steps**

---

## ✅ Next Action

1. **Login to Jenkins now**
2. **Gather the information using this guide**
3. **Reply with the checklist filled out**
4. **I'll give you server-specific setup instructions**

🎯 Let's do this!
