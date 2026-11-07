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

## Follow-up read-only check

Count mentions of this rule ID in the current JSON alerts file (a count of **matching log lines**, not proof of a current agent connection):

```bash
sudo grep -c '"id":"100101"' /var/ossec/logs/alerts/alerts.json
```

If the file contains no match, that is evidence only for **that particular file**, not a comprehensive historical search. Don't commit complete raw alert files or unredacted command lines.

See [Wazuh SIEM deployment](../05-wazuh-siem/README.md) for manager and agent inventory.

*Evidence capture: 7 November 2026. Internal SOC training portfolio.*
