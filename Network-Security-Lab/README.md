# Network Security Lab

A pfSense-based network security build completed as part of my Cert IV in Cybersecurity, split into two parts:

## [Firewall Configuration](Firewall-Configuration/README.md)
Building a pfSense firewall with LAN, DMZ, and Internet zones — NAT configuration, least-privilege firewall rules per zone, and live testing (curl, nslookup, ping) to prove each rule actually works as intended.

## [VPN Tunnel](VPN-Tunnel/README.md)
Building on top of the base firewall: a site-to-site IPSec VPN between two simulated offices, and a client-to-site OpenVPN setup for remote workers — both verified with real connectivity tests, not just configuration screenshots.
