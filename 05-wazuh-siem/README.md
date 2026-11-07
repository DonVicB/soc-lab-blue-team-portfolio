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
- [x] Document Windows 10 local Sysmon Operational channel enabled with 33,076 records (7 November screenshot); [endpoint verification](../04-windows-endpoint-security/README.md).
- [ ] Record recent Sysmon Event ID 1 metadata and then correlate a redacted sample with Wazuh.
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

## Windows 10 local Sysmon confirmation — 7 November 2026

An Administrator PowerShell screenshot from `W.10.Pro_Soc-Lab Client` shows that `Get-WinEvent -ListLog 'Microsoft-Windows-Sysmon/Operational' | Select-Object LogName, RecordCount, IsEnabled` returned **`IsEnabled=True`** with **`RecordCount=33076`**. This verifies a populated local Sysmon event channel, **not** Wazuh ingestion, a specific Event ID 1 event or a fired PowerShell detection. Agent ID `001` was disconnected at the last Wazuh manager check.

See the [Windows endpoint security evidence](../04-windows-endpoint-security/README.md).

## Windows 10 Sysmon Event ID 1 confirmation — 7 November 2026

Windows Administrator PowerShell displayed five recent Sysmon Operational process-creation records with `Event ID=1`, all from provider `Microsoft-Windows-Sysmon`, with timestamps ranging from **14:58:42 to 14:59:37** (guest local time). The log was already shown as enabled with 33,076 records of all event types. This provides **source-side event evidence**, but the last Wazuh inventory showed agent `001` disconnected and no Wazuh rule `100101` alert has been verified.

See the [Windows endpoint Sysmon evidence](../04-windows-endpoint-security/README.md).

## Windows 10 Wazuh agent service verification — 7 November 2026

The Windows 10 guest PowerShell screenshot verifies **`WazuhSvc` (DisplayName `Wazuh`) is Running and has StartType Automatic**. This local service state does **not** prove communication with the Wazuh manager. Agent `001` (`DonW10`) had been reported **Disconnected** in an earlier manager-side inventory; a current manager-side status has not been checked.

See the [Windows endpoint evidence](../04-windows-endpoint-security/README.md). Do not characterize Wazuh's live Windows telemetry as operational until agent registration/connection and event arrival are checked.

## Windows 10 to Wazuh TCP connectivity — 7 November 2026

A Windows PowerShell screenshot confirms `Test-NetConnection -ComputerName 10.10.10.20 -Port 1514` returned **`TcpTestSucceeded=True`** using interface `soc nw` and source address **`10.10.10.110`**.

This is evidence of a reachable **TCP listener on Wazuh server port 1514**, not proof of authenticated Wazuh agent enrollment, current manager connection, Sysmon ingestion or a working alert rule. The latest manager-side recheck at about 19:23 AEDT confirms agent `001` (`DonW10`) remains `Disconnected`, while agent `002` (`DESKTOP-2MNIK0U`) is now `Active`; the Windows 10 VM hostname was subsequently observed as `DESKTOP-2MNIK0U`, matching the active registration; agent ID and actual event ingestion remain to be corroborated.

**Inventory difference:** The October 24 screenshot recorded Windows internal IPv4 `10.10.10.11`; this November 7 TCP test used `10.10.10.110`. The discrepancy is captured rather than silently overwriting the original VM inventory. Additional `Get-NetIPAddress` evidence would be needed to know whether the address was replaced or added.

See the [Windows endpoint evidence and test output](../04-windows-endpoint-security/README.md).

## Next evidence collection

Agent `002` details are now screenshot-verified. Next, use the **read-only current alert-file count** documented in Step 5.13 to look for evidence of alerts attributed to this agent, keeping those results distinct from Sysmon ingestion.



## Step 5.11 — Manager-side agent recheck (7 November 2026, approximately 19:23 AEDT)

**Evidence:** A new Ubuntu Wazuh VM screenshot includes the output of:

```bash
sudo /var/ossec/bin/agent_control -l
```

| ID | Registered name | Manager-reported status |
| --- | --- | --- |
| `000` | `wazuh-server` | **Active/Local** |
| `001` | `DonW10` | **Disconnected** |
| `002` | `DESKTOP-2MNIK0U` | **Active** |

