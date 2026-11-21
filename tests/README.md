# Tests

Detection tests for the rules in `rules/`. Each file lists the commands that trigger a rule, the rule ID to look for in the Wazuh dashboard, and the expected result.

| Use case | Test file |
|---|---|
| Ransomware behavior | [ransomware.md](ransomware.md) |
| Data exfiltration | [exfiltration.md](exfiltration.md) |
| LOLBins abuse | [lolbins.md](lolbins.md) |
| Brute force and login success | Use `/var/ossec/bin/wazuh-logtest`: feed failed SSH login lines from the same source IP, then a successful login, and check which rule fires at each step. |

## Before testing
- Enable `logall` and `logall_json` in `ossec.conf` on the manager so you can confirm events reach Wazuh.
- Validate the rules file with `/var/ossec/bin/wazuh-analysisd -t` before restarting the manager. Empty output means it is clean.
- Run all tests on an isolated lab machine, never on a production system.

## Notes
- Tests simulate behavior and verify that rules fire. They are not live malware.
- Sysmon deduplicates rapid repeated connections, so frequency tests need a short pause between connections.
