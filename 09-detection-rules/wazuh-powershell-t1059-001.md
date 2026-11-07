# Wazuh custom detection — Suspicious PowerShell process command line

> **Status:** Local rule **present and screenshot-verified** on 7 November 2026. A matching alert, end-to-end event flow and current agent ingestion **have not yet been verified**.

## Detection objective

Identify Windows Sysmon process-creation events associated with PowerShell or PowerShell Core whose command lines contain selected suspicious arguments or scripting expressions. This is a lab detection engineering exercise, not a standalone determination of malicious behaviour.

| Field | Screenshot-verified configuration |
| --- | --- |
| Wazuh custom rule ID | `100101` |
| Rule severity level | `10` |
| Rule group | `Windows,sysmon,powershell,mitre,` |
| Parent/event group condition | `<if_group>sysmon_event1</if_group>` |
| Process executable field | `win.eventdata.image` |
| Command-line field | `win.eventdata.commandLine` |
| Match engine | `pcre2` for both fields |
| MITRE ATT&CK | `T1059.001` — PowerShell |
| Description | `Suspicious PowerShell command line detected on $(agent.name)` |
| File | `/var/ossec/etc/rules/local_rules.xml` |

## Detection logic — observed, not reconstructed

In the screenshot, the process-image regular expression is case-insensitive and seeks PowerShell executables, including `powershell.exe` and `pwsh.exe`.

The command-line expression is also case-insensitive. Visible terms include:

- `-enc`, `-encodedcommand`, `-nopl`, `-noprofile`
- `-executionpolicy bypass`
- `iex`, `invoke-expression`, `downloadstring`, `frombase64string`

The screenshot shows the `win.eventdata.commandLine` PCRE2 expression wrapped across the terminal display. The list above describes what can be read; **it is not a guaranteed full or byte-accurate reproduction of the original regex**. The full rule XML has not yet been copied from a source file into this repository.

**Expected correlation:** A Sysmon process-creation event classified into the `sysmon_event1` group, with the specified PowerShell image and one of the selected command-line patterns, is intended to match rule `100101`. It may also match benign administrative activity; severity 10 is the configured Wazuh rule level, not proof of confirmed compromise.

## Evidence and limitations

The Wazuh server terminal screenshot on **7 November 2026** shows a successful read-only command:

```bash
sudo grep -nEi -C 5 'T1059[.]001|powershell' /var/ossec/etc/rules/local_rules.xml
```

The output shows the XML group, rule ID `100101`, level `10`, `if_group`, two `field` conditions, description and `<mitre><id>T1059.001</id></mitre>`.

An initial command with the file path missing its leading slash failed; the corrected command above succeeded. No custom rule changes were made during this evidence collection.

**Important operational distinction:** The rule's presence on disk does **not** prove that Wazuh loaded it successfully, that a live Sysmon event was processed, or that a matching alert fired. Earlier on 7 November 2026 the manager reported **two registered Windows agents disconnected** (IDs `001` and `002`). Therefore, **do not claim current live endpoint detection** based on this screenshot alone.

## Remaining evidence to collect

- [x] Record the local XML rule's identity, level, conditions, description and ATT&CK mapping from screenshot.
- [ ] Safely capture the **complete** command-line PCRE2 rule from the source file and review for secrets before publication.
- [ ] Verify the rule is accepted by Wazuh (configuration test or representative `wazuh-logtest`).
- [ ] Capture a controlled Sysmon process-creation example and corresponding Wazuh alert with rule ID `100101`.
- [ ] Record event time, agent, event ID, triggering command, alert details and assessment with sensitive fields redacted.
- [ ] Evaluate false positives (legitimate admin PowerShell use) and missed detections before describing efficacy.

## Step 5.5 — Alert search baseline, 7 November 2026

The lab owner ran the recommended read-only count against the current Wazuh JSON alert file:

```bash
sudo grep -c '"id":"100101"' /var/ossec/logs/alerts/alerts.json
```

**Operator-reported result:** `0`.

**Assessment:** No line in the *current file* matched that exact text. This is **not evidence that the rule is broken**, has never fired, or that historical/rotated alerts contain no matches. The command is a literal text search, so different JSON formatting may also affect results. The previous agent inventory reported both registered Windows agents disconnected. Do **not** describe this rule as a demonstrated live detection from the present evidence.

## Step 5.6 — Synthetic Sysmon JSON Logtest, 7 November 2026

A screenshot of the Ubuntu Wazuh VM confirms that `sudo /var/ossec/bin/wazuh-logtest` started successfully as version `v4.14.9`. The operator pasted the following **synthetic** process-creation event (format shown with indentation for readability):

