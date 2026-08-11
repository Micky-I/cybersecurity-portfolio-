# Azure Cloud Security: Segmented Network with RBAC-Controlled Access

## Context
Built as part of my Cert IV in Cybersecurity, using an Azure for Students tenant. The brief was to prepare a cloud environment for migrating an on-premises web application with an MS-SQL backend — designing a network that keeps the database tier isolated from direct internet access, with access control managed through Azure AD groups and role-based permissions rather than shared credentials.

## Objective
Deploy a segmented Azure Virtual Network with separate frontend and backend subnets, apply Network Security Groups to enforce least-privilege traffic rules between them, deploy a web server and SQL server into the correct tiers, then set up Azure AD users, a security group, and RBAC role assignments to manage who can access the environment — and verify all of it with real connectivity tests.

## Architecture

- **Frontend subnet** (10.9.0.0/24) — hosts the web server, reachable from the internet via a static public IP
- **Backend subnet** (10.9.1.0/24) — hosts the SQL server, with no public IP and no direct internet exposure
- Two Network Security Groups enforce what can cross between the subnets and the internet

![Virtual network with both subnets configured](screenshots/vnet-with-subnets.png)

## Build Process

### 1. Network foundation
Created the resource group, then built out the frontend and backend subnets inside a single VNet, and a static public IP address reserved for the web tier only.

![Frontend subnet configuration](screenshots/frontend-subnet-config.png)

![Backend subnet configuration](screenshots/backend-subnet-config.png)

![Static public IP address configuration](screenshots/public-ip-config.png)

### 2. Network Security Groups — least-privilege segmentation
This was the core security control of the build. Rather than one flat network, each subnet got its own NSG with only the traffic it actually needs:

- **Frontend NSG:** allows inbound HTTP (port 80) from anywhere, since this is the public-facing web tier.
- **Backend NSG:** allows inbound SQL (port 1433) *only* from the frontend subnet's address range (10.0.0.0/24) — the database is not reachable from the internet or from anywhere else, only from the web server that's meant to talk to it.

![Frontend NSG rule — HTTP allowed from any source](screenshots/frontend-nsg-rule-http.png)

![Backend NSG rule — SQL allowed only from the frontend subnet](screenshots/backend-nsg-rule-sql.png)

Both NSGs were then explicitly associated with their matching subnets to take effect.

![NSG associated with the frontend subnet](screenshots/nsg-frontend-subnet-association.png)

![NSG associated with the backend subnet](screenshots/nsg-backend-subnet-association.png)

### 3. Virtual machine deployment
Deployed a Windows Server VM into the frontend subnet (with the public IP attached) to act as the web server, and a SQL Server VM into the backend subnet (private IP only) to act as the database. Trusted launch (secure boot + vTPM) was enabled on both VMs as a baseline hardening measure.

![Frontend web server VM specification](screenshots/frontend-vm-spec.png)

![Backend SQL server VM specification](screenshots/backend-vm-sql-spec.png)

An additional 32GB disk was attached to the SQL VM to provide dedicated storage for database backups, separate from the OS disk.

![Additional backup disk attached to the SQL VM](screenshots/backend-vm-extra-disk.png)

### 4. Identity and access management
Rather than sharing local admin credentials, access to the VMs was managed through Azure AD: created user accounts, grouped them into a single security group, and delegated the "Virtual Machine User Login" role to that group at the resource group's management scope — so access can be managed by adding/removing group membership rather than touching individual VM permissions.

![Creating the VM users security group](screenshots/security-group-creation.png)

![Security group membership](screenshots/security-group-members.png)

![Delegating the Virtual Machine User Login role to the group](screenshots/rbac-role-assignment.png)

![Confirmed role assignment at the management group scope](screenshots/rbac-role-confirmed.png)

A separate "Password Administrator" role was assigned to the group owner, so account issues could be handled without needing full administrative access to the tenant.

![Password Administrator role assigned to the group owner](screenshots/password-admin-role.png)

## Testing & Verification
NSGs don't allow RDP by default (correctly — RDP shouldn't be open to the internet permanently), so temporary rules were added specifically for testing, on the understanding they'd be removed afterward.

![Temporary RDP rule added to the frontend NSG](screenshots/temp-rdp-rule-frontend.png)

![Temporary RDP rule added to the backend NSG](screenshots/temp-rdp-rule-backend.png)

**Test 1 — External access to the web tier:** Connected to the web server VM over RDP using its public IP, confirming the frontend subnet is reachable from outside the network as intended.

![Initiating RDP connection to the web server](screenshots/rdp-connect-initiate.png)

![RDP session established on the web server](screenshots/rdp-session-connected.png)

**Test 2 — Internal-only access to the database tier:** From inside the web server's RDP session, connected onward to the SQL server using its private IP — proving the database is reachable from the web tier (as required) but was never exposed directly to the internet.

![Jumping from the web VM to the SQL VM via its private IP](screenshots/rdp-jump-to-sql-vm.png)

![System Information confirming the session landed on the SQL VM](screenshots/rdp-confirmed-on-sql-vm.png)

**Test 3 — Bidirectional connectivity:** Ran `ipconfig` and `ping` from both VMs against each other's private IPs to confirm routing was working correctly in both directions across the subnet boundary.

![Bidirectional ping test between the web and SQL VMs](screenshots/bidirectional-connectivity-test.png)

**Test 4 — Storage confirmation:** Verified the additional backup disk was visible and correctly sized on the SQL VM.

![Additional disk visible and unallocated on the SQL VM](screenshots/sql-vm-disk-confirmed.png)

**Cleanup:** Once testing was complete, the temporary RDP rules were removed from both NSGs, returning the network to its intended least-privilege state — the backend subnet should never have an open RDP path from the internet in normal operation.

![Final resource inventory after cleanup](screenshots/final-resource-inventory.png)

## Verification Summary
- Frontend subnet: reachable from the internet via public IP, confirmed via RDP
- Backend subnet: not directly internet-reachable; only accessible from the frontend subnet, confirmed via internal RDP jump and ping
- NSG rules: correctly scoped per tier (HTTP open on frontend, SQL restricted to frontend-only on backend)
- RBAC: VM access delegated through Azure AD group membership rather than individual permissions, confirmed at the management group scope
- Temporary test rules: added, used, and removed — network returned to its intended locked-down state

## Lessons Learned
The most useful part of this build was thinking through access in terms of *who needs to reach what* before writing any NSG rules — the backend subnet only ever needed to accept traffic from the frontend subnet's address range, never from "Any," and getting that source restriction right (10.0.0.0/24 rather than a wildcard) is what actually makes the segmentation meaningful rather than just organizational. Managing VM access through an Azure AD security group and RBAC, instead of handing out local admin passwords, was also a good practical example of how identity-based access control scales better than credential sharing — removing someone's access later is just a group membership change, not a password rotation across every machine they could reach. Remembering to strip the temporary RDP test rules afterward reinforced that a "temporary" opening for testing is still a real exposure until it's actually closed.

## Tools & Technologies
- Microsoft Azure (Virtual Network, NSGs, VMs, Azure AD)
- RBAC (Virtual Machine User Login, Password Administrator roles)
- RDP (connectivity testing)
- ping, ipconfig (network verification)
