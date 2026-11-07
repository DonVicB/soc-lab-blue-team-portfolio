# Splunk SIEM — Search and Investigation Evidence

> **Portfolio status:** Baseline architecture and previously reported installation documented. Current Splunk version, running service, data sources, searches and actual investigation outputs remain to be verified. This is a training-lab installation, not a production security platform.

## Purpose

Use Splunk to search and correlate security telemetry, investigate endpoint detections, and produce clear SOC analyst case studies with **evidence, timeline, findings, and lessons learned**.

## VMware deployment baseline

| Property | Evidence / status |
| --- | --- |
| VMware guest display name | `Splunk-SIEM` |
| Guest environment | Ubuntu Server VM (terminal screenshot verified, 6 November 2026) |
| Internal SOC network | `ens33: 10.10.10.69/24`, within host-only VMnet1 subnet `10.10.10.0/24` |
| NAT network | `ens34: 192.168.111.135/24`, VMnet8 network `192.168.111.0/24` |
| Default IPv4 route | `192.168.111.2` via `ens34` |
| Resolver seen in screenshot | `192.168.111.2` on `ens34` |
| Splunk product | **Splunk Enterprise Free** — installed and Splunk Web previously accessible according to operator's earlier lab setup report |
| Current installed version | **Not yet verified** |
| Current running service and data ingestion | **Not yet verified** |

The guest addresses are sourced from the earlier `ip -br -4 addr`, `ip -4 route`, and `resolvectl status` screenshot; these are a dated network snapshot, not a fresh connectivity test.

## Evidence collection roadmap

- [x] Record the confirmed VM network layout from the existing screenshots.
- [ ] Record the **installed Splunk version** from the live Splunk VM.
- [ ] Confirm Splunk Web and relevant service status.
- [ ] Inspect available indexes and sources **without exposing raw sensitive event data**.
- [ ] Capture a redacted Splunk search demonstrating log investigation.
- [ ] Complete a structured incident case study: Microsoft Defender detection/quarantine during a controlled CALDERA Sandcat exercise, using only evidence actually observed.
- [ ] Explain the investigative decision process and map validated activity to MITRE ATT&CK where supported.

## First read-only check

On the `Splunk-SIEM` Ubuntu VM:

```bash
sudo /opt/splunk/bin/splunk version
```

Record the command output. This should identify the installed Splunk version, assuming the earlier installation uses the standard `/opt/splunk` directory. If the path is different, document the observed location rather than guessing the version.

## Future case study format

Each investigation should include a clear question or alert, a defined scope, redacted search evidence, time-ordered events, analysis, conclusion, and an improvement or containment recommendation. Separate observed facts from proposed responses and do not claim an attack succeeded based solely on the presence of a test agent.

## Security and portfolio handling

Do not commit raw Splunk indexes, credentials, tokens, agents' secrets, sensitive `splunkd.log` contents, unredacted Defender events or copied full packet captures. Use small sanitized screenshots and minimal example queries. Recheck security implications before making this private repository public.

*Baseline started: 7 November 2026.*
