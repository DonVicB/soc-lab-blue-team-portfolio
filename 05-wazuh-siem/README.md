# Wazuh SIEM — SOC lab deployment

> **Documentation status:** In progress. This page distinguishes the lab owner's live checks from previously discussed configuration and from tests not yet reproduced for the portfolio.

## Objective

Deploy Wazuh as a central security monitoring platform for the VMware SOC training lab. Collect endpoint security telemetry, inspect alerts, and develop repeatable blue-team investigations.

## Environment

| Component | Current evidence |
| --- | --- |
| VMware VM display name | `Ubuntu Wazuh` — verified from VMware screenshot, 24 October 2026 |
| Ubuntu guest hostname | `wazuh-server` — visible in terminal prompt and login banner |
| Operating system | Ubuntu 26.04.1 LTS — as shown in guest login banner |
| Internal management address | `10.10.10.20/24` on `ens160` (VMnet1 subnet) — screenshot verified |
| NAT address | `192.168.111.128/24` on `ens192` — screenshot verified |
| Default IPv4 route | Via `192.168.111.2` on `ens192` — screenshot verified |
| Wazuh manager service | **active** — screenshot-verified from `sudo systemctl is-active wazuh-manager`, 7 November 2026 |

See [VMware network inventory](../01-lab-architecture/guest-ip-dns.md) for the detailed interface and DNS evidence.

## Service verification

The lab owner ran:

```bash
sudo systemctl is-active wazuh-manager
```

**Screenshot-verified result:** `active` (7 November 2026).

**Interpretation:** At the time of the check, systemd reported the `wazuh-manager` service as active. This confirms manager process state as reported by the operator, **not** that events are being received, all manager components are healthy, agents are connected, or alerts are being generated.

## Workstreams to document

- [x] Record screenshot-verified Wazuh manager service status.
- [ ] Capture the installed Wazuh manager version.
- [x] Enumerate registered agents and their current status using the manager CLI (7 November 2026; two endpoints disconnected).
- [ ] Document Windows 10 agent connectivity and endpoint log collection (current agent status: disconnected; do not claim live collection).
- [ ] Document the Sysmon event source and one redacted example detection.
- [ ] Document the previously discussed ATT&CK **T1059.001 (PowerShell)** custom detection rule, including the exact rule, test event and alert evidence, when collected.
- [ ] Document a repeatable alert investigation with timestamps and redacted supporting evidence.

## Agent inventory — 7 November 2026

**Evidence:** Screenshot of the Wazuh Ubuntu VM terminal with manager service check and corrected agent-list command:

```bash
sudo systemctl is-active wazuh-manager
sudo /var/ossec/bin/agent_control -l
```

The first agent command omitted the leading `/` in the absolute path and resulted in `command not found`. The corrected command executed successfully. No agent state was modified.

| ID | Displayed agent | Reported address | Status at capture |
| --- | --- | --- | --- |
| `000` | `wazuh-server` (server) | `127.0.0.1` | **Active/Local** |
| `001` | `DonW10` | `any` | **Disconnected** |
| `002` | Second registered Windows desktop (hostname omitted in portfolio) | `any` | **Disconnected** |

**Assessment:** Wazuh Manager is active and has two registered endpoint agents, but **0 of 2 endpoint agents are connected** in this screenshot. The local manager entry `000` is not an additional remotely monitored Windows endpoint. The `IP: any` field is the registered agent's address setting, **not** proof of an active connection or the guest's current IP. The list of agentless devices is empty.

**Scope:** This confirms registration and connection states at the time of capture; it does **not** establish why agents were disconnected, whether they were previously connected, whether historical logs are still searchable, or whether any Sysmon detection fired recently. Record the observation without changing endpoints during documentation.

## Next read-only evidence collection

Record the installed Wazuh version on the Ubuntu manager:

```bash
sudo /var/ossec/bin/wazuh-control info
```

Then document the configured agent event sources and existing PowerShell/Sysmon detection evidence separately. Do not represent these disconnected agents as actively streaming events unless a subsequent check confirms that.

## Security and evidence handling

Do not commit authentication credentials, agent registration keys, certificates/private keys, raw endpoint logs containing personal information or full configuration exports. Lab IPs are included here for private inventory; review them before publishing the repository publicly.

*Last updated: 7 November 2026.*
