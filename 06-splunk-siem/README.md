# Splunk SIEM — Search and Investigation Evidence

> **Portfolio status:** Baseline architecture and installed Splunk version verified. Splunk CLI process status is now verified; indexed data sources, searches, and actual investigation outputs remain to be verified. This is a training-lab installation, not a production security platform.

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
| Current data ingestion | **Not yet verified** |

The guest addresses are sourced from the earlier `ip -br -4 addr`, `ip -4 route`, and `resolvectl status` screenshot; these are a dated network snapshot, not a fresh connectivity test.

## Evidence collection roadmap

- [x] Record the confirmed VM network layout from the existing screenshots.
- [x] Record the **installed Splunk version**, `10.6.1.1` (build `b2ae2962645c`) from the live Splunk VM.
- [x] Confirm `splunkd` and helper processes are running using the Splunk CLI.
- [ ] Verify Splunk Web access in the present session.
- [ ] Inspect available indexes and sources **without exposing raw sensitive event data**.
- [ ] Capture a redacted Splunk search demonstrating log investigation.
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

## Step 6.3 — Next evidence: Splunk indexed datasets

In Splunk Web, open **Search & Reporting** and run this read-only SPL query (no raw event fields):

```spl
| eventcount summarize=false index=*
| stats sum(count) as stored_events by index
| sort - stored_events
```

The query counts stored events in non-internal indexes visible to the user; it does not rely on the selected time picker. The results help identify which datasets exist, but do not prove that any were ingested recently. Internal indexes (for example `_internal`) are excluded by `index=*`. If the results are empty, investigate available indexes and permissions before drawing conclusions about ingestion.

Only commit counts and appropriately redacted screenshots—not raw endpoint events, credentials or Splunk secrets.

## Future case study format

Each investigation should include a clear question or alert, a defined scope, redacted search evidence, time-ordered events, analysis, conclusion, and an improvement or containment recommendation. Separate observed facts from proposed responses and do not claim an attack succeeded based solely on the presence of a test agent.

## Security and portfolio handling

Do not commit raw Splunk indexes, credentials, tokens, agents' secrets, sensitive `splunkd.log` contents, unredacted Defender events or copied full packet captures. Use small sanitized screenshots and minimal example queries. Recheck security implications before making this private repository public.

*Baseline started: 7 November 2026.*
