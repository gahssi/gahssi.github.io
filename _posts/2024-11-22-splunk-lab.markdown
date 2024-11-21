---
layout: post
title:  "Splunk Lab Demo"
date:   2024-11-20 08:29:55 -0700
categories: jekyll update
---

# Detecting Malicious PowerShell Activity with Splunk and Sysmon

## Introduction

As part of a recent cybersecurity lab assignment, I explored how to leverage Splunk, Sysmon, and Atomic Red Team to detect malicious PowerShell activities on a Windows machine. This blog post documents the steps I took and some of the lessons learned in using Splunk for blue teaming efforts.

---

## Setup of Splunk and Universal Forwarder

### Installing Splunk Enterprise

To begin, I installed Splunk Enterprise on an Ubuntu Desktop VM to serve as the centralized log analysis and management system. When you first install Splunk it will automatically install an Enterprise Trial license enabled by default. The Splunk Enterprise Trial license is good for 60 days, and after that it will downgrade to the Splunk Free license, which has limited features.

1. Download Splunk Enterprise from the [official website](https://www.splunk.com/en_us/download/splunk-enterprise.html).
2. Install Splunk using the `.deb` package:

   ```bash
   sudo dpkg -i splunk-package-name.deb
   ```

3. Start Splunk and accept the license agreement:

   ```bash
   sudo /opt/splunk/bin/splunk start --accept-license --answer-yes
   ```

4. Create an Administrator Account when prompted.

### Setting Up Splunk Universal Forwarder on Windows

On a Windows 10 VM, I installed the Splunk Universal Forwarder to send logs to the Splunk server.

1. Download the Universal Forwarder from the [Splunk website](https://www.splunk.com/en_us/download/universal-forwarder.html).
2. Install the Forwarder using the installer, specifying the Splunk server's IP address and receiving port.
3. Configure Forwarding by creating/editing the `inputs.conf` file to ensure logs are sent to the correct destination.

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
2. **Install Sysmon** with an augmented configuration:

   ```bash
   sysmon -i sysmonconfig-export.xml
   ```

   *Note: I used [OlafHartong's default Sysmon config](https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml).*

### Configuring the Universal Forwarder to Collect Sysmon Logs

1. Edit `inputs.conf` to add the Sysmon event log:

   ```ini
    [WinEventLog://Microsoft-Windows-Sysmon/Operational]
    index = windows
    sourcetype = XmlWinEventLog
    renderXml = 0
    disabled = 0
    current_only = 0
    start_from = oldest
    checkpointInterval = 5
   ```

2. Restart the Universal Forwarder to apply changes.

### Verifying Log Reception in Splunk

On the Splunk server:

1. Search for Sysmon logs in Splunk Web:

   ```sql
   index=windows sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
   ```

2. Confirm log entries are being received and indexed.

---

## Testing PowerShell Exploitation with Atomic Red Team

### Installing Atomic Red Team

Atomic Red Team simulates adversary techniques to test detection capabilities.

1. Install Git on Windows if not already installed.
2. Clone the Atomic Red Team Repository:

   ```powershell
   git clone https://github.com/redcanaryco/atomic-red-team.git C:\AtomicRedTeam
   ```

3. Install the `Invoke-AtomicRedTeam` Module:

   ```powershell
   Install-Module Invoke-AtomicRedTeam -Scope CurrentUser
   ```

4. Set the Path to Atomic Tests:

   ```powershell
   $PSDefaultParameterValues = @{"Invoke-AtomicTest:PathToAtomicsFolder"="C:\AtomicRedTeam\atomics"}
   ```

### Executing Atomic Tests for PowerShell Exploitation

#### Test 1: Download and Execute Mimikatz

```powershell
Invoke-AtomicTest T1059.001 -TestNumbers 1
```

#### Test 2: Execute base64-encoded PowerShell from Windows Registry

```powershell
Invoke-AtomicTest T1027 -TestNumbers 3
```

#### Test 3: Create Winlogon Helper DLL Registry Entry

```powershell
Invoke-AtomicTest T1547.004 -TestNumbers 1
```

---

## Analysis, Alerts, and Automated Responses

#### Alert for Malicious PowerShell Activity

```sql
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

#### Alert for Registry Modifications to Winlogon

```sql
index=windows sourcetype=XmlWinEventLog EventCode=13 
| where like(TargetObject, "%\\Microsoft\\Windows NT\\CurrentVersion\\Winlogon\\Shell")
| table _time ComputerName User TargetObject Details
| sort - _time
```

#### Creating a Script to Kill Malicious Processes

To enhance the detection capabilities, I explored automating responses to detected threats.

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

To configure Splunk to execute the script on alert, I modified the alert action to run the script, passing the script's name.

---

## Lessons Learned

Working on this lab was a valuable experience in practicing blue teaming and threat detection. Setting up Splunk and integrating it with Sysmon allowed me to see how powerful these tools can be in monitoring and analyzing system activities. Using Atomic Red Team to simulate real-world attacks made the learning process engaging as I could directly observe how malicious activities manifest in logs and how they can be detected.

One of the biggest takeaways was the importance of precise configuration and tuning of queries. Splunk's Search Processing Language is quite deep, and there are many opportunities for optimizing your queries to minimize the number of trips to the indexers and wasteful calculations. Crafting effective alerts also requires a good grasp of your underlying system behaviors and threat model.

---

## Screenshots

*Note: Placeholder for screenshots to be added.*

- ![Splunk Search Results for Malicious PowerShell Activity](path/to/screenshot1.png)
- ![Sysmon Event Logs in Splunk](path/to/screenshot2.png)
- ![Atomic Red Team Test Execution](path/to/screenshot3.png)
- ![Automated Response Script Execution](path/to/screenshot4.png)
- ![Splunk Alert Configuration](path/to/screenshot5.png)