```json
{
  "win": {
    "system": {
      "providerName": "Microsoft-Windows-Sysmon",
      "channel": "Microsoft-Windows-Sysmon/Operational",
      "severityValue": "INFORMATION",
      "eventID": "1"
    },
    "eventdata": {
      "image": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
      "commandLine": "powershell.exe -enc TEST"
    }
  }
}
```

**Screenshot-visible test results:**

| Stage | Result |
| --- | --- |
| Phase 1 | Completed pre-decoding, input event echoed |
| Phase 2 | Completed decoding, decoder name `json` |
| Fields extracted | `win.eventdata.image`, `win.eventdata.commandLine`, `win.system.eventID=1`, provider and channel |
| Phase 3 | **No Phase 3 rule-filtering output shown** |
| Rule `100101` | **No matching rule output; not triggered in this test** |

**Interpretation:** Field extraction works for this JSON sample. The rule requires `<if_group>sysmon_event1</if_group>`, which depends on a previously matched parent rule belonging to that group. The sample was decoded using `json`; the screenshot does not demonstrate that the Windows EventChannel rule chain or the `sysmon_event1` group was reached. This is the **likely reason** for the absent Phase 3 match, but the screenshot alone does not prove the exact internal rule-evaluation path.

Wazuh's documented `<if_group>` semantics require a prior match in the specified group. Logtest is a rules sandbox; its result for this manually supplied JSON is **inconclusive for live EventChannel events**. Do not modify built-in rules to force a passing test.

**Outcome:** `100101` remains **configuration-verified, alert-unverified**. Neither live endpoint ingestion nor rule effectiveness has been demonstrated. The manager's last agent inventory reported both Windows agents disconnected.

## Next evidence priority — Windows Sysmon source

Exit the interactive test with Ctrl+C, then on the Windows 10 lab endpoint inspect the Sysmon Operational log metadata:

```powershell
Get-WinEvent -ListLog 'Microsoft-Windows-Sysmon/Operational' | Select-Object LogName, RecordCount, IsEnabled
```

**Results (7 November screenshots):** The Windows 10 Sysmon Operational log is **enabled** with **33,076 records** across all event types. A follow-up read-only query confirmed **five recent Sysmon Event ID 1 process-creation records**, with local timestamps **14:58:42–14:59:37**, all from `Microsoft-Windows-Sysmon`. This confirms source events exist, **not** that the PowerShell rule's conditions matched, Wazuh received them, or rule `100101` fired. See [Windows endpoint security evidence](../04-windows-endpoint-security/README.md). Agent connectivity and alert correlation remain pending.

## Later evidence — Sysmon process-creation telemetry confirmed (7 November 2026)

After the earlier synthetic Logtest produced no rule `100101` match, the actual Windows VM was matched to Wazuh agent `002` (`DESKTOP-2MNIK0U`), and the manager reported that agent **Active**. The owner subsequently reported these results from the current `alerts.json` file:

| Filter | Matching lines |
| --- | ---: |
| Agent `002` | 229 |
| Agent `002` + `Microsoft-Windows-Sysmon` | 148 |
| Agent `002` + Sysmon provider + `"eventID":"1"` | **60** |

**Revised assessment:** Unlike the earlier manually pasted JSON simulation, the manager alert file now contains lines referencing **real Sysmon Event ID 1 process-creation data from the active Windows agent**. That is meaningful evidence of source-side logging and Wazuh alert-file receipt. These are grep line counts, not counts of unique parsed alerts. **No evidence yet shows that custom rule `100101` was the alerting rule for any of those 60 lines.** The earlier direct count of rule `100101` returned zero at that time; a structured rule-ID summary is the next step before any claim that the custom detection fired.

See [Wazuh alert evidence](../05-wazuh-siem/README.md).

## Step 5.17 — Actual rule IDs present in Sysmon process-creation alerts

A screenshot on **7 November 2026** shows the manager running a read-only Python parser over `alerts.json` and counting `rule.id` on alerts filtered to agent `002`, Sysmon provider, and Event ID `1`. It found **60 parsed records** under six IDs: `92052` (25), `92032` (14), `92021` (8), `92066` (8), `92031` (4), and `92055` (1).

**Custom rule `100101` appeared in none of these 60 parsed alerts.** This is a **negative finding limited to this specific current-file subset**, not proof of a global absence or a broken rule. Earlier Logtest input was decoded as ordinary JSON without showing a matching parent `sysmon_event1` group, so it did not constitute a conclusive live rule evaluation. Revisit rule `100101` during a dedicated controlled detection engineering case study, rather than claiming it matched.

See the [Wazuh alert breakdown](../05-wazuh-siem/README.md).

Do not commit complete raw alert files or unredacted endpoint command lines.

See [Wazuh SIEM deployment](../05-wazuh-siem/README.md) for manager and agent inventory.

*Evidence capture: 7 November 2026. Internal SOC training portfolio.*
