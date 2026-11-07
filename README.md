# SOC Lab | Blue Team Cybersecurity Portfolio

Hands-on Security Operations Centre (SOC) home lab documenting security infrastructure, endpoint telemetry, threat detection, adversary emulation, and incident response.

> **Repository status:** Documentation in progress. This repository records practical lab work and is not a production environment.

## About this project

I built this lab to strengthen my practical blue-team skills through designing systems, collecting security logs, simulating attacker behaviour in an authorised environment, and investigating the resulting events.

The goal is to make each exercise repeatable and explain **what happened, how it was detected, how it was investigated, and what should be improved**.

## Lab environment

| Component | Purpose | Documentation status |
| --- | --- | --- |
| VMware Workstation | Virtualisation and isolated lab networks | [Eight-VM inventory and topology documented](01-lab-architecture/README.md); some configuration discrepancies flagged |
| Windows Server 2019 / Active Directory Domain Services | Identity management, domain controller, DNS, users and groups | To document |
| Windows 10 endpoint | Workstation and security telemetry | [Windows endpoint evidence](04-windows-endpoint-security/README.md): Sysmon process creation and active Wazuh agent verified |
| Sysmon (System Monitor) | Detailed Windows endpoint events | [Source and Wazuh process-creation alert evidence](04-windows-endpoint-security/README.md): 60 parsed Sysmon Event ID 1 manager alerts |
| Wazuh | Security information and event management (SIEM), alerts and rules | [Manager and detection evidence](05-wazuh-siem/README.md): active agent `002`, 60 parsed Sysmon Event ID 1 alerts across six rules; custom rule `100101` unverified |
| Splunk | Security log searches and investigations | [Installed version verified](06-splunk-siem/README.md): Splunk 10.6.1.1 (7 Nov 2026); service, ingestion and investigation evidence pending |
| Kali Linux | Authorised attack simulation | To document |
| MITRE CALDERA | Adversary emulation | Setup / validation to document |
| Wireshark | Network traffic analysis | To document |

The inventory and configurations will be checked against the current virtual machines before detailed setup guides are published.

## Security exercises

### PowerShell activity detection

- **ATT&CK technique:** T1059.001 — Command and Scripting Interpreter: PowerShell.
- **Focus:** Generate observable endpoint activity, inspect events, and validate detection coverage.
- **Planned evidence:** Relevant event fields, detection logic, analysis notes and redacted screenshots.

### Microsoft Defender / Sandcat investigation

- **Focus:** Investigate a Microsoft Defender quarantine or blocking event associated with MITRE CALDERA's Sandcat agent.
- **Planned evidence:** Defender detection details, corresponding Splunk events, timeline, ATT&CK mapping and analyst conclusions.
- **Status:** Investigation documentation pending; no outcome is claimed here.

## Lab architecture documentation

- [VMware SOC lab architecture](01-lab-architecture/README.md) — all eight guest internal IPv4 addresses documented from live evidence; some routing/DNS and Metasploitable3 isolation remain to verify.
- [Wazuh SIEM deployment and verification](05-wazuh-siem/README.md) — active Windows agent `002`, 60 parsed Sysmon Event ID 1 alert records; custom PowerShell rule `100101` not yet validated.
- [Detection engineering index](09-detection-rules/README.md) — PowerShell rule `100101`, MITRE T1059.001, configuration verified; alert validation pending.
- [Windows 10 endpoint security](04-windows-endpoint-security/README.md) — local Sysmon Event ID 1 and manager alert evidence verified.
- [Splunk SIEM search and investigation plan](06-splunk-siem/README.md) — installed Splunk 10.6.1.1 verified; next service status and log-source evidence.

## Planned repository structure

```text
01-lab-architecture/
02-vmware-networking/
03-active-directory/
04-windows-endpoint-security/
05-wazuh-siem/
06-splunk-siem/
07-kali-attack-simulations/
08-mitre-caldera/
09-detection-rules/
10-incident-investigations/
11-automation-scripts/
12-screenshots-and-diagrams/
```

Documentation, example configurations, lab-safe scripts and redacted supporting evidence will be added incrementally. The folders will be created as content is ready.

## Standard investigation format

Each case study should cover:

1. **Objective and scope** — What was tested and which systems were authorised.
2. **Lab setup** — Relevant components and logging sources.
3. **Execution** — Repeatable procedure and commands, with explanations.
4. **Evidence** — Timestamps, event IDs, queries, alerts and safely redacted screenshots.
5. **Analysis** — Event correlation, ATT&CK mapping and findings.
6. **Response and improvements** — Containment considerations, tuning and lessons learned.

## Security and privacy

- This is an **isolated, authorised training lab**.
- Do not commit credentials, tokens, certificates, private keys, password vaults, or unredacted configuration exports.
- Redact personal data, device identifiers and any network details that should remain private.
- Keep VMware disks, snapshots, packet captures containing sensitive traffic, and other large artefacts outside Git.
- Review all material before changing this repository from private to public.

## Roadmap

- [x] Create a private GitHub repository.
- [x] Add an initial portfolio README.
- [x] Add `.gitignore` to exclude secrets, VM disks, and raw SOC evidence.
- [x] Capture current eight-VM IP inventory (some mismatches and routing/segmentation checks remain).
- [x] Create network architecture diagram (annotated with evidence gaps).
- [ ] Document Windows Server / Active Directory and Windows 10.
- [x] Document Sysmon → Wazuh alert baseline (60 parsed Event ID 1 records; custom rule `100101` pending separate validation).
- [ ] Document Splunk ingestion, searches and dashboards.
- [ ] Document controlled Kali / CALDERA simulations.
- [ ] Publish evidence-based investigation case studies.
- [ ] Review for sensitive information before any public portfolio release.

---

*Built for hands-on SOC analyst learning, detection engineering, and incident response practice.*
