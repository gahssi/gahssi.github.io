---
layout: post
title:  "Splunk Lab Demo"
date:   2024-11-20 08:29:55 -0700
categories: jekyll update
---

# Detecting Malicious PowerShell Activity with Splunk and Sysmon

## Introduction

In the ever-evolving landscape of cybersecurity, detecting and responding to threats promptly is crucial. As part of a recent cybersecurity lab assignment, I explored how to leverage Splunk, Sysmon, and Atomic Red Team to detect malicious PowerShell activities on a Windows machine. This blog post documents the steps I took, the challenges faced, and the lessons learned in using Splunk for blue teaming efforts.

---

## Setup of Splunk and Universal Forwarder

### Installing Splunk Enterprise

To begin, I installed Splunk Enterprise on an Ubuntu virtual machine to serve as the centralized log analysis platform.

1. **Download Splunk Enterprise** from the [official website](https://www.splunk.com/en_us/download/splunk-enterprise.html).
2. **Install Splunk** using the `.deb` package:

   ```bash
   sudo dpkg -i splunk-package-name.deb
   ```

3. **Start Splunk** and accept the license agreement:

   ```bash
   sudo /opt/splunk/bin/splunk start --accept-license
   ```

4. **Create an Administrator Account** when prompted.

### Setting Up Splunk Universal Forwarder on Windows

On the Windows 10 virtual machine, I installed the Splunk Universal Forwarder to send logs to the Splunk server.

1. **Download the Universal Forwarder** from the [Splunk website](https://www.splunk.com/en_us/download/universal-forwarder.html).
2. **Install the Forwarder** using the installer, specifying the Splunk server's IP address and receiving port.
3. **Configure Forwarding** by editing the `outputs.conf` file to ensure logs are sent to the correct destination.

### Enabling Data Receiving on Splunk Server

On the Splunk server, I enabled listening on the receiving port (default is 9997).

```bash
sudo /opt/splunk/bin/splunk enable listen 9997
```

---

## Integrating Sysmon Logs with Splunk

### Installing Sysmon on Windows

Sysmon provides detailed logs of system activities, which are essential for detecting malicious behavior.

1. **Download Sysmon** from the [Microsoft Sysinternals website](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon).
2. **Install Sysmon** with a comprehensive configuration:

   ```bash
   sysmon -i sysmonconfig-export.xml
   ```

   *Note: I used the [SwiftOnSecurity Sysmon config](https://github.com/SwiftOnSecurity/sysmon-config) for extensive logging.*

### Configuring the Universal Forwarder to Collect Sysmon Logs

1. **Edit `inputs.conf`** to add the Sysmon event log:

   ```ini
   [WinEventLog://Microsoft-Windows-Sysmon/Operational]
   disabled = 0
   ```

2. **Restart the Universal Forwarder** to apply changes.

### Verifying Log Reception in Splunk

On the Splunk server:

1. **Search for Sysmon Logs** in Splunk Web:

   ```spl
   index=windows sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
   ```

2. **Confirm Log Entries** are being received and indexed.

---

## Testing PowerShell Exploitation with Atomic Red Team

### Installing Atomic Red Team

Atomic Red Team simulates adversary techniques to test detection capabilities.

1. **Install Git** on Windows if not already installed.
2. **Clone the Atomic Red Team Repository**:

   ```powershell
   git clone https://github.com/redcanaryco/atomic-red-team.git C:\AtomicRedTeam
   ```

3. **Install the `Invoke-AtomicRedTeam` Module**:

   ```powershell
   Install-Module Invoke-AtomicRedTeam -Scope CurrentUser
   ```

4. **Set the Path to Atomic Tests**:

   ```powershell
   $PSDefaultParameterValues = @{"Invoke-AtomicTest:PathToAtomicsFolder"="C:\AtomicRedTeam\atomics"}
   ```

### Executing Atomic Tests for PowerShell Exploitation

#### Test 1: PowerShell Execution with EncodedCommand (T1059.001)

```powershell
Invoke-AtomicTest T1059.001 -TestNumbers 3
```

*Simulates execution of a base64-encoded PowerShell command.*

#### Test 2: PowerShell Download Cradle (T1059.001)

```powershell
Invoke-AtomicTest T1059.001 -TestNumbers 4
```

*Simulates downloading and executing code from a remote URL.*

#### Test 3: Obfuscated PowerShell via XOR Encoding (T1027)

```powershell
Invoke-AtomicTest T1027 -TestNumbers 2
```

*Executes an obfuscated script to test detection of hidden code execution.*

#### Test 4: Reversible Encoding Using Certutil (T1027)

```powershell
Invoke-AtomicTest T1027 -TestNumbers 5
```

*Demonstrates file obfuscation using built-in Windows utilities.*

#### Test 5: Persistence via Winlogon Helper DLL (T1547.004)

```powershell
Invoke-AtomicTest T1547.004 -TestNumbers 1
```

*Adds a registry entry for persistence.*

---

## Analysis, Alerts, and Automated Responses

### Creating Splunk Alerts for Malicious Activity

#### Alert for Malicious PowerShell Activity

```spl
index=windows sourcetype=XmlWinEventLog EventCode=1 
(Image="*\\powershell.exe" OR ParentImage="*\\powershell.exe") 
(
  CommandLine="*EncodedCommand*" OR 
  CommandLine="*IEX*" OR 
  CommandLine="*-W Hidden*" OR 
  CommandLine="*-ExecutionPolicy Bypass*" OR 
  CommandLine="*New-Object Net.WebClient*" OR
  CommandLine="*Invoke-Expression*" OR
  CommandLine="*Invoke-WebRequest*" OR
  CommandLine="*DownloadString*" OR
  CommandLine="*FromBase64String*"
)
| table _time ComputerName User CommandLine ParentProcessId ProcessId
| sort - _time
```

*Detects suspicious PowerShell commands indicative of malicious activity.*

#### Alert for Certutil Misuse

```spl
index=windows sourcetype=XmlWinEventLog EventCode=1 
Image="*\\certutil.exe" 
(CommandLine="*-encode*" OR CommandLine="*-decode*" OR CommandLine="*-urlcache*")
| table _time ComputerName User CommandLine ParentProcessId ProcessId
| sort - _time
```

*Monitors for encoding and decoding operations using `certutil.exe`.*

#### Alert for Registry Modifications to Winlogon

```spl
index=windows sourcetype=XmlWinEventLog EventCode=13 
| where like(TargetObject, "%\\Microsoft\\Windows NT\\CurrentVersion\\Winlogon\\Shell")
| table _time ComputerName User TargetObject Details
| sort - _time
```

*Detects changes to the Winlogon registry keys used for persistence.*

### Adjusting Alerts for Accurate Detection

During testing, I realized that some alerts did not trigger due to differences in registry paths and Sysmon configurations. I updated the alerts and Sysmon settings accordingly.

#### Updating Sysmon Configuration for Registry Events

Added the following to the Sysmon configuration to log registry events under HKCU:

```xml
<EventFiltering>
  <RegistryEvent onmatch="include">
    <TargetObject condition="contains">\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell</TargetObject>
  </RegistryEvent>
</EventFiltering>
```

*Ensures that modifications to the `Winlogon\Shell` key are logged.*

### Implementing Automated Response Actions

To enhance the detection capabilities, I explored automating responses to detected threats.

#### Creating a Script to Kill Malicious Processes

```powershell
# kill_malicious_process.ps1
param(
    [string]$TargetComputer
)

$ScriptBlock = {
    $processes = Get-WmiObject Win32_Process -Filter "Name='powershell.exe'"

    foreach ($proc in $processes) {
        $cmdLine = $proc.CommandLine

        if ($cmdLine -match 'Invoke-WebRequest' -or $cmdLine -match 'IEX' -or $cmdLine -match 'DownloadString' -or $cmdLine -match 'EncodedCommand') {
            Write-Host "Terminating malicious process ID $($proc.ProcessId) with command line: $cmdLine"
            Stop-Process -Id $proc.ProcessId -Force
        }
    }
}

Invoke-Command -ComputerName $TargetComputer -ScriptBlock $ScriptBlock
```

*This script identifies and terminates malicious PowerShell processes.*

#### Configuring Splunk to Execute the Script on Alert

- **Added the script to Splunk's `bin/scripts` directory.**
- **Modified the alert action** to run the script, passing the target computer's name.

---

## Lessons Learned

### Importance of Accurate Event Logging

- **Sysmon Configuration is Critical:** Properly configuring Sysmon ensures that all relevant events are captured.
- **Understanding Event IDs:** Knowing which Event IDs correspond to specific activities is essential for accurate alerts.

### Challenges with Alert Tuning

- **False Positives:** Overly broad alerts can generate noise, so it's important to fine-tune search criteria.
- **Dynamic Paths in Registry Keys:** Using wildcards and regex in Splunk queries helps account for variations.

### Automated Responses

- **Balancing Security and Functionality:** Automated actions can mitigate threats quickly but must be implemented carefully to avoid unintended consequences.
- **Secure Remote Execution:** Enabling PowerShell remoting introduces security considerations that must be addressed.

---

## Conclusion

This exercise highlighted the powerful capabilities of Splunk and Sysmon in detecting and responding to malicious activities. By simulating attacks with Atomic Red Team, I was able to test and refine detection mechanisms, gaining valuable insights into threat detection and incident response processes. The hands-on experience reinforced the importance of meticulous configuration and the potential of automation in enhancing cybersecurity defenses.

---

## Screenshots

*Note: Placeholder for screenshots to be added.*

- ![Splunk Search Results for Malicious PowerShell Activity](path/to/screenshot1.png)
- ![Sysmon Event Logs in Splunk](path/to/screenshot2.png)
- ![Atomic Red Team Test Execution](path/to/screenshot3.png)
- ![Automated Response Script Execution](path/to/screenshot4.png)
- ![Splunk Alert Configuration](path/to/screenshot5.png)

---

*Thank you for reading! Feel free to share your thoughts or ask questions in the comments below.*