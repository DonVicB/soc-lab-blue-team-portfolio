# Case study — Defender detects SandCat-labelled Trojan during SOC lab exercise

**Status:** Investigation in progress — **detection and named quarantine actions verified in native endpoint event records**, action outcome and complete CALDERA-to-Defender correlation still pending.

## Analyst question

Did Microsoft Defender detect and quarantine the Sandcat agent used for an authorised MITRE CALDERA adversary-emulation exercise, and can the activity be reconstructed from endpoint telemetry?

## Environment and scope

- **Lab:** Private VMware SOC training environment.
- **Endpoint:** Windows 10 Pro VM `DESKTOP-2MNIK0U`.
- **Relevant tooling:** Microsoft Defender Antivirus, Sysmon, Splunk (index `main`), MITRE CALDERA/Sandcat.
- **Authorisation:** Controlled testing in the owner's isolated lab, not a production or third-party system.
- **Investigation date:** 8 November 2026; Windows event records below were created **19 October 2026**.

## Detection evidence — Microsoft Defender Operational log

On the Windows guest, an Administrator PowerShell read-only query:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Windows Defender/Operational'; Id=1116,1117; StartTime=[datetime]'2026-10-19 00:00:00'; EndTime=[datetime]'2026-10-19 01:00:00'} |
  Sort-Object TimeCreated
```

was followed by targeted parsing of the `Threat Name`, `Severity Name`, `Category Name`, and `Action Name` XML fields. Screenshot evidence confirmed:

| Windows guest local time | Defender ID | Threat classification | Recorded action |
| --- | --- | --- | --- |
| 2026-10-19 00:21:06 | `1116` — detection | `Trojan:Win64/SandCat.RTS!MTB`, **Severe**, **Trojan** | Not Applicable (detection event) |
| 2026-10-19 00:21:22 | `1117` — action | Same threat label | **Quarantine** |
| 2026-10-19 00:25:12 | `1116` — detection | Same threat label | Not Applicable (detection event) |
| 2026-10-19 00:25:23 | `1117` — action | Same threat label | **Quarantine** |

**Observed:** Two `1116` detections and two `1117` actions explicitly named **Quarantine** for `Trojan:Win64/SandCat.RTS!MTB`. The classification and action fields come from **native Microsoft Defender endpoint records**, **not from Splunk**.

**Evidence limits:** The screenshot did not show error codes, action status, affected paths, or file hashes. Thus, the specific quarantine action is **recorded**, but successful completion and exact artifact identity remain to be verified. The threat label aligns with the earlier owner's CALDERA Sandcat test, but no direct correlation of binary hash, process tree, CALDERA operation time, or affected file has yet been shown. Do not claim that Sandcat executed successfully or that Defender definitely completed remediation.

## Earlier Splunk hunting — what did and did not work

1. Splunk `main` receives Windows/Sysmon events from `DESKTOP-2MNIK0U` (e.g., 5,132 Sysmon events in one 24-hour snapshot).
2. A Year-to-date `sandcat OR caldera` keyword query returned **0** hits; this does not disprove the historical test.
3. A broader Defender/quarantine keyword search returned **1,408** hits, but did not independently identify actual Defender actions.
4. Restricting to `quarantine OR quarantined` found two Sysmon Event ID `1` process records. Both were `brave.exe` with `quarantine.mojom.Quarantine` service text — benign-looking **browser utility-process false leads**, not Defender quarantine evidence.
5. Pivoting away from broad keywords to the **native Microsoft Defender Operational channel** finally exposed the threat-specific `1116`/`1117` sequence above.

**Practical SOC lesson:** A keyword match is a lead, not proof. Confirm the event provider, event ID, threat name and action metadata before attributing security outcomes.

## Next read-only validation

Inspect only selected action-result XML fields on the Windows 10 VM for the two `1117` entries (see [Step 6.12 in Splunk documentation](../06-splunk-siem/README.md)). Then determine whether the Defender Operational channel or another reliable threat/event source is searchable in Splunk, and correlate evidence with the documented CALDERA exercise.

## Open evidence and response questions

- [x] Confirm Windows Defender detected a threat labelled `Trojan:Win64/SandCat.RTS!MTB`.
- [x] Confirm Defender recorded two action events labelled **Quarantine**.
- [ ] Check action outcome/error codes and whether remediation completed.
- [ ] Relate the detection timestamps to CALDERA operation details and controlled process execution.
- [ ] Establish which relevant data is present or absent in Splunk and Wazuh.
- [ ] Produce a final event timeline, analysis and defensible outcome/response recommendations.
- [ ] Create redacted evidence exhibits; do **not** commit raw endpoint screenshots that expose account names, paths, hashes or unrelated personal browser data.

**Current case disposition:** Detection plus recorded quarantine actions **substantiated**. Final remediation outcome, source attribution and SIEM-level correlation **not yet established**.
