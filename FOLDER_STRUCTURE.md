# Repository Folder Structure

This repository is organized to reflect the SOC home lab workflow described in the project documentation: attack simulation (Kali Linux) → telemetry collection (Sysmon / Windows Event Log) → log forwarding and indexing (Splunk) → detection, threat hunting, and reporting.

```
soc-home-lab/
├── README.md
├── docs/
│   ├── INCIDENT_RESPONSE_REPORT.md
│   └── IOCs.md
├── screenshots/
│   ├── reconnaissance/
│   ├── brute-force/
│   ├── powershell-execution/
│   ├── encoded-command/
│   └── splunk-dashboards/
├── attack-scripts/
│   ├── nmap-syn-scan.txt
│   ├── nmap-aggressive-scan.txt
│   ├── hydra-ssh-bruteforce.txt
│   ├── powershell-execution.txt
│   └── powershell-encoded-command.txt
├── detection-queries/
│   ├── reconnaissance/
│   │   ├── syn-scan-detection.spl
│   │   └── aggressive-scan-detection.spl
│   ├── brute-force/
│   │   └── ssh-bruteforce-detection.spl
│   ├── powershell-execution/
│   │   ├── powershell-execution-detection.spl
│   │   └── encoded-command-detection.spl
│   └── threat-hunting/
│       ├── hunt1-powershell-usage.spl
│       ├── hunt2-cmd-usage.spl
│       └── hunt3-parent-child-process.spl
├── mitre-attack-mapping/
│   └── mitre-mapping.md
└── configs/
    ├── sysmon/
    └── splunk/
```

## Folder Descriptions

| Folder / File | Purpose |
| --- | --- |
| `README.md` | Main project overview: executive summary, objectives, lab architecture, tools used, attack scenarios, threat hunting, IOCs, MITRE mapping, recommendations, and conclusion. |
| `docs/INCIDENT_RESPONSE_REPORT.md` | Full incident response report for the SSH brute-force compromise (IR-2026-0619-SSH), including timeline, technical analysis, impact assessment, and remediation recommendations. |
| `docs/IOCs.md` | Standalone list of indicators of compromise (IPs, processes, event IDs) identified during the lab exercise. |
| `screenshots/` | Evidence captures referenced in the reports (e.g., Figures 1–12: nmap scans, Hydra output, failed logon events, PowerShell session, Splunk dashboards). Subfolders correspond to each attack stage. |
| `attack-scripts/` | The exact attacker-side commands used in each scenario (Nmap reconnaissance, Hydra brute force, PowerShell execution and encoded command). |
| `detection-queries/` | Splunk Search Processing Language (SPL) queries used to detect each corresponding attack technique, organized by category. |
| `mitre-attack-mapping/` | MITRE ATT&CK technique/tactic mapping for all observed activity. |
| `configs/sysmon/` | Place for Sysmon configuration files (e.g., `sysmonconfig.xml`) used to generate the endpoint telemetry. |
| `configs/splunk/` | Place for Splunk-side configuration files (e.g., inputs.conf, props.conf for the Universal Forwarder/indexer setup). |

> Note: `.gitkeep` files are included in otherwise-empty folders (`screenshots/*`, `configs/*`) purely so Git tracks the empty directories — they should be removed/replaced once actual screenshots or config files are added.
