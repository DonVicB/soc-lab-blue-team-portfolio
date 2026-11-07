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

## Next read-only collection

Inspect the five most recent Sysmon **Event ID 1** process-creation records without printing full command-line contents or other sensitive message fields:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; Id=1} -MaxEvents 5 |
  Select-Object TimeCreated, Id, ProviderName
```

This is a basic event-source verification; the results must be reviewed before any screenshots or unredacted logs are shared or committed.

## Next portfolio milestones

- [x] Confirm Sysmon Operational log exists and is enabled.
- [x] Record local log's total record count from the screenshot.
- [ ] Confirm recent Sysmon Event ID 1 process-creation records.
- [ ] Validate Windows Wazuh agent connectivity and log collection.
- [ ] Correlate a controlled, redacted Sysmon process event with Wazuh rule `100101` or document why it does not fire.

See [Wazuh SIEM](../05-wazuh-siem/README.md) and [PowerShell detection rule analysis](../09-detection-rules/wazuh-powershell-t1059-001.md).

*Evidence recorded: 7 November 2026. Private training lab.*
