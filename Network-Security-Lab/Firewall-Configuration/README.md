# Firewall Configuration: Building the pfSense Base Firewall

## Context
Built as part of my Cert IV in Cybersecurity (Network Security unit). Before any VPN could be layered on top (see the [VPN Tunnel](../VPN-Tunnel/README.md) section of this repo), the brief required deploying and configuring a working firewall for a fictional organisation, then integrating it into the organisation's existing network monitoring platform.

## Objective
Select and deploy a firewall appliance for a fictional client, configure its base networking (interfaces, addressing, hostname), confirm it correctly passes and filters traffic for the LAN, and integrate it into a monitoring system so its status, performance, and logs are visible to the rest of the organisation.

## Why pfSense
I chose pfSense (specifically evaluated against the pfSense+ 6100 appliance spec) because it natively supports both site-to-site and client-to-site VPNs, is free/open-source with an optional paid tier for commercial support, and includes hardware-accelerated VPN performance (AES-NI/QAT) along with multi-WAN and high-availability options — more than enough headroom for a small-to-medium organisation's needs while staying within a realistic budget for the fictional client.

## Build Process

### 1. Base firewall setup
Installed pfSense on the firewall appliance (FW-1), configured the hostname and domain, and set static IP addressing on the LAN interface.

![Initial pfSense setup wizard — hostname and domain configuration](screenshots/fw1-initial-setup.png)

![LAN interface configuration — static IPv4 addressing](screenshots/lan-interface-config.png)

Once the base configuration was applied, I checked the system dashboard to confirm the device was running as expected — correct hostname, version, and hardware detected — before connecting a client PC and confirming it could route out to the internet through the new firewall. No point building anything further on top of a firewall that can't pass basic traffic yet.

![pfSense system dashboard confirming hostname, version, and hardware](screenshots/pfsense-dashboard.png)

### 2. Monitoring integration
The client's requirements specified that the new firewall needed to be monitored and audited by the organisation's existing security systems, not just administered standalone. I deployed a monitoring VM running Observium, enabled SNMP on FW-1 so it could be polled, and added the firewall as a monitored device using SNMP and syslog.

![Enabling SNMP on the firewall so it can be polled by the monitoring platform](screenshots/snmp-enable.png)

![Adding the firewall as a monitored device in Observium via SNMP](screenshots/observium-add-device.png)

I then confirmed the monitoring platform was correctly polling the firewall's live status — hostname, uptime, memory, and interface health all reporting back accurately in real time.

![Observium dashboard confirming the firewall is being actively monitored](screenshots/observium-monitoring-overview.png)

## Verification Summary
- Base connectivity: confirmed the LAN client could route to the internet through the new firewall
- System identity: confirmed hostname, version, and hardware matched the intended configuration
- Monitoring: confirmed the firewall was successfully polled via SNMP and reporting into Observium in real time

## Lessons Learned
This stage reinforced that a firewall isn't "done" once it's passing traffic — a deployed device that isn't reporting into the organisation's monitoring stack is effectively invisible to the SOC/network team until something goes wrong. Getting SNMP and syslog integration working correctly, and actually confirming the monitoring platform was polling live (not just configured), was as important a checkpoint as the basic connectivity test.

## Tools & Technologies
- pfSense (firewall appliance)
- SNMP (device polling)
- Syslog (remote logging)
- Observium (network monitoring platform)
- VMware (virtualised lab environment)
