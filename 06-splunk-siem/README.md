# Splunk SIEM — Search and Investigation Evidence

> **Portfolio status:** Splunk installation, daemon, web search, indexed Windows/Sysmon data, and preliminary security keyword searches verified from screenshots. A specific Defender quarantine event and a completed CALDERA/Sandcat investigation remain **unverified**. Training lab, not production.

## Purpose

Use Splunk to search and correlate security telemetry, investigate endpoint detections, and produce clear SOC analyst case studies with **evidence, timeline, findings, and lessons learned**.

## VMware deployment baseline

| Property | Evidence / status |
| --- | --- |
| VMware guest display name | `Splunk-SIEM` |
| Guest environment | Ubuntu Server 26.04.1 LTS (login banner screenshot verified, 7 November 2026) |
| Internal SOC network | `ens33: 10.10.10.69/24`, within host-only VMnet1 subnet `10.10.10.0/24` |
| NAT network | `ens34: 192.168.111.135/24`, VMnet8 network `192.168.111.0/24` |
| Default IPv4 route | `192.168.111.2` via `ens34` |
| Resolver seen in screenshot | `192.168.111.2` on `ens34` |
| Splunk product | **Splunk Enterprise Free** — installed and Splunk Web previously accessible according to operator's earlier lab setup report |
| Current installed version | **Splunk 10.6.1.1**, build `b2ae2962645c` (CLI screenshot verified, 7 November 2026) |
| Splunk service (7 November 2026) | **Running**: `splunkd` PID `5732`; Splunk helper processes running (screenshot verified) |
| Indexed Windows data | **Verified in `main`**: 29,337 events across seven sourcetypes in last 24h, host `DESKTOP-2MNIK0U` (8 Nov screenshot) |

The guest addresses are sourced from the earlier `ip -br -4 addr`, `ip -4 route`, and `resolvectl status` screenshot; these are a dated network snapshot, not a fresh connectivity test.

## Evidence collection roadmap

- [x] Record the confirmed VM network layout from the existing screenshots.
- [x] Record the **installed Splunk version**, `10.6.1.1` (build `b2ae2962645c`) from the live Splunk VM.
- [x] Confirm `splunkd` and helper processes are running using the Splunk CLI.
- [x] Verify Splunk Web Search & Reporting in the present session.
- [x] Confirm stored data in `main` and seven Windows/Perfmon/Sysmon sourcetypes without exposing raw events.
- [x] Capture aggregate Splunk keyword-search evidence (broad Defender/CALDERA match and narrow zero-result check; no incident conclusions yet).
- [ ] Complete a structured incident case study: Microsoft Defender detection/quarantine during a controlled CALDERA Sandcat exercise, using only evidence actually observed.
- [ ] Explain the investigative decision process and map validated activity to MITRE ATT&CK where supported.

## Step 6.1 — Installed Splunk version (7 November 2026)

**Evidence:** Screenshot of the `Splunk-SIEM` Ubuntu VM terminal showing:

```bash
sudo /opt/splunk/bin/splunk version
```

Output:

```text
Splunk 10.6.1.1 (build b2ae2962645c)
```

This demonstrates that the Splunk command-line executable is installed in the expected `/opt/splunk/bin/` location and reports version **10.6.1.1**, build `b2ae2962645c`. The login banner in the same screenshot identifies **Ubuntu 26.04.1 LTS**. The screenshot does **not** establish that the daemon or Splunk Web is running at the time of capture, whether an active license is Free or Enterprise, or whether security logs are currently searchable. A prior operator report describes a Splunk Enterprise Free installation and accessible web interface, but this has not yet been retested.

## Step 6.2 — Splunk service verified (7 November 2026)

**Evidence:** Screenshot of the `Splunk-SIEM` Ubuntu VM terminal running the read-only command:

```bash
sudo /opt/splunk/bin/splunk status
```

Output includes:

```text
splunkd is running (PID: 5732).
splunk helpers are running (PIDs: 6098 6160 6217 6320 7074 8278 8584 8586 8588 8655 8714).
```

**Assessment:** Splunk's daemon and helper processes were active when checked. The PID values are only a snapshot, not persistent identifiers. This verifies process state but not Splunk Web accessibility, search permissions, source configurations or actual security-event ingestion.

The same screenshot shows `sudo apt update` reporting temporary DNS resolution failures for Ubuntu package repositories. That is a **separate maintenance observation**, not a Splunk process failure, and is outside this portfolio milestone.

## Step 6.3 — Splunk Web and stored-index evidence (7 November 2026)

