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
| Wazuh manager service | **active** — owner-reported result from `sudo systemctl is-active wazuh-manager` on 7 November 2026; command output not supplied as screenshot |

See [VMware network inventory](../01-lab-architecture/guest-ip-dns.md) for the detailed interface and DNS evidence.

## Service verification

The lab owner ran:

```bash
sudo systemctl is-active wazuh-manager
```

**Reported result:** `active`.

**Interpretation:** At the time of the check, systemd reported the `wazuh-manager` service as active. This confirms manager process state as reported by the operator, **not** that events are being received, all manager components are healthy, agents are connected, or alerts are being generated.

## Workstreams to document

- [x] Record Wazuh manager service status (owner-reported).
- [ ] Capture the installed Wazuh manager version.
- [ ] Enumerate registered agents and their current status using the manager CLI.
- [ ] Document Windows 10 agent connectivity and endpoint log collection.
- [ ] Document the Sysmon event source and one redacted example detection.
- [ ] Document the previously discussed ATT&CK **T1059.001 (PowerShell)** custom detection rule, including the exact rule, test event and alert evidence, when collected.
- [ ] Document a repeatable alert investigation with timestamps and redacted supporting evidence.

## Next read-only evidence collection

On the Ubuntu Wazuh VM:

```bash
sudo /var/ossec/bin/agent_control -l
```

Capture the agent ID, displayed endpoint name and connection status; redact addresses, keys and unrelated personally identifying information before committing screenshots. The command lists registered Wazuh agents and does not change agent state.

## Security and evidence handling

Do not commit authentication credentials, agent registration keys, certificates/private keys, raw endpoint logs containing personal information or full configuration exports. Lab IPs are included here for private inventory; review them before publishing the repository publicly.

*Last updated: 7 November 2026.*
