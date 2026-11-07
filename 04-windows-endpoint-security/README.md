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

## Next read-only evidence collection — Windows Wazuh agent service

In the Windows 10 PowerShell terminal, check the local service status:

```powershell
Get-Service | Where-Object { $_.Name -like '*Wazuh*' -or $_.DisplayName -like '*Wazuh*' } |
  Select-Object Name, DisplayName, Status, StartType
```

This does not change any services. An empty result would indicate no matching service name was returned, not by itself prove that a Wazuh agent is absent. Correlate it later with the server agent list.

## Next portfolio milestones

- [x] Confirm Sysmon Operational log exists and is enabled.
- [x] Record local log's total record count from the screenshot.
- [x] Confirm five recent Sysmon Event ID 1 process-creation records (7 November 2026; screenshot-verified).
- [ ] Validate Windows Wazuh agent connectivity and log collection.
- [ ] Correlate a controlled, redacted Sysmon process event with Wazuh rule `100101` or document why it does not fire.

See [Wazuh SIEM](../05-wazuh-siem/README.md) and [PowerShell detection rule analysis](../09-detection-rules/wazuh-powershell-t1059-001.md).

*Evidence recorded: 7 November 2026. Private training lab.*
