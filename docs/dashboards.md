# Grafana dashboards

Security dashboards built on Wazuh alert data from the lab.

## Alerts by MITRE ATT&CK tactic
Alert counts grouped by ATT&CK tactic, showing which stages of an attack the rules are catching.
![Alerts by MITRE tactic](images/grafana-dashboard-1.png)

## Top 10 alerts by rule
The most frequently firing rules, including the custom detections.
![Top 10 alerts by rule](images/grafana-dashboard-2.png)

## Critical Sysmon alerts
High-severity alerts generated from Sysmon endpoint telemetry.
![Critical Sysmon alerts](images/grafana-dashboard-3.png)

## Source IPs to countries (Sankey)
Flow of source IP addresses to their countries of origin.
![Sankey: IP addresses to countries](images/grafana-dashboard-4.png)

## Geolocation map
Geographic view of connection sources.
![Geomap](images/grafana-dashboard-5.png)

## Alert trend and timeline
Alert volume over time, used to spot spikes and attack windows.
![Alert trend and timeline](images/grafana-dashboard-6.png)
