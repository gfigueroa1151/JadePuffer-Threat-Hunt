# JadePuffer-Threat-Hunt
Full Threat Hunting Report of JadePuffer Incident


# Hunt 23 — JadePuffer Threat Hunt Report

## Overview
This repository contains my threat hunting report and documentation for **Hunt 23: JadePuffer**, completed as part of a cybersecurity internship in a simulated Azure/Microsoft Sentinel environment.

The scenario is based on the **first documented end-to-end agentic ransomware operation**, as reported by the Sysdig Threat Research Team in July 2026. An autonomous LLM agent completed a full ransomware kill chain across four Linux hosts in under 17 minutes — with a single human prompt as its only instruction.

---

## Scenario Summary
- **Threat Actor:** Autonomous LLM agent (`jadepuffer-agent`, session `jp-7f3c9a21`)
- **Initial Access:** CVE-2025-3248 — Unauthenticated RCE via Langflow's `/api/v1/validate/code` endpoint
- **Duration:** ~17 minutes (19:20–19:37 UTC)
- **Hosts Compromised:** ff-lf-01, ff-minio-01, ff-nacos-01, ff-db-01

---

## Kill Chain
| Stage | Finding |
|---|---|
| Initial Access | CVE-2025-3248 exploited via unauthenticated POST request |
| Command & Control | Reverse shell beacon to 45.131.66.106:4444, secondary channel on 104.16.132.229:8080 |
| C2 Persistence | Cron job planted on langflow account, beaconing every minute |
| Credential Access | pg_dump exfiltrated 214 secrets across 10 provider families |
| Discovery | Subnet sweep of 10.4.0.0/24; MinIO default credentials (minioadmin:minioadmin) accepted |
| Lateral Movement | terraform-state/credentials.json retrieved from MinIO bucket |
| Privilege Escalation | Nacos JWT forged; backdoor account `svc_maint` (UID 997) created |
| Impact | AES_ENCRYPT on 1,342 rows; DROP TABLE config_info, history; ransom note inserted |

---

## Tools & Environment
- **SIEM:** Microsoft Sentinel (Log Analytics Workspace: LAW-HuntPractice)
- **Query Language:** KQL (Kusto Query Language)
- **Log Sources:** 10 custom Linux log tables including `LLMAgentLogs_CL`, `LinuxProcess_CL`, `LinuxSystem_CL`, `LinuxAuth_CL`, and more

---

## Key Takeaway
The `LLMAgentLogs_CL` table proved to be the most valuable detection surface throughout this hunt. Because the agent narrated its own reasoning in real time, it provided a complete picture of attacker intent that traditional process and network logs alone could not. This introduces a novel detection opportunity specific to agentic attacks.

---

## Files
- `JadePuffer_Threat_Hunting_Report.docx` — Full DFIR-style incident report

---

## About
**Analyst:** Giancarlos Figueroa-Robles  
IT Support Specialist transitioning into blue team cybersecurity (SOC/Incident Response).  
Certifications: CompTIA Security+, PenTest+, SSCP, Network+, A+, Project+
