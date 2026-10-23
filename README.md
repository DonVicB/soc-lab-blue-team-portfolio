# SOC Lab | Blue Team Cybersecurity Portfolio

Hands-on Security Operations Centre (SOC) home lab documenting security infrastructure, endpoint telemetry, threat detection, adversary emulation, and incident response.

> **Repository status:** Documentation in progress. This repository records practical lab work and is not a production environment.

## About this project

I built this lab to strengthen my practical blue-team skills through designing systems, collecting security logs, simulating attacker behaviour in an authorised environment, and investigating the resulting events.

The goal is to make each exercise repeatable and explain **what happened, how it was detected, how it was investigated, and what should be improved**.

## Lab environment

| Component | Purpose | Documentation status |
| --- | --- | --- |
| VMware Workstation | Virtualisation and isolated lab networks | To document |
| Windows Server 2019 / Active Directory Domain Services | Identity management, domain controller, DNS, users and groups | To document |
| Windows 10 endpoint | Domain-joined workstation and security telemetry | To document |
| Sysmon (System Monitor) | Detailed Windows endpoint events | To document |
| Wazuh | Security information and event management (SIEM), alerts and rules | To document |
| Splunk | Security log searches and investigations | Setup / validation to document |
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
- [ ] Add `.gitignore` and contributor-safe documentation rules.
- [ ] Capture and sanitise current lab inventory.
- [ ] Create a current-state network architecture diagram.
- [ ] Document Windows Server / Active Directory and Windows 10.
- [ ] Document Sysmon, Wazuh and validated detection rules.
- [ ] Document Splunk ingestion, searches and dashboards.
- [ ] Document controlled Kali / CALDERA simulations.
- [ ] Publish evidence-based investigation case studies.
- [ ] Review for sensitive information before any public portfolio release.

---

*Built for hands-on SOC analyst learning, detection engineering, and incident response practice.*