**Evidence:** Splunk Web **Search & Reporting** was accessible on the SOC lab instance at `http://10.10.10.69:8000`. The screenshot shows the actual search:

```spl
| eventcount summarize=false index=*
| search index=main
```

It returned one Statistics row for index `main`, server `splunk-siem`, count provider `local`, with **90,533** stored events at the time of collection.

**Interpretation:** This confirms Splunk Web search access and that the `main` index held data. This was **not** an all-index inventory: the query filtered to `main`. The `eventcount` count is not constrained to the selected UI time picker, and `provider=local` is count provenance, **not** the security event provider.

## Step 6.4 — Indexed telemetry by host and sourcetype (8 November 2026)

**Evidence:** Screenshot of Search & Reporting, time picker **Last 24 hours**, running:

```spl
index=main
| stats count by host sourcetype
| sort - count
```

The screenshot covers approximately **7 November 2026 06:00 to 8 November 2026 06:47** (Splunk search UI time range). All seven result rows identify host `DESKTOP-2MNIK0U`.

| Sourcetype | Matching event count |
| --- | ---: |
| `Perfmon:Network Interface` | 12,584 |
| `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` | 5,132 |
| `Perfmon:CPU Load` | 4,395 |
| `WinEventLog:Security` | 3,777 |
| `Perfmon:Available Memory` | 3,146 |
| `WinEventLog:System` | 163 |
| `WinEventLog:Application` | 140 |
| **Total** | **29,337** |

**Interpretation:** Splunk contains substantial recent events from this single lab endpoint, including Sysmon, Windows Security and other log sources. Dedicated Microsoft Defender Operational events are **not listed as a separate sourcetype** in this seven-row result. The counts reflect only the selected Last 24 hours and visible `main` data; they do not establish that Defender events are missing globally.

## Step 6.5 — Broad Defender/CALDERA keyword search (8 November 2026)

**Evidence:** Search & Reporting screenshot, time picker **All time**, running:

```spl
index=main (sandcat OR caldera OR quarantine OR "Windows Defender" OR "Microsoft Defender")
| stats count by host sourcetype
| sort - count
```

It returned **1,408** keyword-matching events associated with host `DESKTOP-2MNIK0U`:

| Sourcetype | Matching events |
| --- | ---: |
| `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` | 1,010 |
| `WinEventLog:Application` | 209 |
| `WinEventLog:Security` | 124 |
| `WinEventLog:System` | 65 |
| **Total** | **1,408** |

**Interpretation:** These are matches to **any** of the search terms (logical OR); the result does **not** prove Sandcat was executed, CALDERA was identified, or Defender quarantined a file. The source breakdown is useful for prioritizing investigation.

## Step 6.6 — Narrow Sandcat/CALDERA keyword search (8 November 2026)

**Evidence:** Screenshot of the following SPL query:

```spl
index=main (sandcat OR caldera)
| stats count min(_time) as first_seen max(_time) as last_seen by host sourcetype
| convert ctime(first_seen) ctime(last_seen)
| sort - count
```

The UI was set to **Year to date**, spanning **1 January 2026 to 8 November 2026 around 07:03 AEDT**. It returned **0 events** and no Statistics rows.

**Interpretation:** No events matched the **literal terms** `sandcat` or `caldera` in `main` for the selected date range. This is a *negative text-search result*, **not** evidence that the earlier controlled exercise did not occur, nor proof that no related telemetry exists. Relevant events may use an executable name, encoded command, generic process details or a Defender threat identifier without either word. The time picker was **not** All time, so older data was outside this search.

## Step 6.7 — Defender/quarantine keyword search with timestamps (8 November 2026)

**Evidence:** Screenshot of Splunk Search & Reporting, time picker **Year to date** (1 January 2026 to 8 November 2026 around 07:06 AEDT), executing:

```spl
index=main (quarantine OR "Windows Defender" OR "Microsoft Defender")
| stats count min(_time) as first_seen max(_time) as last_seen by host sourcetype
| convert ctime(first_seen) ctime(last_seen)
| sort - count
```

All returned rows show host `DESKTOP-2MNIK0U` in index `main`. Times are rendered by Splunk as displayed, and may reflect Splunk's configured display time zone.

| Sourcetype | Matched events | First seen (Splunk display) | Last seen (Splunk display) |
| --- | ---: | --- | --- |
| `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` | 1,010 | 2026-10-17 19:15:04.551 | 2026-11-07 22:18:35.864 |
| `WinEventLog:Application` | 209 | 2026-08-14 16:06:17.440 | 2026-11-07 19:01:42.985 |
| `WinEventLog:Security` | 124 | 2026-09-16 21:19:14.648 | 2026-10-18 18:19:55.026 |
| `WinEventLog:System` | 65 | 2026-08-14 16:48:24.278 | 2026-11-07 14:56:40.180 |
| **Total** | **1,408** | | |

