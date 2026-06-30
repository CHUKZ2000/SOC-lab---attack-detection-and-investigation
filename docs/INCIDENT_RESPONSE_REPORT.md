# INCIDENT RESPONSE REPORT

## Unauthorized SSH Access via Brute-Force Credential Compromise

**Incident ID:** IR-2026-0619-SSH

**Detection Source:** Splunk Enterprise / Sysmon / Windows Event Log

**Affected Asset:** DESKTOP-C2V109N (10.1.1.105)

**Report Date:** June 19, 2026

| SEVERITY: HIGH | STATUS: CONTAINED | SCOPE: SINGLE HOST |
| --- | --- | --- |

## 1. Executive Summary

On June 19, 2026, monitoring within the SOC home lab environment detected and confirmed a successful unauthorized access incident against a Windows endpoint (DESKTOP-C2V109N, 10.1.1.105). The activity originated from a host at 10.1.1.106 and progressed through reconnaissance, credential brute-forcing, and post-access command execution before being fully identified and documented through Splunk and Sysmon telemetry.

The attacker conducted network reconnaissance against the target, identified an exposed SSH service, and used automated credential-guessing (Hydra) to obtain valid credentials for the local account "chukz." Following successful authentication, the attacker spawned a PowerShell session and attempted to execute a Base64-encoded command, a technique commonly used to evade detection and signature-based defenses.

*This was a controlled exercise conducted in an isolated home-lab environment; no production systems or external assets were affected. The findings below are presented in the same structure used for a real-world incident response engagement, including timeline, technical analysis, indicators of compromise, and remediation recommendations.*

| Incident Type | Unauthorized access via brute-forced SSH credentials |
| --- | --- |
| Root Cause | Weak/guessable password on local account "chukz"; SSH service exposed without lockout or rate-limiting controls |
| Impact | Attacker obtained interactive shell access and executed PowerShell commands on the target host |
| Detection Method | Sysmon Event IDs 1 & 3, Windows Event ID 4625, correlated in Splunk Enterprise |
| Current Status | Contained — lab exercise concluded; remediation recommendations issued below |

## 2. Incident Timeline

All times reflect the lab environment clock (June 19, 2026). The sequence below reconstructs attacker activity using Sysmon, Windows Event Log, and Splunk evidence.

| Time | Stage | Activity |
| --- | --- | --- |
| 07:35 | Reconnaissance | SYN scan (nmap -sS) from attacker host identifies SSH (port 22) open on 10.1.1.105 |
| 08:07 | Reconnaissance | Aggressive scan (nmap -A) performs service/version detection and OS fingerprinting against the same host |
| 12:52 | Credential Attack | Hydra brute-force attack launched against SSH using a username/password list; 4 failed attempts logged (Event ID 4625) before one valid credential pair (account: chukz) is identified |
| 12:52 | Initial Access | Successful SSH authentication achieved as "chukz" — attacker gains interactive shell access |
| ~12:54–20:35 | Execution | PowerShell session spawned from the compromised account; Sysmon Event ID 1 logs process creation for powershell.exe |
| 21:01 | Defense Evasion | Attempted execution of a Base64-encoded PowerShell command (-EncodedCommand); command fails with a parser error but the attempt is captured in Sysmon command-line logging |

## 3. Technical Analysis

### 3.1 Reconnaissance

The attacker began with a SYN scan to identify open ports on the target, followed by an aggressive scan to enumerate services, versions, and the operating system. Sysmon Event ID 3 (network connection) recorded each probe.

```
nmap -sS 10.1.1.105
nmap -A 10.1.1.105
```

*Figure 1. Initial SYN scan from the attacker host (Kali Linux) showing port 22/tcp open on the target.*

*Figure 2. Aggressive scan (-A) returning OS fingerprint guesses and SSH service/version banner (OpenSSH for_Windows_7.7).*

Detection query used to identify the scanning activity in Splunk:

```
index=sysmon EventCode=3
| stats dc(DestinationPort) as PortsScanned by SourceIp DestinationIp
```

*Figure 3. Splunk results confirming the scanning source (10.1.1.106) and a one-to-many destination port pattern consistent with reconnaissance.*

### 3.2 Credential Attack & Initial Access

Following reconnaissance, the attacker used Hydra to perform a dictionary-based brute-force attack against the exposed SSH service. The attack used a username and password list targeting the discovered host.

```
hydra -l username.txt -P passwd.txt -t 4 ssh://10.1.1.105
```

*Figure 4. Hydra output showing a successful credential match — 1 of 1 target compromised, valid password found for account "chukz."*

Windows Event ID 4625 (failed logon) corroborates the brute-force attempts immediately preceding the successful authentication:

```
index=wineventlog EventCode=4625
| stats count by Account_Name Source_Network_Address
```

*Figure 5. Failed logon events show 4 unsuccessful attempts against the "chukz" account shortly before the successful Hydra authentication.*

This represents a confirmed account compromise rather than a contained attempt: the attacker obtained valid, working credentials and used them to establish an interactive session.

