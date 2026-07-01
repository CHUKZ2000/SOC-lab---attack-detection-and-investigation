# Splunk Installation and Configuration Guide

## Lab Architecture

```
+-----------------------+        Port 9997        +------------------------+
| Windows 10/11 Client  | ---------------------> | Splunk Enterprise      |
| Sysmon Installed      |                        | (Indexer/Search Head)  |
| Universal Forwarder   |                        |                        |
+-----------------------+                        +------------------------+
        |
        | Windows Logs
        | Sysmon Logs
        | Security Logs
```

---

## Part 1 — Install Splunk Enterprise

### Step 1: Download
- Go to Splunk Enterprise Downloads
- Download: `Splunk Enterprise (Windows 64-bit)`

### Step 2: Run the installer
- Double-click the downloaded installer
- Accept the license agreement
- Leave installation folder as default:
  - `C:\Program Files\Splunk`

### Step 3: Create administrator account
- Username: `admin`
- Password: `StrongPassword123!`
- Click `Next`, then `Install`
- Wait until installation completes

### Step 4: Open Splunk Web
- Open browser: `http://localhost:8000`
- Login with:
  - Username: `admin`
  - Password: `StrongPassword123!`
- Confirm the Splunk dashboard loads successfully

---

## Part 2 — Configure Receiving Port

Universal Forwarders require the indexer to listen on a receiving port.

### Configure Splunk to receive data
1. Go to `Settings`
2. Select `Forwarding and Receiving`
3. Click `Configure Receiving`
4. Click `New Receiving Port`
5. Enter port: `9997`
6. Click `Save`

> Splunk is now listening for incoming forwarders on port `9997`.

---

## Part 3 — Create an Index (Optional but Recommended)

### Add a new index
1. Go to `Settings`
2. Select `Indexes`
3. Click `New Index`
4. Name: `windows`
5. Click `Save`

### Optional additional indexes
- `sysmon`
- `security`
- `network`

---

## Part 4 — Install Splunk Universal Forwarder

### Step 1: Download
- Go to Splunk Universal Forwarder Downloads
- Choose: `Windows 64-bit MSI`

### Step 2: Run the installer
- Run the downloaded MSI
- Accept the license agreement
- Click `Next`

### Step 3: Install location
- Leave default installation folder:
  - `C:\Program Files\SplunkUniversalForwarder`

### Step 4: Choose service account
- For a lab environment, select `Local System`
- Click `Next`

### Step 5: Create Universal Forwarder admin account
- Username: `admin`
- Password: `StrongPassword123!`

### Step 6: Configure receiving indexer
- Enter indexer address: `192.168.56.20:9997`
  - Replace `192.168.56.20` with your Splunk server IP
- Click `Next`
- Click `Install`

> The Splunk Universal Forwarder service should start automatically after installation.

---

## Part 5 — Verify Universal Forwarder

### Confirm service status
- Open `Services`
- Confirm `SplunkForwarder` status is `Running`

---

## Part 6 — Configure `outputs.conf`

### File location
- `C:\Program Files\SplunkUniversalForwarder\etc\system\local`

### Create `outputs.conf`

```ini
[tcpout]
defaultGroup = indexers

[tcpout:indexers]
server = 192.168.56.20:9997

[tcpout-server://192.168.56.20:9997]
```

> Replace `192.168.56.20` with your Splunk server IP.

### Restart the Universal Forwarder
1. Open Command Prompt as Administrator
2. Run:
   ```powershell
   cd "C:\Program Files\SplunkUniversalForwarder\bin"
   splunk restart
   ```

---

## Part 7 — Configure `inputs.conf`

### File location
- `C:\Program Files\SplunkUniversalForwarder\etc\system\local`

### Create `inputs.conf`

```ini
[WinEventLog://Application]
disabled = 0
index = windows

[WinEventLog://Security]
disabled = 0
index = windows

[WinEventLog://System]
disabled = 0
index = windows

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = sysmon
```

### Collected data
- Application logs
- Security logs
- System logs
- Sysmon event logs

> Restart the forwarder after saving `inputs.conf`.

---

## Part 8 — Open Firewall Port on Splunk Enterprise

### Create inbound rule
1. Open `Windows Defender Firewall`
2. Go to `Inbound Rules`
3. Click `New Rule`
4. Select `Port`
5. Choose `TCP`
6. Enter port: `9997`
7. Select `Allow the connection`
8. Finish the rule wizard

---

## Part 9 — Verify Connection in Splunk

### Search for forwarded events
- `index=windows`
  - Confirm Windows event data is arriving
- `index=sysmon`
  - Confirm Sysmon event data is arriving

---

## Part 10 — Useful Search Queries

- Windows Logons:
  ```spl
  index=windows EventCode=4624
  ```
- Failed Logins:
  ```spl
  index=windows EventCode=4625
  ```
- PowerShell activity:
  ```spl
  index=sysmon Image="*powershell.exe"
  ```
- Network Connections:
  ```spl
  index=sysmon EventCode=3
  ```
- Process Creation:
  ```spl
  index=sysmon EventCode=1
  ```
- RDP Logins:
  ```spl
  index=windows EventCode=4624 Logon_Type=10
  ```

---

## Troubleshooting and Validation

### Verify forwarder status
```powershell
cd "C:\Program Files\SplunkUniversalForwarder\bin"
splunk status
```

### Restart forwarder
```powershell
splunk restart
```

### View forwarder logs
```powershell
cd "C:\Program Files\SplunkUniversalForwarder\var\log\splunk"
notepad splunkd.log
```
