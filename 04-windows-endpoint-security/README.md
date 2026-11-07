# Windows 10 endpoint security — Sysmon evidence

> **Status:** Local Sysmon event channel is present, enabled and populated. Wazuh ingestion and current endpoint-agent connectivity have **not** been verified.

## Environment

- VMware VM: `W.10.Pro_Soc-Lab Client`.
- Guest hostname historically referred to as `DonW10`; the latest command output does not show the hostname.
- Internal lab IPv4: `10.10.10.11/24`, recorded from the 24 October 2026 network inventory.
- Security telemetry: Windows Sysmon Operational log.

## Step 5.7 — Local Sysmon event log (7 November 2026)

**Evidence:** Screenshot of Administrator PowerShell in the selected Windows 10 VM, with the read-only command:

```powershell
Get-WinEvent -ListLog 'Microsoft-Windows-Sysmon/Operational' | Select-Object LogName, RecordCount, IsEnabled
```

| Field | Screenshot-verified result |
| --- | --- |
| LogName | `Microsoft-Windows-Sysmon/Operational` |
| RecordCount | `33076` at time of screenshot |
| IsEnabled | `True` |

**Interpretation:** The Sysmon Operational log exists, is enabled, and contains 33,076 records at capture time. The number is a **total log record count**, not a count of process-creation (Event ID 1) records and not proof of any particular detection. No change was made to Sysmon during this check.

**Known separate limitation:** The Wazuh manager previously reported agent ID `001` (`DonW10`) disconnected, as well as one other Windows endpoint. A healthy local log does **not** establish that Wazuh is receiving those records. The custom Wazuh PowerShell rule `100101` (ATT&CK `T1059.001`) was observed on disk, but has not generated a verified match in the evidence collected so far.

## Step 5.8 — Sysmon Event ID 1 process-creation evidence (7 November 2026)

**Evidence:** Administrator PowerShell screenshot from the selected Windows 10 VM running:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; Id=1} -MaxEvents 5 |
  Select-Object TimeCreated, Id, ProviderName
```

| TimeCreated (Windows guest local time) | Event ID | ProviderName |
| --- | --- | --- |
| 2026-11-07 14:59:37 | `1` | `Microsoft-Windows-Sysmon` |
| 2026-11-07 14:59:11 | `1` | `Microsoft-Windows-Sysmon` |
| 2026-11-07 14:59:01 | `1` | `Microsoft-Windows-Sysmon` |
| 2026-11-07 14:58:50 | `1` | `Microsoft-Windows-Sysmon` |
| 2026-11-07 14:58:42 | `1` | `Microsoft-Windows-Sysmon` |

**Assessment:** This confirms the Windows endpoint recorded five recent Sysmon process-creation events (Event ID `1`) and the latest visible record is from `14:59:37`. The earlier `33,076` count is **all** Sysmon Operational records, not an Event ID `1` count. This screenshot contains metadata only: it does **not** show the process image, full command line, whether an event is suspicious, whether data was forwarded to Wazuh, or whether custom rule `100101` generated an alert.

The same screenshot has unrelated ping results, but these are outside the present Sysmon evidence milestone and were not interpreted as verification of Wazuh transport.

## Step 5.9 — Windows Wazuh agent service status (7 November 2026)

**Evidence:** Administrator PowerShell screenshot from the selected Windows 10 lab VM after running:

```powershell
Get-Service | Where-Object { $_.Name -like '*Wazuh*' -or $_.DisplayName -like '*Wazuh*' } |
  Select-Object Name, DisplayName, Status, StartType
