---
title: "DFIR INVESTIGATION – AFTER WMIEXEC ABUSE"
author: Mercy W.
tags:
  - dfir
  - labs
  - intermediate
---

### **Introduction**
Wmiexec (specifically the Impacket Python script wmiexec.py) is a dual-use tool heavily abused by threat actors for lateral movement and remote code execution within Windows networks, often to evade detection. Its abuse stems from leveraging the legitimate Windows Management Instrumentation (WMI) framework.

<img src="{{ '/assets/wmiexec1.png' | relative_url }}" alt="wmiexec">
<img src="{{ '/assets/wmiexec2.png' | relative_url }}" alt="wmiexec">

Detections after wmiexec has been used on a system:

1. EVENT ID 4624 – Security Logs

Note the following:
- Account Name
- Network Address and Source Port
- Authentication information - NTLM, important to note wmiexec.py can also be used with a -k flag for Kerberos Auth.
- Logon Type 3
  
<img src="{{ '/assets/wmiexec3.png' | relative_url }}" alt="wmiexec">
<img src="{{ '/assets/wmiexec4.png' | relative_url }}" alt="wmiexec">


2. SYSMON – Event ID 1
Note the following: 
 - Check the command line and parent process
 - Check executed commands i.e. cmd.exe /Q /c <executed command>
  -  Switch explanation:
        - /Q: Turns echo off. When /Q is used, the command prompt will not display the commands being executed, resulting in quieter output.
        - /c: Carries out the command specified after /c, then terminates the command prompt instance.
   
<img src="{{ '/assets/wmiexec5.png' | relative_url }}" alt="wmiexec">

3. Hayabusa Rules Detect Impacket Use

Hayabusa can detect the use of Impacket tools such as wmiexec.py.

<img src="{{ '/assets/wmiexec6.png' | relative_url }}" alt="wmiexec">
<img src="{{ '/assets/wmiexec7.png' | relative_url }}" alt="wmiexec">


#### **Conclusion**
In conclusion, while wmiexec is often considered stealthy because it does not drop files or open an interactive session, it does leave detectable artifacts across several log sources. To improve detection and response:

1. Use a SIEM solution to centralize and retain all logs for longer periods. This helps correlate events such as logon activity, WMI operations, and process execution.

2. Enable Sysmon to capture detailed telemetry, especially:

   - Process creation (Event ID 1)

    - Parent and child process relationships

    - The full command line executed (cmd.exe /Q /c <command>)

3. Enable Security Audit Logging (Event ID 4688) with command-line logging turned on. This allows you to see:

    - The command line used

    - The parent process (commonly wmiprvse.exe when wmiexec is used)

4. Monitor for unusual remote logons (Event ID 4624, Logon Type 3) coming from unexpected source hosts.

5. Baseline normal WMI usage so that unusual WMI process activity stands out.

By combining these logging and monitoring practices, defenders can reliably detect wmiexec activity even though it is considered a “low-noise” technique.
