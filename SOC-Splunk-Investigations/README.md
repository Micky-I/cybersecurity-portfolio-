# SOC & Splunk Investigations

Two SOC investigations completed as part of my Cert IV in Cybersecurity, using Splunk to detect, investigate, and document security incidents against real log data (Windows Security Event Logs and FortiGate firewall traffic).

Each investigation follows the same format: detection query → findings → hypothesis → MITRE ATT&CK mapping → recommendations → actual outcome, with supporting screenshots throughout.

## [Incident 1: Brute Force / Username Enumeration Detection](incident-01-bruteforce-detection/README.md)
Investigated a high volume of failed Windows logon attempts (EventCode 4625) across multiple hosts, using SPL to pivot from a broad anomaly search down to a specific source IP and build a timeline — then followed through to escalation and resolution.

## [Incident 2: Persistent IKE Handshake Retries (VPN Reconnaissance)](incident-02-vpn-ike-retries/README.md)
Investigated repeated, low-volume VPN handshake attempts in FortiGate firewall traffic logs — a different log source and a subtler pattern than a single flagged event, requiring session-bucketing and volume analysis rather than a simple keyword match.
