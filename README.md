# wazuh-detection-lab

Custom Wazuh detection rules mapped to MITRE ATT&CK, with Python tooling for alert triage.

## Detections
| Detection | ATT&CK mapping | Folder |
|---|---|---|
| Brute force attack | T1110 Brute Force | `rules/brute-force/` |
| Ransomware activity | T1486 Data Encrypted for Impact | `rules/ransomware/` |
| Data exfiltration | TA0010 Exfiltration | `rules/exfiltration/` |

## Lab stack
- **Wazuh SIEM:** endpoint agents with centralized log collection, correlation, and real-time alerting
- **Sysmon:** endpoint telemetry feeding the detections
- **VirusTotal:** automated enrichment of suspicious files, IPs, and domains
- **IRIS:** incident and case tracking
- **Grafana:** dashboards for security events
- **Juniper firewall logs:** perimeter visibility alongside endpoint data

## Repository structure
- `rules/`: custom Wazuh rules (XML), one folder per detection
- `scripts/`: Python tools for alert analysis and triage
- `tests/`: sample events used to verify each rule fires
- `docs/`: write-up per detection: what it catches, how it works, known false positives

## Status
Three detections built. Test events and write-ups are being added for each one.

## Disclaimer
All data comes from a personal lab. No employer data is included. For defensive use only.

## License
MIT
