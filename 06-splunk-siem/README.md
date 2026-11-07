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

## Step 6.7 — Next step: isolate Defender and quarantine references

In **Splunk Web → Search & Reporting**, set the time picker to **All time**, then run:

```spl
index=main (quarantine OR "Windows Defender" OR "Microsoft Defender")
| stats count min(_time) as first_seen max(_time) as last_seen by host sourcetype
| convert ctime(first_seen) ctime(last_seen)
| sort - count
```

This returns aggregate counts and timestamps by source without exposing raw endpoint command lines or Defender event payloads. A positive result warrants a separate, carefully redacted event-ID and time-correlation investigation. A zero result would require validating data coverage and other detection indicators before concluding the incident is absent. Do not claim a confirmed quarantine from broad keyword matches alone.

## Future case study format

Each investigation should include a clear question or alert, a defined scope, redacted search evidence, time-ordered events, analysis, conclusion, and an improvement or containment recommendation. Separate observed facts from proposed responses and do not claim an attack succeeded based solely on the presence of a test agent.

## Security and portfolio handling

Do not commit raw Splunk indexes, credentials, tokens, agents' secrets, sensitive `splunkd.log` contents, unredacted Defender events or copied full packet captures. Use small sanitized screenshots and minimal example queries. Recheck security implications before making this private repository public.

*Baseline started: 7 November 2026.*