**Analysis:** This count matches Step 6.5's broader OR query, which additionally included `sandcat` and `caldera`. The separate Step 6.6 `sandcat OR caldera` search returned **0** over the Year-to-date period. Therefore the keyword matches within this period can be attributed to one or more of `quarantine`, `Windows Defender`, or `Microsoft Defender`, **not** demonstrably to CALDERA/Sandcat names. This does **not** mean 1,408 actual Defender detections or quarantine actions occurred: broad text references to Defender may be benign. No dedicated Defender Operational sourcetype is listed in these results.

**Time-range nuance:** The user was asked to select All time, but this screenshot clearly shows **Year to date**. The reported event timestamps are within that range. Do not relabel the observed query as All time.

## Step 6.8 — Quarantine terminology narrowed to two Sysmon records (8 November 2026)

**Evidence:** Splunk Search & Reporting screenshot, selected time picker **Year to date** (1 January 2026 to 8 November 2026 around 07:12 AEDT), with this SPL:

```spl
index=main (quarantine OR quarantined)
| stats count min(_time) as first_seen max(_time) as last_seen by host sourcetype
| convert ctime(first_seen) ctime(last_seen)
| sort - count
```

The screenshot returns **2 matched events**, in one Statistics row:

| Property | Result |
| --- | --- |
| Host | `DESKTOP-2MNIK0U` |
| Index | `main` |
| Sourcetype | `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` |
| Matching events | **2** |
| First seen (Splunk display) | **2026-10-18 21:24:05.213** |
| Last seen (Splunk display) | **2026-11-05 22:32:19.528** |

**Analysis:** These two records contain terms matching `quarantine` or `quarantined`. They are Sysmon telemetry records, **not yet confirmed Microsoft Defender action events**. The appearance of the terms in a command line, file path, application output or another field could produce matches without an actual quarantine. The screenshot does not show an event ID, image, parent process, threat name or Defender action status. Times reflect the Splunk display and are not independently normalized to UTC.

**Time-range nuance:** The query was run **Year to date**, not the requested All time. No conclusion is drawn about data outside that selected range.

## Step 6.9 — Two quarantine keyword hits explained as Brave process creation (8 November 2026)

**Evidence:** Splunk Search & Reporting screenshot displaying the expanded events from:

```spl
index=main (quarantine OR quarantined) sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
```

The Splunk time picker remains **Year to date**. The `Events` tab shows **two** Sysmon XML events with the following screenshot-supported metadata:

| Splunk display time | Sysmon Event ID | Executable image | Matched text |
| --- | --- | --- | --- |
| **2026-10-18 21:24:05.213** | `1` — Process Create | `brave.exe` (Brave Browser) | `--utility-sub-type=quarantine.mojom.Quarantine` |
| **2026-11-05 22:32:19.528** | `1` — Process Create | `brave.exe` (Brave Browser) | `--utility-sub-type=quarantine.mojom.Quarantine` |