**Change from earlier on 7 November:** Agent `002` has transitioned from **Disconnected to Active**, so **1 of 2 non-manager registered agents** is currently connected. Agent `001` remains Disconnected. The local manager entry `000` is not counted as a Windows endpoint.

The Windows 10 VM's own `WazuhSvc` service had already been shown running (Automatic), and a PowerShell TCP probe reached `10.10.10.20:1514`. Those results do not identify which registered agent corresponds to the current Windows 10 VM; `DonW10` may be an older or differently named registration. **Do not infer that the tested Windows VM is disconnected or that its Sysmon data is being ingested until the guest's identity is matched to agent `001` or `002`.**

**Next read-only identity check:** On the actual Windows 10 SOC client, run `hostname`, compare the returned hostname with registered names, then correlate with the Wazuh agent configuration if the names differ. No agent IDs, registrations or networking settings were changed in this documentation step.

## Step 5.12 — Windows hostname matched to active Wazuh agent (7 November 2026, approximately 19:26 AEDT)

**Evidence:** Screenshot from the selected VMware `W.10.Pro_Soc-Lab Client` guest in Administrator PowerShell:

```powershell
hostname
```

**Returned:** `DESKTOP-2MNIK0U`.

This is an **exact string match** with the manager's agent ID `002` registration, which was shown as **Active** at approximately 19:23 AEDT. The separate `DonW10` registration, ID `001`, remained Disconnected.

**Interpretation:** The selected Windows 10 VM is very likely the endpoint represented by active registration `002`, rather than registration `001`. The matching hostname is strong correlating evidence but does not independently expose the local agent's registration ID, authenticate that registration, or establish whether Wazuh is ingesting Sysmon Event ID 1 from it. Do not delete or re-enroll `DonW10` based on this name match alone.

**Next evidence:** On Ubuntu Wazuh, run the **read-only** command `sudo /var/ossec/bin/agent_control -i 002` to review the active registration's manager-side details. A later alert query is required to demonstrate real endpoint telemetry.

## Step 5.13 — Agent `002` details corroborated (7 November 2026, approximately 21:03 AEDT)

**Evidence:** Ubuntu Wazuh manager terminal screenshot from:

```bash
sudo /var/ossec/bin/agent_control -i 002
```

| Property | Manager-reported value |
| --- | --- |
| Agent ID | `002` |
| Agent Name | `DESKTOP-2MNIK0U` |
| Registered IP criterion | `any` |
| Status | **Active** |
| Operating system | **Microsoft Windows 10 Pro** |
| Agent client version | **Wazuh v4.14.9** |
| Syscheck | Start and completion timestamps present (7 November 2026) |

The manager-side agent name matches the Windows 10 VM's screenshot-verified `hostname` result. The manager also reports a Windows 10 Pro agent client, consistent with the guest. Combined, these observations **strongly corroborate that agent `002` represents the current Windows 10 lab VM**. The earlier `DonW10` agent ID `001` remains a separate disconnected registration. The `any` registered IP setting does **not** disclose or prove a particular current endpoint IP.

Configuration and shared-file hashes shown in the console are intentionally **not** transcribed to the portfolio. The screenshot's syscheck timestamps indicate integrity-check activity was recorded; they do not independently verify that Windows Sysmon Event ID 1 telemetry reached the manager.

**Status:** Manager-side connection and identity corroborated, **Sysmon event ingestion and custom PowerShell rule `100101` still not verified**. Do not state that a full detection pipeline has been demonstrated.

## Next read-only verification — incoming alert records for agent 002

On the Ubuntu Wazuh manager:

```bash
sudo grep -c '"id":"002"' /var/ossec/logs/alerts/alerts.json
```

The result is the number of lines with this literal agent-ID fragment in the **current JSON alert file**. A positive count would support that alerts associated with `002` exist (not necessarily Sysmon alerts or real-time ingestion), while `0` could also be explained by retention, rule thresholds, a different JSON formatting style, or logs residing elsewhere. Do not post raw alerts without redaction.

## Security and evidence handling

Do not commit authentication credentials, agent registration keys, certificates/private keys, raw endpoint logs containing personal information or full configuration exports. Lab IPs are included here for private inventory; review them before publishing the repository publicly.

*Last updated: 7 November 2026.*
