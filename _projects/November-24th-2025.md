---
title: "DFIR INVESTIGATION – AFTER   SMBEXEC ABUSE"
author: Mercy W.
tags:
  - dfir
  - labs
  - intermediate
---

### **Introduction**


SMBExec is a post-exploitation tool commonly used by attackers and red-team operators to execute commands remotely on a Windows system over SMB.
It works by authenticating to the target using valid credentials, then creating and running a temporary Windows service to execute commands. 
Because it abuses built-in Windows service mechanisms, SMBExec leaves behind a predictable set of forensic artifacts in authentication logs, service control logs, and Sysmon events. Understanding these artifacts makes it easier to detect or investigate SMBExec activity.

<img src="{{ '/assets/smbexec1.png' | relative_url }}" alt="smbexec">

### Detections

All detections must be interpreted in context — a single login event by itself does not confirm malicious activity.


1. Successful Authentication → Event ID 4624

  - Logon Type 3 (Network logon)
  - Source IP
    
<img src="{{ '/assets/smbexec2.png' | relative_url }}" alt="smbexec">
  
After the network logon, SMBExec immediately creates a temporary service on the target host.

2. Service Creation Events

  - Event ID 7045 → Indicates a new service was installed.

<img src="{{ '/assets/smbexec3.png' | relative_url }}" alt="smbexec">

  - SMBExec repeatedly creates services to execute commands, and these executions will also appear as Event ID 7045 entries.
  - 
<img src="{{ '/assets/smbexec4.png' | relative_url }}" alt="smbexec">

3. Sysmon Logs
Sysmon provides more granular visibility into SMBExec activity:

- Event ID 13 — Registry Value Set

    Logs the creation or modification of the service registry key.

  <img src="{{ '/assets/smbexec5.png' | relative_url }}" alt="smbexec">

- Event ID 11 — File Create

  Captures the creation (and quick deletion) of temporary service-related files used by SMBExec.

  <img src="{{ '/assets/smbexec6.png' | relative_url }}" alt="smbexec">

- Event ID 1 — Process Creation

  Shows cmd.exe being spawned with services.exe as the parent process — a strong indicator of service-based command execution.

  <img src="{{ '/assets/smbexec7.png' | relative_url }}" alt="smbexec">


Conclusion

SMBExec leaves a consistent and recognizable footprint across Windows logs. By correlating:

* Event ID 4624 + Logon Type 3
* Event ID 7045 (Service creation & execution)
* Sysmon Events 13, 11, and 1 (Registry, file, and process activity)

You can confidently detect or investigate remote command execution performed via SMBExec.
The strength of detecting SMBExec lies in contextual correlation—linking authentication events with suspicious service creation and subsequent command execution. When these artifacts appear together, they form a clear behavioral signature of SMBExec use, whether by attackers or red-team operators.
