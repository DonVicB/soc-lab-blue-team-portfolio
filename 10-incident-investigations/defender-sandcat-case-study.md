# Case study — Defender detects SandCat-labelled Trojan during SOC lab exercise

**Status:** Investigation in progress — **SandCat threat detections, quarantine actions, and successful completion status verified in native Microsoft Defender event records**. CALDERA-to-Defender process/artifact correlation and SIEM coverage still pending.

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

**Evidence limits (updated at Step 6.12):** A later screenshot provides error codes and descriptions for both quarantine actions, confirming that Defender **reported successful completion** of each action. Affected paths, file hashes, process ancestry, exact CALDERA operation timestamps, and the agent's runtime behavior remain unverified. Do not infer that Sandcat successfully executed or that all potentially associated artifacts were remediated.

## Earlier Splunk hunting — what did and did not work

1. Splunk `main` receives Windows/Sysmon events from `DESKTOP-2MNIK0U` (e.g., 5,132 Sysmon events in one 24-hour snapshot).
2. A Year-to-date `sandcat OR caldera` keyword query returned **0** hits; this does not disprove the historical test.
3. A broader Defender/quarantine keyword search returned **1,408** hits, but did not independently identify actual Defender actions.
4. Restricting to `quarantine OR quarantined` found two Sysmon Event ID `1` process records. Both were `brave.exe` with `quarantine.mojom.Quarantine` service text — benign-looking **browser utility-process false leads**, not Defender quarantine evidence.
5. Pivoting away from broad keywords to the **native Microsoft Defender Operational channel** finally exposed the threat-specific `1116`/`1117` sequence above.

**Practical SOC lesson:** A keyword match is a lead, not proof. Confirm the event provider, event ID, threat name and action metadata before attributing security outcomes.

## Step 6.12 — Defender quarantine completion confirmed (8 November 2026)

**Evidence:** Administrator PowerShell screenshot from `DESKTOP-2MNIK0U` after querying Defender Operational `1117` events in the 19 October 2026 00:00–01:00 local window and extracting `Threat Name`, `Action Name`, `Error Code`, and `Error Description`. Both records report:

| Event time (guest local) | Event ID | Threat | Action name | Error code | Error description |
| --- | --- | --- | --- | --- | --- |
| 2026-10-19 00:21:22 | `1117` | `Trojan:Win64/SandCat.RTS!MTB` | **Quarantine** | `0x00000000` | **The operation completed successfully.** |
| 2026-10-19 00:25:23 | `1117` | `Trojan:Win64/SandCat.RTS!MTB` | **Quarantine** | `0x00000000` | **The operation completed successfully.** |

**Analyst conclusion:** Microsoft's Defender Operational events demonstrate two detections, two quarantine actions and **successful completion reported for both actions**. This is stronger than merely observing event ID `1117`, but does not yet prove the Sandcat agent ran, identify the precise quarantined file(s), or show any other containment steps.

## Next read-only validation — Splunk/Sysmon time correlation

In Splunk Search & Reporting, select **All time**, then query the historical hour spanning the Defender actions to check that Sysmon data exists in the corresponding time window:

```spl
index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" earliest="10/18/2026:23:45:00" latest="10/19/2026:00:45:00"
| stats count min(_time) as first_seen max(_time) as last_seen by host
| convert ctime(first_seen) ctime(last_seen)
```

This is a **coverage check**, not a direct CALDERA attribution. Compare the result with Defender event times only after checking Splunk time-zone alignment. If no events exist, inspect collection history before treating it as proof of no process activity.

## Open evidence and response questions

- [x] Confirm Windows Defender detected a threat labelled `Trojan:Win64/SandCat.RTS!MTB`.
- [x] Confirm Defender recorded two action events labelled **Quarantine**.
- [x] Confirm both Defender quarantine action events report `0x00000000` and **The operation completed successfully**.
- [ ] Relate the detection timestamps to CALDERA operation details and controlled process execution.
- [ ] Establish which relevant data is present or absent in Splunk and Wazuh.
- [ ] Produce a final event timeline, analysis and defensible outcome/response recommendations.
- [ ] Create redacted evidence exhibits; do **not** commit raw endpoint screenshots that expose account names, paths, hashes or unrelated personal browser data.

**Current case disposition:** Two SandCat-labelled threat detections and two **successfully completed quarantine actions reported by Defender**. Exact quarantined artifact(s), CALDERA process execution, and SIEM-level correlation **not yet established**.