The XML identifies the **Microsoft-Windows-Sysmon** provider; the `Image`, `OriginalFileName` and `ParentImage` fields indicate Brave. The command line's `quarantine.mojom.Quarantine` component is a Chromium-family browser file-download/quarantine service name; see Chromium's [quarantine service interface](https://chromium.googlesource.com/chromium/src/+/HEAD/components/services/quarantine/public/mojom/quarantine.mojom). The screenshot shows **process execution**, not an antimalware detection or the result of quarantining a specific file.

**Analyst assessment:** These are **benign-looking browser utility process events** that matched a broad quarantine keyword search. Neither event establishes that Microsoft Defender detected, blocked or quarantined the earlier CALDERA Sandcat test agent. Classify them as an **investigative false lead**, not as two actual quarantine actions or a validated Defender false positive. No evidence yet ties either event to Sandcat.

**Evidence hygiene:** The browser screenshot contains host/account paths, browser tabs, process command lines and hashes. Do not commit that unredacted image. Only minimal sanitized metadata and the conclusion above are retained here.

## Step 6.10 — Native Defender detection and action events located (8 November 2026)

**Evidence:** Screenshot from the **actual Windows 10 SOC lab VM**, Administrator PowerShell, running the following read-only query:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Windows Defender/Operational'; Id=1116,1117,1118,1119} -MaxEvents 30 |
  Select-Object TimeCreated, Id, ProviderName
```

The screenshot shows four events from the **Microsoft-Windows-Windows Defender** provider, all on **19 October 2026** (Windows local time as displayed):

| Time (Windows display) | Event ID | Supported interpretation |
| --- | --- | --- |
| 00:21:06 | `1116` | Defender detected a threat |
| 00:21:22 | `1117` | Defender recorded an action |
| 00:25:12 | `1116` | Defender detected a threat |
| 00:25:23 | `1117` | Defender recorded an action |

**Assessment:** This confirms at least **two Defender threat detection event records and two corresponding action event records**, grouped by time, on the real endpoint. The screenshot does **not** reveal threat names, action types, action results, affected paths, or whether either detection was related to the earlier **CALDERA Sandcat** lab exercise. Consequently, **do not label these as confirmed quarantine actions** until the `1117` action details and detection correlation are examined.

This is independent of the two **Brave browser Sysmon** quarantine-string events from Step 6.9, which were benign-looking keyword false leads. The dedicated Defender Operational log is present on the Windows guest; earlier Splunk indexed-sourcetype inventory has **not** shown that dedicated Defender channel is forwarded to Splunk.

## Step 6.11 — Native Defender threat and quarantine action verified (8 November 2026)

**Evidence:** Screenshot from the Windows 10 VM's **Microsoft-Windows-Windows Defender/Operational** channel. The owner queried event IDs `1116`/`1117` on **19 October 2026** and extracted only named XML fields:

- `Threat Name`: **`Trojan:Win64/SandCat.RTS!MTB`** in all four records.
- `Severity Name`: **Severe**.
- `Category Name`: **Trojan**.
- `Action Name`: **Not Applicable** in the two `1116` detection records; **Quarantine** in both `1117` protection-action records.

| Windows guest local time | Defender ID | Threat classification | Action name |
| --- | --- | --- | --- |
| **2026-10-19 00:21:06** | `1116` | `Trojan:Win64/SandCat.RTS!MTB`, Severe/Trojan | Not Applicable |
| **2026-10-19 00:21:22** | `1117` | Same | **Quarantine** |
| **2026-10-19 00:25:12** | `1116` | Same | Not Applicable |
| **2026-10-19 00:25:23** | `1117` | Same | **Quarantine** |

**Assessment:** Defender **detected a SandCat-labelled Trojan twice and recorded two named quarantine actions**. The matching SandCat family name is consistent with the earlier controlled CALDERA/Sandcat exercise, but the screenshot does **not** establish a binary hash, process chain, exact CALDERA operation time or successful remediation. Event `1117` and `Action Name=Quarantine` demonstrate a **recorded action**, not independently a verified action outcome. This is **native endpoint evidence**, **not yet confirmed present in Splunk's `main` index**.

A proper SOC analyst must not conflate these events with the Brave browser `quarantine.mojom.Quarantine` false lead from Step 6.9.

**Case-study draft:** [Microsoft Defender SandCat investigation](../10-incident-investigations/defender-sandcat-case-study.md) — facts, false leads, limitations and remaining checks.

## Step 6.12 — Next read-only validation: quarantine outcome

On the Windows 10 VM, use Administrator PowerShell to extract only targeted XML fields from the two `1117` action records:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Windows Defender/Operational'; Id=1117; StartTime=[datetime]'2026-10-19 00:00:00'; EndTime=[datetime]'2026-10-19 01:00:00'} |
ForEach-Object {
  $e = $_; $xml = [xml]$e.ToXml()
  $fields = $xml.Event.EventData.Data | Where-Object { $_.Name -match 'Threat Name|Action Name|Error Code|Error Description|Action Status' }
  [pscustomobject]@{ Time = $e.TimeCreated; EventID = $e.Id; Details = (($fields | ForEach-Object { "$($_.Name)=$($_.'#text')" }) -join '; ') }
} | Format-List
```

This command is **read-only** and does not deliberately print raw messages, full paths or command lines. If no status or error fields are available, that absence should be documented rather than declaring success. Do not commit raw/unredacted screenshots.

## Future case study format

Each investigation should include a clear question or alert, a defined scope, redacted search evidence, time-ordered events, analysis, conclusion, and an improvement or containment recommendation. Separate observed facts from proposed responses and do not claim an attack succeeded based solely on the presence of a test agent.

## Security and portfolio handling

Do not commit raw Splunk indexes, credentials, tokens, agents' secrets, sensitive `splunkd.log` contents, unredacted Defender events or copied full packet captures. Use small sanitized screenshots and minimal example queries. Recheck security implications before making this private repository public.

*Baseline started: 7 November 2026.*
