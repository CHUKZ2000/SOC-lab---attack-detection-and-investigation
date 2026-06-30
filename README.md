# SOC Home Lab: Attack Detection and Incident Response Using Splunk, Sysmon, Windows, and Kali Linux

## Executive Summary

This project demonstrates the implementation of a Security Operations Center (SOC) home lab designed to simulate real-world cyberattacks and detect malicious activities using Splunk and Sysmon. The objective was to generate attack telemetry from a Kali Linux attacker machine, collect logs from a Windows endpoint using Sysmon and Splunk Universal Forwarder, and analyze the resulting events in Splunk Enterprise.

Several attack scenarios were executed, including network reconnaissance using Nmap, brute-force authentication attempts, and PowerShell execution. The collected logs were analyzed to identify indicators of compromise (IOCs), map activities to the MITRE ATT&CK framework, and develop detection queries capable of identifying suspicious behaviour.

The project successfully demonstrated the end-to-end SOC workflow, including log collection, alert generation, threat hunting, incident investigation, and reporting.

## Project Objectives

The primary objectives of this project were:

- Deploying a functional SOC home lab environment.
- Collecting endpoint and security logs using Sysmon and Splunk.
- Simulating adversarial activities using Kali Linux.
- Developing detection queries for malicious behaviours.
- Investigating generated security events.
- Producing an incident report documenting findings.
- Mapping observed techniques to the MITRE ATT&CK framework.

## Lab Architecture

**Attacker Machine**
- Kali Linux

**Target Machine**
- Windows 10

**Monitoring Server**
- Windows 10

### Data Flow

Kali Linux generated attack traffic toward the Windows endpoint. Sysmon captured process creation, network connections, and system activities. Splunk Universal Forwarder transmitted logs to Splunk Enterprise for indexing, analysis, and visualization.

## Tools Used

| Tool | Purpose |
| --- | --- |
| Splunk Enterprise | Log Management and SIEM |
| Sysmon | Endpoint Telemetry Collection |
| Splunk Universal Forwarder | Log Forwarding |
| Kali Linux | Attack Simulation |
| Nmap | Network Reconnaissance |
| Hydra | Brute Force Simulation |
| Windows Event Logs | Authentication Monitoring |

## Attack Scenario 1: Network Reconnaissance

**Objective**

I Simulated an attack performing network reconnaissance against the target host.

**Attack Command**

```
nmap -sS TARGET_IP
```

**Observed Activity**

The Windows endpoint received multiple connection attempts across numerous ports. Sysmon generated Event ID 3 records for each network connection.

**Detection Query**

```
index=sysmon EventCode=3
| stats count by SourceIp DestinationPort
```

**Findings**

The source IP generated connection attempts to multiple ports within a short period. This behavior is consistent with reconnaissance activities commonly performed by attackers prior to exploitation.

## Attack Scenario 2: Aggressive Nmap Scan

**Objective**

I Identifed service enumeration and operating system fingerprinting attempts.

**Attack Command**

```
nmap -A TARGET_IP
```

**Detection Query**

```
index=sysmon EventCode=3
| stats dc(DestinationPort) as PortsScanned by SourceIp DestinationIp
```

**Findings**

The attacker scanned a large number of ports and attempted service detection. Such activity can indicate pre-exploitation intelligence gathering.

## Attack Scenario 3: Authentication Brute Force

**Objective**

I Simulated unauthorized login attempts.

**Attack Tool**

Hydra

**Attack Command**

```
hydra -l administrator -P passwords.txt ssh://TARGET_IP
```

**Detection Query**

```
index=wineventlog EventCode=4625
| stats count by Account_Name Source_Network_Address
```

**Findings**

Multiple failed authentication events were generated from a single source IP. Repeated login failures against privileged accounts may indicate credential attack activity.

## Attack Scenario 4: PowerShell Execution

**Objective**

I Monitored potentially suspicious command-line activity.

**Command Executed**

```
powershell.exe
```

**Detection Query**

```
index=sysmon Image="*powershell.exe"
```

**Findings**

Sysmon successfully recorded PowerShell execution events. PowerShell is frequently abused by threat actors for execution, persistence, and lateral movement.

## Attack Scenario 5: Encoded PowerShell Commands

**Objective**

I Detected obfuscated PowerShell activity.

**Command Executed**

```
powershell.exe -EncodedCommand
```

**Detection Query**

```
index=sysmon EventCode=1 CommandLine="*-EncodedCommand*"
```

**Findings**

The encoded command parameter was successfully identified within process creation logs. This behavior is often associated with malicious scripts attempting to evade detection.

## Threat Hunting Activities

### Hunt 1: PowerShell Usage

**Query:**

```
index=sysmon Image="*powershell.exe"
```

**Purpose:**

Identify command-line execution activity and suspicious scripting behavior.

### Hunt 2: Command Prompt Usage

**Query:**

```
index=sysmon Image="*cmd.exe"
```

**Purpose:**

Identify command execution activity and possible attacker interaction.

### Hunt 3: Parent-Child Process Relationships

**Query:**

```
index=sysmon EventCode=1
| stats count by ParentImage Image
```

**Purpose:**

Detect unusual process relationships that may indicate malicious execution chains.

## Indicators of Compromise (IOCs)

**IP Addresses**

- SourceIp-10.1.1.105
- DestinationIp-10.1.1.106

**Processes**

- powershell.exe
- cmd.exe
- hydra
- nmap

**Event IDs**

- Sysmon Event ID 1 (Process Creation)
- Sysmon Event ID 3 (Network Connection)
- Windows Event ID 4625 (Failed Login)

## MITRE ATT&CK Mapping

| Activity | ATT&CK ID | Technique |
| --- | --- | --- |
| Network Scan | T1046 | Network Service Discovery |
| Brute Force | T1110 | Brute Force |
| PowerShell Execution | T1059.001 | PowerShell |
| Command Execution | T1059 | Command and Scripting Interpreter |
| Account Discovery | T1087 | Account Discovery |

## Recommendations

- Enable endpoint logging using Sysmon across all systems.
- Configure Splunk alerts for excessive failed login attempts.
- Monitor PowerShell execution and encoded commands.
- Establish baseline network behaviour to identify anomalies.
- Implement account lockout policies to mitigate brute-force attacks.
- Regularly review authentication logs and endpoint telemetry.

## Conclusion

This project successfully demonstrated the deployment and operation of a SOC home lab capable of detecting and investigating common attack techniques. Using Splunk and Sysmon, security events generated from simulated attacks were collected, analyzed, and correlated to produce actionable findings.

The project highlights practical SOC analyst skills, including threat detection, log analysis, incident investigation, threat hunting, and MITRE ATT&CK mapping. The resulting environment serves as a foundation for future detection engineering and incident response exercises.
