# Detection engineering — SOC lab

Documented custom rules are evidence-led. Each entry separates a configuration observed on disk from a verified event and a demonstrated alert.

| Rule | Platform | ATT&CK technique | Evidence status |
| --- | --- | --- | --- |
| [Suspicious PowerShell command line — `100101`](wazuh-powershell-t1059-001.md) | Wazuh, Sysmon process-creation events | `T1059.001` | Rule configuration screenshot-verified, 7 Nov 2026; runtime match and alert pending |

## Validation standard

For each rule, capture:

1. Rule identifier, severity, source fields and original detection logic.
2. The test event, with time and source identified.
3. Corresponding alert or query result, redacted as needed.
4. Triage interpretation, false-positive considerations, and scope limits.

Rule presence in a configuration file alone is **not** proof of an operational detection. See the [Wazuh SIEM overview](../05-wazuh-siem/README.md).

*Private training lab documentation — do not upload secrets or raw personal event data.*
