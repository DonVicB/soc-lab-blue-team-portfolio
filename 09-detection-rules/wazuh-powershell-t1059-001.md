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

## Step 5.6 — Interactive Wazuh Logtest launched, 7 November 2026

A screenshot of the Ubuntu Wazuh VM confirms that `sudo /var/ossec/bin/wazuh-logtest` started successfully as version `v4.14.9` and printed `Type one log per line`. **No test event or matching rule output has been supplied yet.** A running test tool is not evidence that custom rule `100101` fired.

**Important tool limitation:** Standard Windows Sysmon EventChannel events use Wazuh's internal `windows_eventchannel` decoder. Pasting their reconstructed JSON form into CLI `wazuh-logtest` ordinarily invokes the `json` decoder and may not traverse the normal `60000` → `60004` → `61600` → `61603` parent-rule path needed by `<if_group>sysmon_event1</if_group>`. Therefore a negative interactive JSON test is **inconclusive** for live Windows EventChannel detection. Do not modify `0575-win-base_rules.xml` just to force a passing test in the running manager. Wazuh's built-in decoder behavior and this issue are documented in project discussions and ruleset files.

**Next:** Safely feed one synthetic Sysmon Event 1 JSON record into the interactive tool to observe decoding and rule evaluation, label any result **simulated**, and distinguish JSON test limitations from agent ingestion. Reconnect/validate Windows agents and use a real Sysmon event only as a separate controlled exercise.

## Next validation step

Use Wazuh's interactive test utility to check loaded rules using a representative, sanitized event, without changing the deployed rules:

```bash
sudo /var/ossec/bin/wazuh-logtest
```

Collect the initial output and, in a subsequent step, feed a safe representative Sysmon process-create event. An interactive logtest match is proof of **test matching only**, not confirmation of live agent telemetry or alert generation.

Do not commit complete raw alert files or unredacted endpoint command lines.

See [Wazuh SIEM deployment](../05-wazuh-siem/README.md) for manager and agent inventory.

*Evidence capture: 7 November 2026. Internal SOC training portfolio.*
