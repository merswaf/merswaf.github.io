---
title: "Building a Beginner DFIR Lab: Integrating Sysmon & Splunk for Visibility"
author: Mers Waf
---

### **Introduction**

Welcome to my guide on setting up a core Digital Forensics and Incident Response (DFIR) practice lab! If you're new to security and want to understand how to detect malicious activity on a network, this is the perfect starting point. The goal is to create a controlled environment where we can monitor everything that happens.

![](assets/beginnersplunk.png)

My lab consists of five machines on a single Active Directory domain:

- **1x Active Directory Domain Controller (DC)** (Windows Server 2022)
- **2x General Servers** (Windows Server 2022) - One is a general-purpose server, and one is a dedicated file share.
- **2x Windows 11 Endpoints**

The key to visibility? We'll install **Sysmon** on every machine and forward all those logs to a **Splunk** server for analysis. Let's get started.

---

### **Part 1: Gaining Deep Visibility with Sysmon**

**What is Sysmon?**

System Monitor (Sysmon) is a powerful Windows system service and device driver from Microsoft. Once installed, it runs constantly, even after reboots, logging detailed system activity to the Windows event log. It provides invaluable information on:

- Process creations (with full command lines and hashes)
- Network connections
- Changes to file creation times
- And much more...

By analyzing these logs, you can identify malicious or suspicious activity that would otherwise be invisible. A key feature is that it runs as a **protected process**, making it very difficult for attackers or malware to tamper with.

**Installing and Configuring Sysmon**

For the best results, we won't use the default configuration. Instead, we'll use a renowned, community-tested configuration file from **SwiftOnSecurity** that provides high-quality event tracing.

1. **Download the Configuration File:** Grab the latest `sysmonconfig-export.xml` from the [SwiftOnSecurity sysmon-config GitHub repository](https://github.com/SwiftOnSecurity/sysmon-config/tree/master).
2. **Run the Installation:** Open an elevated Command Prompt (Run as Administrator) on each of your five lab machines and run
    
    
    ```
    sysmon.exe -accepteula -i sysmonconfig-export.xml -l -n
    ```
    

**Pro Tip:** If you need to update your configuration later, use: `sysmon -c sysmonconfig-export.xml`. To uninstall, use `sysmon -u`.

---

### **Part 2: Centralizing Logs with Splunk**

Now that we have great logs on each host, we need a central place to collect and analyze them. That's where Splunk comes in.

**Step 1: Set Up the Splunk Universal Forwarder**

The Universal Forwarder is a lightweight agent that sends data to your main Splunk server.

1. Install the Splunk Universal Forwarder on each of your five Windows machines.
2. During installation, set the **receiver** as your main Splunk server's IP address and use the default port **9997**.

**Step 2: Configure Splunk to Receive Data**

On your main Splunk server, you need to open a port to listen for this incoming data.

- Go to **Settings > Data > Forwarding and Receiving**.
- Under **Receive Data**, click **Add New**.
- Add port **9997** and save.

**Step 3: Verify Communication**

To check if your hosts are successfully sending data, run this simple search on your Splunk server:

```
index=_internal | stats count by host
```

You should see your five lab machines listed.

---

### **Part 3: Teaching Splunk to Understand Sysmon**

Raw logs aren't very useful. We need to parse them into structured fields. Install these essential free add-ons from Splunkbase onto your **Splunk Search Head**:

1. [Splunk Add-on for Microsoft Sysmon](https://splunkbase.splunk.com/app/1914)
2. [Splunk Add-on for Microsoft Windows](https://splunkbase.splunk.com/app/742)

**To install an add-on:**

1. Log in to Splunk Web.
2. Go to **Apps > Manage Apps**.
3. Click **Install app from file** and upload the downloaded `.tgz` files.
4. Restart Splunk when prompted.

---

### **Part 4: Configuring the Forwarders to Collect the Right Data**

**Critical Step: Service Permissions**

The Sysmon event log is privileged. You **must** configure the `SplunkForwarder` service on each endpoint to run as **Local System** for it to access the logs. You can change this in `Services.msc`.

**Create Indexes on Splunk Server:**

On your main Splunk server, create two new indexes to organize the data:

- `wineventlog` - For standard Windows logs
- `sysmon` - Specifically for Sysmon data

**Create the inputs.conf File:**

This file tells the Universal Forwarder *what* logs to collect and *where* to send them. On each endpoint, create a file called `inputs.conf` and place it in:

`C:\Program Files\SplunkUniversalForwarder\etc\system\local\`

Paste the following configuration into the file:

```
# ================================
# Standard Windows Event Logs
# ================================
[WinEventLog://Application]disabled = 0
index = wineventlog

[WinEventLog://System]disabled = 0
index = wineventlog

[WinEventLog://Security]disabled = 0
index = wineventlog

# ================================
# Sysmon Event Logs
# ================================
[WinEventLog://Microsoft-Windows-Sysmon/Operational]disabled = 0
renderXml = true # Crucial for parsing Sysmon's XML data
index = sysmon

# ================================
# Optional: PowerShell Logs
# ================================
[WinEventLog://Microsoft-Windows-PowerShell/Operational]disabled = 0
index = wineventlog

[WinEventLog://Windows PowerShell]disabled = 0
index = wineventlog
```

**Restart the Splunk Universal Forwarder service** on each machine to apply the changes.

**(Optional) outputs.conf:** If you need to change the IP address of the Splunk server after installation, create an `outputs.conf` file in the same folder with this content:

---

### **Conclusion: You're Ready to Hunt!**

You now have a fully instrumented lab environment! Sysmon is diligently recording activity on every machine, and Splunk is centrally collecting and parsing those logs.

To test your setup, try running this search in Splunk: The check sources to ensure all endpoints are covered

```
index=sysmon
```
![](assets/splunksearch.png)
 
Your journey into DFIR has begun. The next step is to simulate adversary activity in your lab and practice hunting for it using the powerful data you're now collecting. Happy hunting!

**P.S.** For troubleshooting, remember the Universal Forwarder logs are located in: `C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log`.
