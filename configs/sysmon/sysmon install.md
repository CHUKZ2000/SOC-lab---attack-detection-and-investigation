# Install and Configure Sysmon on Windows

## Overview
Sysmon (System Monitor) is a Windows system service from Microsoft Sysinternals. It provides detailed logging of system activity, including:

- Process creation
- Network connections
- File creation time changes
- Registry modifications
- Driver loading
- DNS queries
- Image loading

Sysmon logs are extremely useful for threat hunting, malware analysis, digital forensics, and SIEM platforms such as Splunk. Sysmon runs as a Windows service and writes events to:

`Applications and Services Logs > Microsoft > Windows > Sysmon > Operational`

---

## Prerequisites
Before installing Sysmon, make sure you have:

- Windows 10/11 or Windows Server
- Administrator privileges
- Internet connection
- PowerShell or Command Prompt

---

## Step 1: Create a Working Folder
Create a folder to store the Sysmon files.

Example:

```text
C:\Tools\Sysmon
```

Folder layout:

```text
C:\Tools\
└── Sysmon\
```

---

## Step 2: Download Sysmon
Download Sysmon from Microsoft's official Sysinternals page.

- Official download: Microsoft Sysinternals Sysmon Download

Extract the downloaded ZIP file. After extraction, you should have:

- `Sysmon.exe`
- `Sysmon64.exe`
- `Eula.txt`

---

## Step 3: Download a Sysmon Configuration File
Sysmon does very little with its default configuration. Use a community-maintained config file for better detection.

One common option is the SwiftOnSecurity Sysmon configuration.

Download and save the configuration file as:

```text
C:\Tools\Sysmon\sysmonconfig-export.xml
```

After this step, your folder should contain:

```text
C:\Tools\Sysmon\
  Sysmon64.exe
  Sysmon.exe
  sysmonconfig-export.xml
```

---

## Step 4: Install Sysmon
Run the following command from the Sysmon folder:

```powershell
Sysmon64.exe -accepteula -i sysmonconfig-export.xml
```

Options:

| Option | Meaning |
| --- | --- |
| `-accepteula` | Automatically accept the Microsoft EULA |
| `-i` | Install Sysmon |
| `sysmonconfig-export.xml` | Load this configuration file |

If successful, you should see confirmation that the Sysmon service and driver were installed and started.

---

## Step 5: Verify Installation
Verify the Sysmon service is running.

**Command prompt:**

```powershell
sc query Sysmon
```

Expected output:

- `STATE : RUNNING`

**PowerShell:**

```powershell
Get-Service Sysmon
```

Expected output:

- `Status : Running`

---

## Step 6: Verify Event Logs
Open Event Viewer and navigate to:

- `Applications and Services Logs`
  - `Microsoft`
    - `Windows`
      - `Sysmon`
        - `Operational`

You should begin seeing Sysmon events immediately.

---

## Step 7: Generate Test Events
Run the following commands in Command Prompt to generate Sysmon events:

```powershell
ipconfig
whoami
ping google.com
notepad.exe
```

---

## Step 8: Verify Events
Open Event Viewer again and confirm Sysmon events are being recorded.

Common event IDs:

| Event ID | Description | 
| --- | --- |
| `1` | Process Creation |
| `3` | Network Connection |
| `7` | Image Loaded |
| `8` | Create Remote Thread |
| `11` | File Created |
| `13` | Registry Value Set |
| `22` | DNS Query |

Event details may vary depending on the configuration file used.

---

## Useful Commands

| Command | Purpose |
| --- | --- |
| `Sysmon64.exe -i config.xml` | Install Sysmon |
| `Sysmon64.exe -c config.xml` | Update configuration |
| `Sysmon64.exe -c` | Display current configuration |
| `Sysmon64.exe -s` | Display configuration schema |
| `Sysmon64.exe -u` | Uninstall Sysmon |
| `sc query Sysmon` | Check service status |
| `Get-Service Sysmon` | Check service in PowerShell |
| `Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational"` | View Sysmon logs |

---

## Verification Checklist

- [x] Sysmon downloaded
- [x] Configuration file downloaded
- [x] Installed with administrator privileges
- [x] Service running
- [x] Event Viewer shows Sysmon logs
- [x] Process Creation events generated
- [x] Network Connection events generated
- [x] Configuration verified
- [x] Ready to integrate with Splunk, Microsoft Sentinel, or another SIEM

