
Install and Configure Sysmon on Windows
Overview
Sysmon (System Monitor) is a Windows system service developed by Microsoft Sysinternals. It provides detailed logging of system activities such as:
•	Process creation
•	Network connections
•	File creation time changes
•	Registry modifications
•	Driver loading
•	DNS queries
•	Image loading
These logs are extremely useful for threat hunting, malware analysis, digital forensics, and SIEM platforms such as Splunk. Sysmon runs as a Windows service and writes its events to Applications and Services Logs → Microsoft → Windows → Sysmon → Operational. 

Prerequisites
Before installing Sysmon, ensure you have:
•	Windows 10/11 or Windows Server
•	Administrator privileges
•	Internet connection
•	PowerShell or Command Prompt

Step 1: Create a Working Folder
Create a folder that will contain the Sysmon files.
C:\Tools\Sysmon
Your folder should eventually look like this:
C:\Tools\
    └── Sysmon\

Step 2: Download Sysmon
Download Sysmon from Microsoft's official Sysinternals page.

Official Download
Microsoft Sysinternals Sysmon Download
Extract the ZIP file.
After extraction, you should see:
Sysmon.exe
Sysmon64.exe
Eula.txt

Step 3: Download a Sysmon Configuration File
Sysmon does very little with its default configuration. Most security professionals use community-maintained configuration files.
One of the most widely used is the SwiftOnSecurity configuration.
Download:
sysmonconfig-export.xml
Save it inside
C:\Tools\Sysmon\
Now your folder should look like
C:\Tools\Sysmon\
    Sysmon64.exe
    Sysmon.exe
    sysmonconfig-export.xml

Step 4: Install Sysmon
Run
Sysmon64.exe -accepteula -i sysmonconfig-export.xml
Explanation:
Option	Meaning
-accepteula	Accept Microsoft EULA automatically
-i	Install Sysmon
sysmonconfig-export.xml	Load the configuration file
If successful, you'll see messages indicating that the Sysmon service and driver have been installed and started. 

Step 5: Verify Installation
Run
sc query Sysmon
Expected output
STATE : RUNNING
Or use
Get-Service Sysmon
PowerShell output
Status : Running

Step 6: Verify Event Logs
Open
Event Viewer
Navigate to
Applications and Services Logs
    Microsoft
        Windows
            Sysmon
                Operational
You should immediately begin seeing events.

Step 7: Generate Test Events
Open Command Prompt and execute
ipconfig
whoami
ping google.com
notepad.exe
These commands generate Sysmon events.

Step 8: Verify Events
Open Event Viewer again.
You should observe events such as:
Event ID	Description
1	Process Creation
3	Network Connection
7	Image Loaded
8	Create Remote Thread
11	File Created
13	Registry Value Set
22	DNS Query
The exact event types depend on your configuration file.

Useful Commands
Command	Purpose
Sysmon64.exe -i config.xml	Install Sysmon
Sysmon64.exe -c config.xml	Update configuration
Sysmon64.exe -c	Display current configuration
Sysmon64.exe -s	Display configuration schema
Sysmon64.exe -u	Uninstall Sysmon
sc query Sysmon	Check service status
Get-Service Sysmon	Check service in PowerShell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational"	View Sysmon logs

Verification Checklist
•	✅ Sysmon downloaded
•	✅ Configuration file downloaded
•	✅ Installed with administrator privileges
•	✅ Service running
•	✅ Event Viewer shows Sysmon logs
•	✅ Process Creation events generated
•	✅ Network Connection events generated
•	✅ Configuration verified
•	✅ Ready to integrate with Splunk, Microsoft Sentinel, or another SIEM