```

| Property | Screenshot-verified result |
| --- | --- |
| Service Name | `WazuhSvc` |
| DisplayName | `Wazuh` |
| Status | **Running** |
| StartType | **Automatic** |

**Assessment:** The Windows Wazuh agent service is installed, running, and set to start automatically. This is **local Windows service state**, not confirmation of a session with the Wazuh manager or successful Sysmon event forwarding. At the earlier manager-side check on 7 November, registered agent `001` (`DonW10`) was **Disconnected**. The checks occurred at different times; the manager has **not yet been rechecked** after this service screenshot.

The screenshot also contains prior ICMP tests, but this step only documents the Windows service state to avoid unrelated network troubleshooting.

## Step 5.10 — TCP connectivity to Wazuh server (7 November 2026)

**Evidence:** Windows 10 Administrator PowerShell screenshot of the read-only command:

```powershell
Test-NetConnection -ComputerName 10.10.10.20 -Port 1514
```

| Output field | Screenshot-verified result |
| --- | --- |
| ComputerName / RemoteAddress | `10.10.10.20` |
| RemotePort | `1514` |
| InterfaceAlias | `soc nw` |
| SourceAddress | **`10.10.10.110`** |
| TcpTestSucceeded | **`True`** |

**Interpretation:** The Windows endpoint established a TCP connection to the Wazuh server IP on port `1514` at capture time. This proves a reachable **TCP listener**, but does **not** prove the Wazuh agent is connected or authenticated with the manager, that a Wazuh process owns the listener, or that the Sysmon Operational log is being ingested.

**Address discrepancy to reconcile:** The October 24 screenshot of Windows `ipconfig /all` recorded internal `10.10.10.11/24` (Ethernet1). This November 7 connection check explicitly used `10.10.10.110` as its **source** on interface alias `soc nw`. This is an observed change/difference, but the evidence does not establish whether the original IP was replaced, additional addresses were assigned, or the interface configuration changed. Do **not** overwrite the historical inventory without an updated `Get-NetIPAddress` result. No network settings were changed in this step.

## Step 5.11 — Updated manager-side agent states (7 November 2026, approximately 19:23 AEDT)

The manager's latest `sudo /var/ossec/bin/agent_control -l` screenshot reports:

- `001` — `DonW10`: **Disconnected**
- `002` — `DESKTOP-2MNIK0U`: **Active**

The earlier manager-side check had both Windows agents Disconnected. **One of two registered Windows agents is now Active.** This is not yet proof which registration belongs to this Windows 10 VM. `WazuhSvc` is running on the Windows VM and TCP port `1514` on `10.10.10.20` was reachable, but neither test establishes the guest's Wazuh agent ID or actual ingestion.

## Step 5.12 — Guest hostname verified (7 November 2026, approximately 19:26 AEDT)

The Windows 10 VM screenshot shows:

```powershell
hostname
```

**Returned:** `DESKTOP-2MNIK0U`. This **exactly matches** the name of Wazuh manager registration `002`, which was **Active** in the 19:23 AEDT inventory. Registration `001` (`DonW10`) was Disconnected.

**Evidence assessment:** This is strong evidence that the current VM corresponds to agent `002`, and the previous assumption that the disconnected `DonW10` registration necessarily represents the current guest should not be treated as established. Matching hostnames do not independently confirm the local agent ID, agent authorization or incoming Sysmon event records. Manager-side registration details and eventual log ingestion remain to verify.

**Next read-only command (Ubuntu Wazuh manager):**

```bash
sudo /var/ossec/bin/agent_control -i 002
```

Do not show a Windows agent `client.keys` file or commit registration secrets. No Windows settings were changed.







## Step 5.13 — Manager-side Windows agent identity corroborated (7 November 2026, around 21:03 AEDT)

A screenshot from the Ubuntu Wazuh manager running `sudo /var/ossec/bin/agent_control -i 002` lists:

| Item | Screenshot evidence |
| --- | --- |
| Agent ID | `002` |
| Agent name | `DESKTOP-2MNIK0U` (exactly matches Windows `hostname`) |
| Status | **Active** |
| OS | **Microsoft Windows 10 Pro** |
| Client version | **Wazuh v4.14.9** |
| Registration IP | `any` |
| Syscheck status | Start and completion times present in manager output |

This strengthens the identification of the current Windows 10 lab VM as the active Wazuh agent `002`. The `DonW10` registration `001` remains a separate disconnected entry; the screenshot does not explain whether it is stale or belongs to another endpoint. The configuration hashes visible in the terminal screenshot were deliberately not reproduced in the GitHub documentation.

**Verification boundary:** Agent identity and Active registration are now supported by the screenshots, but **receipt of Sysmon Event ID 1 or firing of custom rule `100101` remains unverified**. An agent can show Active without proving that this particular Windows log source has been configured for collection.

**Next read-only command (Ubuntu manager):**

```bash
sudo grep -c '"id":"002"' /var/ossec/logs/alerts/alerts.json
```

Count potential alerts for agent ID `002` in the current JSON alerts file; zero or nonzero cannot alone establish whether Sysmon is forwarded.

## Next portfolio milestones

- [x] Confirm Sysmon Operational log exists and is enabled.
- [x] Record local log's total record count from the screenshot.
- [x] Confirm five recent Sysmon Event ID 1 process-creation records (7 November 2026; screenshot-verified).
- [x] Confirm local Windows Wazuh agent service `WazuhSvc` is running and Automatic (7 November screenshot).
- [x] Match Windows guest hostname `DESKTOP-2MNIK0U` to the manager's Active agent `002` by name (7 November screenshot).
- [x] Corroborate Windows VM identity and active Wazuh registration `002` using manager-side OS and version details (7 November 2026 screenshot).
- [ ] Verify actual Sysmon event ingestion for agent `002` (manager Active alone does not establish it).
- [ ] Correlate a controlled, redacted Sysmon process event with Wazuh rule `100101` or document why it does not fire.

See [Wazuh SIEM](../05-wazuh-siem/README.md) and [PowerShell detection rule analysis](../09-detection-rules/wazuh-powershell-t1059-001.md).

*Evidence recorded: 7 November 2026. Private training lab.*
