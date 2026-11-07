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
| Wazuh version | `v4.14.9` — screenshot-verified `wazuh-control info`, 7 November 2026 |
| Wazuh revision | `rc2` — screenshot-verified |
| Wazuh type | `server` — screenshot-verified |

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
- [x] Capture the installed Wazuh manager version and revision — `v4.14.9`, `rc2`, `server` (7 November screenshot).
- [x] Enumerate registered agents and their current status using the manager CLI (7 November 2026; two endpoints disconnected).
- [ ] Document Windows 10 agent connectivity and endpoint log collection (current agent status: disconnected; do not claim live collection).
- [ ] Document the Sysmon event source and one redacted example detection.
- [x] Document screenshot-verified identity and intended conditions of [custom PowerShell detection rule `100101`](../09-detection-rules/wazuh-powershell-t1059-001.md) (7 November 2026).
- [ ] Validate rule loading, capture the complete regex and correlate a representative Sysmon event with an actual Wazuh alert.
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

## Version verification — 7 November 2026

**Evidence:** A screenshot of the Wazuh server terminal shows:

```bash
sudo /var/ossec/bin/wazuh-control info
```

with the reported output:

```text
WAZUH_VERSION="v4.14.9"
WAZUH_REVISION="rc2"
WAZUH_TYPE="server"
```

These are the exact identifiers displayed by the local installation. The revision field is transcribed as `rc2` without inferring whether it is a release channel or build status. This is a **version inventory**, not a vulnerability, service-health or upgrade assessment. The same screenshot again displays `wazuh-manager` as active and Windows agent IDs `001` and `002` as disconnected.

## Custom PowerShell detection rule — 7 November 2026

A screenshot of the corrected grep command against `/var/ossec/etc/rules/local_rules.xml` confirms custom **Wazuh rule `100101`**, level `10`, group `Windows,sysmon,powershell,mitre,`, inheriting `sysmon_event1`, with PCRE2 checks for `win.eventdata.image` and `win.eventdata.commandLine`. The rule's description refers to suspicious PowerShell command-line activity and maps it to MITRE **T1059.001**. The long command-line regex is terminal-wrapped, so the screenshot alone is not treated as a complete machine-readable copy.

**Evidence classification:** Rule exists on disk; alert firing and ongoing endpoint ingestion **not yet verified**. Both Windows agents were disconnected when last checked.

See the [dedicated rule analysis and validation checklist](../09-detection-rules/wazuh-powershell-t1059-001.md).

## Alert search result — 7 November 2026

The lab owner reports that the current-file search for custom rule `100101`:

```bash
sudo grep -c '"id":"100101"' /var/ossec/logs/alerts/alerts.json
```

returned **`0`**. This means no lines in the current JSON alert file matched the exact text. It is **not** proof that the custom rule has never fired, is broken, or that older/rotated alert files contain no matching alerts. The manager was active but both registered Windows agents were disconnected during the earlier service inventory.

See [rule `100101` evidence and validation](../09-detection-rules/wazuh-powershell-t1059-001.md) for limitations.

## Synthetic Sysmon JSON Logtest — 7 November 2026

The owner supplied a screenshot of `wazuh-logtest v4.14.9` after entering a synthetic Windows Sysmon process-creation JSON event with `powershell.exe -enc TEST`. **Phase 1** completed pre-decoding; **Phase 2** decoded the event as `json` and extracted the PowerShell image, command line and event metadata. **No Phase 3 matching output was shown**, and custom rule `100101` did **not** trigger in this test.

Because the rule depends on `<if_group>sysmon_event1</if_group>`, this manually submitted JSON sample may not have reached the normal Windows EventChannel parent rule group. This is a **test-format/parent-rule limitation**, not proof that the deployed rule is defective. The manager's two Windows endpoint agents were last seen disconnected.

See [detection rule test analysis](../09-detection-rules/wazuh-powershell-t1059-001.md) for the screenshot observations and limitations.

## Next evidence collection

On the Windows 10 lab endpoint, run a read-only check of the Sysmon event log:

```powershell
Get-WinEvent -ListLog 'Microsoft-Windows-Sysmon/Operational' | Select-Object LogName, RecordCount, IsEnabled
```

This will establish whether the source log is enabled and contains records. It does not by itself prove Wazuh has ingested those events.


## Security and evidence handling

Do not commit authentication credentials, agent registration keys, certificates/private keys, raw endpoint logs containing personal information or full configuration exports. Lab IPs are included here for private inventory; review them before publishing the repository publicly.

*Last updated: 7 November 2026.*
