# MITRE ATT&CK Mapping

| ID | Technique | Lab evidence | Confidence |
|---|---|---|---|
| T1190 | Exploit Public-Facing Application | TeamCity endpoint manipulation correlated with CVE-2024-27198 | Medium-high |
| T1505.003 | Server Software Component: Web Shell | JSP uploaded in plugin ZIP and invoked by POST requests | High |
| T1059.004 | Unix Shell | Shell commands supplied through the JSP `cmd` parameter | High |
| T1565.001 | Stored Data Manipulation | Command writes deceptive content to `/tmp/Creds.txt` | High |
| T1610 | Deploy Container | Docker command observed as part of host-access attempt | Medium |

Mappings are based on challenge context and visible PCAP excerpts. Confirm with the full capture and host telemetry before using these mappings in a real incident report.
