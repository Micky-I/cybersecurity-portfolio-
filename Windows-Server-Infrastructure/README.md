# Windows Server Infrastructure: Active Directory, Print/Backup Server, and Monitoring

## Context
Built as part of my Cert IV in Cybersecurity. The brief was to expand a fictional company's network by deploying a secondary domain-adjacent server, extending Active Directory for a new business division, standing up a shared network printer, configuring automated backups, and then hardening/monitoring the environment with baseline reporting and auditing.

## Objective
Deploy and configure a three-machine Windows Server environment (a domain controller, a print/backup server, and a client workstation), extend Active Directory with a new organisational structure, enforce a Group Policy setting and verify it actually applies, configure a working backup and recovery process, and use built-in Windows tools to monitor server health and troubleshoot issues.

## Architecture
- **DC-VM** — Domain Controller running AD DS and DNS for the `micky.com` domain
- **SRV** — joined to the domain; hosts the print server role and Windows Server Backup, with a dedicated 100GB volume for backups
- **CLT1** — a domain-joined Windows 10 client used to test domain logon, printing, and Group Policy enforcement

## Build Process

### 1. Base networking and domain setup
Configured static IP addressing on all three machines, then installed the AD DS and DNS roles on DC-VM to establish the `micky.com` domain. CLT1 and SRV were each joined to the domain and confirmed with the "Welcome to the micky.com domain" prompt.

![DC-VM static IP configuration](screenshots/dc-vm-ip-config.png)

![AD DS and DNS role installation in progress](screenshots/addsdns-role-install.png)

![CLT1 successfully joined to the micky.com domain](screenshots/clt1-domain-joined.png)

![SRV static IP configuration](screenshots/srv-ip-config.png)

![SRV successfully joined to the micky.com domain](screenshots/srv-domain-joined.png)

### 2. Storage for backups
Attached and initialised an additional 100GB virtual disk on SRV, formatted as a dedicated backup volume (E:), kept separate from the OS disk.

![100GB backup drive attached and confirmed in Disk Management](screenshots/srv-backup-drive-attached.png)

### 3. Active Directory expansion
Created a new **Training** organisational unit to represent the company's new division, added **Training** and **Training Managers** security groups inside it, then created user accounts for all six staff members from the provided staff list and placed them in the correct groups.

![Training OU created in Active Directory](screenshots/training-ou-created.png)

![Training and Training Managers security groups created](screenshots/training-groups-created.png)

![All six Training staff accounts created](screenshots/training-users-added.png)

### 4. Print server
Installed the Print and Document Services role on SRV and created a shared TCP/IP network printer, matching the client's requested make/model and IP address.

![Print and Document Services role installation](screenshots/print-server-role-install.png)

![Shared network printer created and set as default](screenshots/network-printer-created.png)

### 5. Group Policy — and proving it worked
Created a Group Policy Object to enforce a specific homepage for the Training OU, linked it to the OU, then actually logged in as a Training-group user on CLT1 to verify the setting took effect — rather than assuming a saved GPO is a working GPO.

![GPO linked to the Training OU](screenshots/gpo-linked-to-ou.png)

![Homepage enforcement configured in the GPO](screenshots/gpo-homepage-setting.png)

![Logged in as a Training user on CLT1 — homepage correctly set to www.micky.com](screenshots/gpo-verified-on-client.png)

### 6. Backup and recovery
Installed the Windows Server Backup feature, configured a daily scheduled backup of the C: drive to the dedicated E: volume, and ran a manual backup to confirm it worked immediately rather than waiting for the schedule.

![Windows Server Backup feature installation](screenshots/backup-feature-install.png)

![Daily backup schedule configured — 9:00 PM, C: to E:](screenshots/backup-schedule-config.png)

![Scheduled backup confirmed active](screenshots/backup-schedule-confirmed.png)

![Manual backup running](screenshots/manual-backup-running.png)

To actually prove the backup worked, I deleted a file from the E: drive and ran a full recovery — not just trusting that a "successful" backup job would also mean a successful restore.

![File recovery completed successfully](screenshots/backup-restore-completed.png)

## Testing, Monitoring & Troubleshooting

**Connectivity:** Confirmed all three machines could reach each other on the network.

![Ping test confirming connectivity to all servers](screenshots/connectivity-test.png)

**Reliability Monitor:** Ran a baseline reliability check on each server. DC-VM came back clean, but SRV's report flagged a genuine event — the machine hadn't been shut down properly at one point — which I used as an opportunity to practice reading and interpreting real system health data rather than a synthetic example.

![Reliability Monitor baseline — DC-VM](screenshots/reliability-monitor-dc.png)

![Reliability Monitor on SRV, showing a real detected event](screenshots/reliability-monitor-srv.png)

**Troubleshooting tools:** Used `ping` and `ipconfig /all` from the command line to diagnose and confirm network configuration across the domain.

![Command-line connectivity troubleshooting](screenshots/troubleshooting-ping.png)

![Full ipconfig output confirming DNS, gateway, and adapter configuration](screenshots/ipconfig-diagnostics.png)

**Event Viewer:** Used the Application log to independently confirm the Group Policy had actually applied — a second, different way of verifying the same GPO result from earlier, using Windows' own audit trail rather than just visual inspection.

![Event Viewer confirming the Group Policy object was applied successfully](screenshots/event-viewer-gpo-applied.png)

**Performance Monitor:** Created a Data Collector Set with the six specific counters required (Available MBytes, Pages/Sec, Paging File % Usage, Processor Time, Disk Time, Network Bytes Total/Sec), generated some activity, and captured a live 60-second sample.

![Data Collector Set configured with the required performance counters](screenshots/data-collector-set-counters.png)

![Live performance capture showing real activity across all six counters](screenshots/performance-capture-live.png)

## Verification Summary
- Domain services: AD DS/DNS installed and functioning — both SRV and CLT1 successfully joined the domain
- Active Directory: Training OU, security groups, and all six staff accounts created and correctly grouped
- Print server: role installed, network printer shared and set as default
- Group Policy: homepage enforcement configured, linked, and independently verified twice (visually on the client, and via Event Viewer)
- Backup: scheduled job configured, manual backup run, and a full delete-and-restore cycle completed successfully
- Monitoring: Reliability Monitor baselines captured for all servers, including a real flagged event on SRV
- Performance monitoring: Data Collector Set built to the exact specification and captured live data

## Lessons Learned
The theme that came up again and again in this build was that "configured" and "working" aren't the same thing, and the gap between them is where the real skill is. Anyone can create a GPO or schedule a backup job — the part that actually matters is checking it: logging in as the affected user to see the policy apply, deleting a real file and restoring it rather than trusting the backup log, and cross-checking the GPO result through Event Viewer as a second, independent source rather than relying on a single point of confirmation. Finding a genuine reliability event on SRV (rather than a clean report) was also a good reminder that real systems produce real noise, and being able to read and interpret that noise — rather than just running the tool — is the actual job.

## Tools & Technologies
- Active Directory Domain Services & DNS
- Group Policy Management
- Print and Document Services
- Windows Server Backup
- Reliability Monitor & Performance Monitor
- Event Viewer
- Command-line diagnostics (ping, ipconfig)