### 3.3 Post-Access Execution & Evasion Attempt

After authenticating, the attacker (operating as "chukz") spawned a Windows PowerShell session — a common next step for situational awareness, persistence, or lateral movement.

*Figure 6. PowerShell session opened from the compromised "chukz" account.*

The attacker subsequently attempted to run a Base64-encoded command using the -EncodedCommand parameter, a technique frequently used to obscure command intent from logging and signature-based detection. The attempt itself failed with a PowerShell parser error, indicating malformed syntax rather than a successfully executed payload — but the attempt was still captured by command-line logging.

```
index=sysmon EventCode=1 CommandLine="*-EncodedCommand*"
```

*Figure 7. Splunk detection query matching the encoded-command pattern in Sysmon process-creation logs (1 event matched).*

## 4. Indicators of Compromise

| Type | Indicator | Context |
| --- | --- | --- |
| Source IP | 10.1.1.106 | Attacker-controlled host |
| Destination IP | 10.1.1.105 | Target endpoint (DESKTOP-C2V109N) |
| Compromised Account | chukz | Local account with weak/guessable password |
| Process | hydra | Brute-force tool used for credential attack |
| Process | powershell.exe | Spawned post-compromise; used for encoded command attempt |
| Process | nmap | Used for reconnaissance (-sS and -A scans) |
| Sysmon Event ID 1 | Process Creation | Captured powershell.exe launch and encoded command line |
| Sysmon Event ID 3 | Network Connection | Captured scan traffic and SSH connection from 10.1.1.106 |
| Windows Event ID 4625 | Failed Logon | Captured 4 failed brute-force attempts against "chukz" |

## 5. MITRE ATT&CK Mapping

| Activity | ATT&CK ID | Technique | Tactic |
| --- | --- | --- | --- |
| Network Scan | T1046 | Network Service Discovery | Discovery |
| OS/Service Fingerprinting | T1592 | Gather Victim Host Information | Reconnaissance |
| Brute Force | T1110 | Brute Force | Credential Access |
| Valid Account Use | T1078 | Valid Accounts | Initial Access |
| PowerShell Execution | T1059.001 | PowerShell | Execution |
| Command Execution | T1059 | Command and Scripting Interpreter | Execution |
| Encoded Command | T1027 | Obfuscated Files or Information | Defense Evasion |

## 6. Impact Assessment

- **Confidentiality:** Attacker gained shell-level access to the target host using valid credentials, exposing any data accessible to the "chukz" account.
- **Integrity:** No evidence of file modification or persistence mechanisms was observed in the captured telemetry; the encoded PowerShell command failed to execute.
- **Availability:** No denial-of-service or destructive activity was observed; the target host remained operational throughout.
- **Scope:** Activity was confined to a single endpoint (10.1.1.105) within an isolated lab network; no lateral movement or external communication was identified in the available logs.

## 7. Recommendations

### 7.1 Immediate Actions

- Reset the password for the "chukz" account and audit it for any other reused or weak credentials.
- Review SSH access logs for the affected host for any additional logins from the same or related source IPs.
- Isolate or restrict SSH exposure on the target host to trusted network segments only.

### 7.2 Detection & Monitoring Improvements

- Configure Splunk alerting on repeated Event ID 4625 failures from a single source within a short time window (brute-force threshold alerting).
- Create a standing detection rule for PowerShell invocations using -EncodedCommand, -EncodedArguments, or similarly obfuscated parameters.
- Baseline normal parent-child process relationships (e.g., sshd → shell → powershell.exe) to flag anomalous chains.
- Extend Sysmon coverage and log forwarding to all endpoints, not just the lab target, to ensure consistent visibility.

### 7.3 Preventive Controls

- Enforce strong, non-guessable password policies and minimum complexity requirements for all local and service accounts.
- Implement account lockout thresholds after repeated failed authentication attempts.
- Where feasible, require key-based authentication or multi-factor authentication for SSH instead of password-only logins.
- Disable or restrict SSH on hosts where it is not operationally required.

## 8. Conclusion

This incident demonstrates a complete, low-complexity attack chain — reconnaissance, credential brute-forcing, initial access, and attempted defense evasion — that was fully visible through Sysmon and Windows Event Log telemetry once centralized in Splunk. The key control gap was the use of a weak, brute-forceable password on an internet-facing-style SSH service with no lockout policy. Implementing the recommendations above (account lockout, stronger credential policy, and proactive brute-force/encoded-command alerting) would have prevented or significantly shortened this attack chain. This exercise validates the SOC lab's detection capability across the full incident lifecycle and provides a template for future detection engineering and tabletop exercises.

## 8. Splunk Dashboard

*Figure 8. Splunk Dashboard*

*Figure 9. Splunk Dashboard 2*

*Figure 10. Splunk Dashboard 3*

*Figure 11. Splunk Dashboard 4*

*Figure 12. Splunk Dashboard 5*

---
IR-2026-0619-SSH | Page 1 of 2
