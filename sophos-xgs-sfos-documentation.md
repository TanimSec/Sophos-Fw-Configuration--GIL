# Sophos XGS Firewall Configuration (SFOS)
## Goinnovior Internal Infrastructure
---

# About This Document

This is the consolidated Sophos XGS Firewall (SFOS) configuration reference for Goinnovior Ltd's internal infrastructure. It is organized as a single progressive guide — beginner setup topics come first, advanced topics come last. Each section is self-contained and follows the same structure: **Overview → Prerequisites → Configuration Steps → Verification → Best Practices**.

Use the Table of Contents below to jump to any topic.

## Table of Contents

1. [Initial Setup and Licensing](#1-initial-setup-and-licensing)
2. [Interface and Zone Configuration](#2-interface-and-zone-configuration)
3. [DHCP and DNS Configuration](#3-dhcp-and-dns-configuration)
4. [Firewall Rules](#4-firewall-rules)
5. [NAT Configuration](#5-nat-configuration)
6. [Web and Application Control](#6-web-and-application-control)
7. [IPS Configuration](#7-ips-configuration)
8. [ATP (Advanced Threat Protection) Configuration](#8-atp-advanced-threat-protection-configuration)
9. [SSL/TLS Inspection](#9-ssltls-inspection)
10. [Site-to-Site VPN](#10-site-to-site-vpn)
11. [Remote Access / SSL VPN](#11-remote-access--ssl-vpn)
12. [Authentication: AD, RADIUS, STAS](#12-authentication-ad-radius-stas)
13. [High Availability (HA)](#13-high-availability-ha)
14. [Traffic Shaping / QoS](#14-traffic-shaping--qos)
15. [Logging, Reporting, and SIEM Integration](#15-logging-reporting-and-siem-integration)
16. [Backup, Restore, and Firmware Upgrades](#16-backup-restore-and-firmware-upgrades)
17. [Sophos Central Management](#17-sophos-central-management)
18. [Troubleshooting and CLI Diagnostics](#18-troubleshooting-and-cli-diagnostics)

---

# 1. Initial Setup and Licensing

## Overview

This section covers the first-boot configuration of a Sophos XGS appliance, initial registration with Sophos Central, and activation of the base and add-on licenses/subscriptions. This is the mandatory starting point before any other configuration on this list.

## Prerequisites

- Physical or virtual XGS appliance racked, powered, and cabled (WAN and LAN ports connected).
- A management laptop able to reach the default appliance IP (`172.16.16.16` on Port 1 by default).
- Sophos Central account credentials for the organization.
- Base license key or activation code (from Sophos partner/reseller invoice).

## Configuration Steps

### Step 1: Access the Initial Setup Wizard

1. Connect a laptop to **Port 1 (LAN)** of the appliance.
2. Set the laptop's NIC to a static IP in the `172.16.16.0/24` range (e.g., `172.16.16.100/24`).
3. Browse to `https://172.16.16.16:4444`.
4. Accept the certificate warning and log in with default credentials (`admin` / `admin` — you will be forced to change this).

### Step 2: Set Admin Credentials and Time Zone

| Field | Value |
|---|---|
| New Admin Password | `<value>` |
| Confirm Password | `<value>` |
| Time Zone | `Asia/Dhaka` |
| NTP Server | `<value or default pool.ntp.org>` |

### Step 3: Configure Network Settings

Navigate: `Network → Interfaces` (during wizard, this appears as "Network Configuration").

| Interface | Zone | IP Address | Purpose |
|---|---|---|---|
| Port1 | LAN | `<value>` | Internal management/network |
| Port2 | WAN | `<value>` | Internet uplink |

### Step 4: Register the Device

1. Navigate to `Administration → Licensing`.
2. Click **Register with Sophos Central** (or enter offline Activation Code if air-gapped).
3. Sign in with the organization's Sophos Central credentials when prompted.
4. Confirm the appliance appears under **Devices** in Sophos Central within a few minutes.

### Step 5: Activate Subscriptions

Navigate: `Administration → Licensing → Subscriptions`.

| Subscription | Status After Activation |
|---|---|
| Base Firewall | Active |
| Network Protection (IPS) | Active / Trial / Not Purchased |
| Web Protection | Active / Trial / Not Purchased |
| Central Orchestration | Active / Trial / Not Purchased |
| Zero-Day Protection (Sandstorm/ATP) | Active / Trial / Not Purchased |
| Enhanced Support / TotalTech | Active / Trial / Not Purchased |

Confirm expiry dates against Goinnovior's procurement records before closing out the setup.

## Verification

- `Administration → Licensing` shows all purchased subscriptions as **Active**, not **Expired** or **Not Subscribed**.
- Device shows **Online** and reporting correctly in Sophos Central under the correct organization/device group.
- Admin login now requires the new password (default credentials rejected).

## Best Practices

- ✔ Change the default admin password immediately — never leave `admin/admin` active.
- ✔ Register the device to Central before doing further configuration, so all changes are backed up centrally from day one.
- ✔ Record serial number, license keys, and expiry dates in the Goinnovior asset register immediately after activation.
- ✔ Set the correct time zone and NTP before configuring any logging — mismatched time stamps make log correlation unreliable later.
- ✔ Enable email notifications for license expiry under `Administration → Notification Settings`.

---

# 2. Interface and Zone Configuration

## Overview

SFOS organizes all traffic handling around **zones** (LAN, WAN, DMZ, VPN, WiFi, Local) and the physical/virtual **interfaces** assigned to them. Correct zone assignment is foundational — firewall rules, NAT, and routing all reference zones rather than raw interfaces.

## Prerequisites

- Initial setup (Section 1) completed and device registered.
- Network diagram or VLAN plan for Goinnovior's site showing required segments (LAN, DMZ, Guest, etc.).

## Configuration Steps

### Step 1: Review Default Zones

Navigate: `Network → Zones`.

| Zone | Type | Typical Use |
|---|---|---|
| LAN | Fixed | Internal trusted network |
| WAN | Fixed | Internet-facing |
| DMZ | Fixed | Publicly reachable servers |
| VPN | Fixed | Site-to-Site / Remote Access tunnels |
| WiFi | Fixed | Wireless clients (if AP controller used) |
| Local | Fixed | Traffic to/from the firewall itself |
| `<custom>` | Custom (LAN-type or DMZ-type) | Department/segment-specific zones |

### Step 2: Configure Physical Interfaces

Navigate: `Network → Interfaces → Physical Interfaces`.

1. Select the port to configure (e.g., Port3).
2. Set:

| Field | Value |
|---|---|
| Name | `<value>` |
| Network Zone | `<LAN / DMZ / custom>` |
| IPv4 Configuration | Static / DHCP |
| IP Address / Netmask | `<value>` |
| Gateway (if WAN) | `<value>` |

3. Click **Apply**.

### Step 3: Create VLAN Sub-Interfaces (if segmenting)

Navigate: `Network → Interfaces → Add Interface → VLAN`.

| Field | Value |
|---|---|
| VLAN ID | `<value>` |
| Parent Interface | `<value>` |
| Zone | `<value>` |
| IP Address | `<value>` |

Repeat per required VLAN (e.g., Data, Voice, Guest, Servers).

### Step 4: Create Custom Zones (Optional)

Navigate: `Network → Zones → Add Zone`.

| Field | Value |
|---|---|
| Zone Name | `<value>` |
| Zone Type | LAN-type / DMZ-type |
| Applicable Services (ACL) | HTTPS, SSH, Ping, User Portal — select as needed |

## Verification

- `Network → Interfaces` shows all ports/VLANs with correct zone, IP, and **Up** status (green).
- `Network → Zones` shows correct interface-to-zone mapping.
- From a host on each segment, confirm reachability to the zone's gateway IP.

## Best Practices

- ✔ Never leave unused physical interfaces in the LAN zone by default — set unused ports to a "Quarantine" or disabled state.
- ✔ Use VLANs for logical segmentation rather than creating excess physical zones where hardware ports are limited.
- ✔ Restrict zone ACLs (`Network → Zones → [Zone] → ACL`) to only the admin services actually required (e.g., disable HTTPS/SSH admin access from WAN zone).
- ✔ Name interfaces and zones descriptively (`DMZ-WebServers`, not `Port5`) so firewall rules remain readable.
- ✔ Document every VLAN ID and subnet in the Goinnovior network diagram before creating it in SFOS.

---

# 3. DHCP and DNS Configuration

## Overview

SFOS can act as the DHCP server for internal segments and as a DNS forwarder/resolver for internal clients. This section covers configuring DHCP scopes per zone/VLAN and DNS forwarding settings.

## Prerequisites

- Interfaces and zones configured (Section 2).
- IP addressing plan for each VLAN/segment (subnet, gateway, reserved ranges).
- Upstream DNS servers to forward to (ISP DNS or internal AD DNS servers).

## Configuration Steps

### Step 1: Configure DNS Forwarding

Navigate: `Network → DNS`.

| Field | Value |
|---|---|
| DNS Server 1 | `<value>` |
| DNS Server 2 | `<value>` |
| DNS Query Route | Use for firewall's own resolution |

### Step 2: Create a DHCP Server Scope

Navigate: `Network → DHCP → Add`.

| Field | Value |
|---|---|
| Name | `<value>` |
| Interface | `<VLAN/Zone interface>` |
| Start IP | `<value>` |
| End IP | `<value>` |
| Netmask | `<value>` |
| Default Gateway | `<value — usually firewall interface IP>` |
| Lease Time | `<value, default 604800 sec>` |
| Primary DNS | `<value>` |
| Secondary DNS | `<value>` |

Repeat for each VLAN requiring DHCP.

### Step 3: Configure MAC Reservations (Optional)

Navigate: `Network → DHCP → [Scope] → Add Reservation`.

| Field | Value |
|---|---|
| MAC Address | `<value>` |
| Reserved IP | `<value>` |
| Description | `<value, e.g., printer, server>` |

### Step 4: Configure DHCP Relay (if DHCP server is elsewhere, e.g., Windows AD)

Navigate: `Network → DHCP → DHCP Relay → Add`.

| Field | Value |
|---|---|
| Interface | `<value>` |
| DHCP Server IP | `<value, e.g., AD server>` |

## Verification

- Client on the relevant VLAN receives an IP lease in the correct range (`ipconfig /all` or `ip a`).
- `Network → DHCP → Current Leases` shows active client leases.
- `nslookup <domain>` from a client resolves correctly through the firewall's DNS forwarding.

## Best Practices

- ✔ Keep a reserved static range outside the DHCP pool for infrastructure devices (switches, APs, servers).
- ✔ Use DHCP relay to a domain controller for segments requiring AD-integrated DNS registration, rather than running SFOS as DHCP server on those VLANs.
- ✔ Set conservative lease times on Guest/WiFi VLANs (shorter) versus corporate LAN (longer) to manage address churn.
- ✔ Avoid pointing internal DNS forwarding directly at public resolvers (e.g., 8.8.8.8) if internal AD-integrated domains need to resolve — forward to AD DNS first, with public DNS as secondary/fallback only.
- ✔ Document all DHCP scopes and MAC reservations in the Goinnovior IP address management (IPAM) sheet.

---

# 4. Firewall Rules

## Overview

Firewall rules in SFOS are the central policy engine — every packet crossing a zone boundary is evaluated against the rule table in top-down order. Rules also serve as attachment points for Web/App Control, IPS, ATP, and traffic shaping policies.

## Prerequisites

- Zones configured (Section 2).
- Any relevant host/network/service objects pre-created under `Hosts and Services`.

## Configuration Steps

### Step 1: Understand Rule Evaluation Order

Rules are processed top-to-bottom; the **first match wins**. Default action for unmatched traffic is **Deny**.

### Step 2: Create a Firewall Rule

Navigate: `Firewall → Add Firewall Rule → New Firewall Rule`.

| Field | Value |
|---|---|
| Rule Name | `<value>` |
| Rule Group | `<value, e.g., "LAN to WAN">` |
| Source Zone | `<value>` |
| Source Network | `<value or Any>` |
| Destination Zone | `<value>` |
| Destination Network | `<value or Any>` |
| Service | `<value or Any>` |
| Action | Accept / Drop |
| Log Traffic | Enabled |

### Step 3: Attach Security Policies to the Rule

Within the same rule, under **Security Features**:

| Feature | Setting |
|---|---|
| Intrusion Prevention | `<IPS policy — see Section 7>` |
| Malware Scanning (ATP/Sandstorm) | `<see Section 8>` |
| Web Policy | `<see Section 6>` |
| Application Control | `<see Section 6>` |
| Traffic Shaping | `<see Section 14>` |

### Step 4: Order and Organize Rules

Use **Rule Groups** to keep related rules together (e.g., "Guest WiFi Access", "DMZ Inbound", "Admin Access"). Drag rules into correct priority order — more specific rules above general/catch-all rules.

## Verification

- `Firewall` list shows rules in intended top-down order with correct hit counters incrementing (`Firewall → [Rule] → Hit Count`).
- Test traffic from source to destination matches expected Allow/Deny behavior.
- `Log Viewer → Firewall` confirms which rule ID handled a given test connection.

## Best Practices

- ✔ Always end each zone-pair rule set with an explicit final Deny-and-Log rule rather than relying on the silent implicit deny — this makes drop events visible in logs.
- ✔ Place more specific rules above broad/generic ones; SFOS does not auto-reorder by specificity.
- ✔ Avoid "Any-Any-Allow" rules entirely, even temporarily during testing — replace with scoped test rules and remove after validation.
- ✔ Name rules descriptively and group them logically so change reviews don't require re-deriving intent from IPs alone.
- ✔ Enable logging on every rule, including Drop rules — silent drops without logs make troubleshooting far harder later.
- ✔ Periodically review rules with zero hit count (`Firewall → Rule → Hits`) for cleanup candidates.

---

# 5. NAT Configuration

## Overview

SFOS handles two primary NAT scenarios: **SNAT** (source NAT, typically for internal clients reaching the internet via masquerade) and **DNAT** (destination NAT / port forwarding, for exposing internal servers to external users). Understanding the difference is essential before configuring either.

## Prerequisites

- Zones and interfaces configured (Section 2).
- Firewall rule concepts understood (Section 4) — DNAT rules require a matching firewall rule permitting the translated traffic.

## Configuration Steps

### Step 1: Default Outbound SNAT (Masquerade)

SFOS auto-creates a default MASQ rule for LAN → WAN. Confirm under: `Rules and Policies → NAT Rules`.

| Field | Value |
|---|---|
| Rule Name | `MASQ (default)` |
| Original Source | Any (LAN) |
| Translated Source | Outbound Interface (WAN) |

### Step 2: Create Custom SNAT Rule (e.g., specific subnet via specific WAN link)

Navigate: `Rules and Policies → NAT Rules → Add NAT Rule`.

| Field | Value |
|---|---|
| Rule Name | `<value>` |
| Original Source | `<value>` |
| Original Destination | Any |
| Outbound Interface | `<value>` |
| Translated Source | `<Interface IP / specific IP>` |

### Step 3: Create DNAT Rule (Port Forward / Publish Internal Server)

Navigate: `Rules and Policies → NAT Rules → Add NAT Rule → DNAT`.

| Field | Value |
|---|---|
| Rule Name | `<value>` |
| Original Destination | `<WAN IP>` |
| Original Service | `<value, e.g., HTTPS>` |
| Translated Destination | `<internal server IP>` |
| Translated Service | `<internal port, if different>` |

### Step 4: Create the Matching Firewall Rule

DNAT alone does not permit traffic — navigate to `Firewall → Add Firewall Rule` and create a WAN → DMZ (or LAN) rule allowing the translated service to the translated destination, referencing Section 4.

## Verification

- From an external network, connect to the WAN IP/port and confirm it reaches the internal server.
- `Log Viewer → Firewall` shows the NAT rule ID and correct source/translated IP pairing for the session.
- Internal clients confirm normal internet access is unaffected after any custom SNAT changes.

## Best Practices

- ✔ Never expose internal admin interfaces (SFOS WebAdmin, RDP, SSH) directly via DNAT to WAN — use VPN (Section 10/11) instead.
- ✔ Scope DNAT rules to specific services/ports only — avoid forwarding broad port ranges to a single internal host.
- ✔ Pair every DNAT rule with a correctly scoped firewall rule (not Any-source) to avoid unintentionally opening access to the entire internet.
- ✔ Use Loopback/Hairpin NAT settings where internal users need to reach a DMZ server via its public FQDN.
- ✔ Log all DNAT-related firewall rules and periodically audit exposed services against the Goinnovior asset register.

---

# 6. Web and Application Control

## Overview

Web Policy governs URL/category-based filtering (HTTP/HTTPS browsing), while Application Control governs identification and control of specific applications (e.g., TikTok, BitTorrent, TeamViewer) regardless of port. Both are attached to firewall rules and share the same policy engine in SFOS.

## Prerequisites

- Firewall rules configured (Section 4) — policies attach to rules, they are not standalone.
- Web Protection subscription active (Section 1) for full category database and cloud lookups.

## Configuration Steps

### Step 1: Create a Web Policy

Navigate: `Web → Policies → Add Policy`.

| Field | Value |
|---|---|
| Policy Name | `<value>` |
| Rule Action | Allow / Deny |
| Category | `<value, e.g., Malware/Phishing, Gambling, Social Media>` |
| Users/Groups | `<value or All>` |
| Schedule | `<value or All the Time>` |

Order category rules top-down within the policy the same way firewall rules are ordered.

### Step 2: Create an Application Control Policy

Navigate: `Applications → Application Filter → Add Filter Policy`.

| Field | Value |
|---|---|
| Policy Name | `<value>` |
| Application/Category | `<value, e.g., Proxy and Tunnel, P2P>` |
| Action | Allow / Deny |
| Micro-App Control | Enable if blocking specific features (e.g., Facebook Chat only) |

### Step 3: Attach Policies to a Firewall Rule

Navigate: `Firewall → [Rule] → Security Features`.

| Field | Value |
|---|---|
| Web Policy | `<policy from Step 1>` |
| Application Control | `<policy from Step 2>` |

### Step 4: Configure Decryption Requirement

Full HTTPS category/app enforcement requires SSL/TLS Inspection (Section 9) attached to the same rule — without it, HTTPS visibility is limited to SNI-based detection only.

## Verification

- Browsing to a blocked category returns the SFOS block page.
- `Reports → Web → Blocked Attempts` shows matching log entries.
- `Reports → Applications` shows correct app identification for known test traffic (e.g., a blocked P2P client fails to connect).

## Best Practices

- ✔ Start new category/app policies in "Log Only" or monitor mode before enforcing Deny, to catch business-critical false positives.
- ✔ Always block high-risk categories (Malware, Phishing, Command and Control) globally regardless of user group.
- ✔ Pair Application Control with SSL/TLS Inspection for accurate enforcement on HTTPS-based apps — SNI-only detection is easily evaded.
- ✔ Apply stricter policies to Guest/WiFi zones than to corporate LAN by default.
- ✔ Review `Reports → Web → Top Blocked` and `Top Allowed` monthly to refine category exceptions for Goinnovior's client-facing teams.

---

# 7. IPS Configuration

## Overview

Intrusion Prevention System (IPS) inspects traffic against signature-based rules to detect and block known exploit patterns, protocol anomalies, and attack traffic. IPS policies are attached to firewall rules, similar to Web/App Control.

## Prerequisites

- Network Protection subscription active (Section 1).
- Firewall rules configured (Section 4).

## Configuration Steps

### Step 1: Review Default IPS Policies

Navigate: `Intrusion Prevention → IPS Policies`.

| Default Policy | Typical Use |
|---|---|
| generalpolicy | Balanced default for LAN-WAN traffic |
| DMZ | Tuned for server-facing segments |
| Guest | Lighter policy for isolated guest networks |

### Step 2: Create a Custom IPS Policy (if needed)

Navigate: `Intrusion Prevention → IPS Policies → Add Policy`.

| Field | Value |
|---|---|
| Policy Name | `<value>` |
| Base Template | Copy from `generalpolicy` |
| Signature Filter | `<value, e.g., Severity: Critical/High only>` |
| Action per Signature Group | Allow / Drop and Log |

### Step 3: Attach IPS Policy to Firewall Rule

Navigate: `Firewall → [Rule] → Security Features → Intrusion Prevention`.

Select the appropriate policy (e.g., DMZ policy for WAN → DMZ rules, general policy for LAN → WAN rules).

### Step 4: Configure IPS Exceptions (if a signature causes a false positive)

Navigate: `Intrusion Prevention → DoS & Spoof Protection` or the specific policy's signature list → mark signature as **Allow** for the affected rule only, not globally.

## Verification

- `Reports → Network and Threats → IPS` shows detected/blocked signature events.
- Test with a known benign signature-triggering tool (e.g., an authorized vulnerability scan) confirms expected Drop-and-Log behavior.
- Confirm no unexpected latency or false-positive blocking on business-critical applications after policy attachment.

## Best Practices

- ✔ Apply IPS to all zone-crossing rules, not just WAN-facing ones — lateral movement between internal VLANs should also be inspected.
- ✔ Use stricter policies (all severities) on DMZ/server-facing rules; balanced policies on general LAN-WAN traffic to avoid performance impact.
- ✔ Tune exceptions narrowly (per-rule) rather than disabling entire signature categories globally.
- ✔ Review IPS reports weekly for repeated source IPs — these often indicate scanning or compromised hosts needing follow-up.
- ✔ Keep IPS signature database set to automatic updates under `Administration → Pattern Updates`.

---

# 8. ATP (Advanced Threat Protection) Configuration

## Overview

ATP correlates DNS, web, and firewall traffic against Sophos's threat intelligence feeds to detect botnet/C2 communication and known-bad destinations. It works alongside Sandstorm (cloud sandboxing) for zero-day file analysis where subscribed.

## Prerequisites

- Zero-Day Protection / ATP subscription active (Section 1).
- Firewall rules and zones configured.

## Configuration Steps

### Step 1: Enable ATP

Navigate: `Protect → Advanced Threat → Settings`.

| Field | Value |
|---|---|
| ATP Status | Enable |
| Inspection Sources | DNS, HTTP/S, Firewall |
| Policy Action | Log Only / Drop and Log |

### Step 2: Configure Source-Based Exclusions (if needed)

Navigate: `Protect → Advanced Threat → Exclusion List → Add`.

| Field | Value |
|---|---|
| Source | `<value, e.g., internal host/subnet>` |
| Destination | `<value>` |
| Reason | `<value, e.g., known-good SaaS integration falsely flagged>` |

### Step 3: Enable Sandstorm File Analysis (if subscribed)

Navigate: `Email Protection` or `Web → Sandstorm Settings` depending on traffic type.

| Field | Value |
|---|---|
| File Types to Analyze | `<value, e.g., executables, Office docs, PDFs>` |
| Action Pending Analysis | Hold until verdict / Allow and Log |

### Step 4: Review Threat Map / Dashboard

Navigate: `Control Center → Advanced Threats` for real-time detections.

## Verification

- `Reports → Advanced Threat Protection` shows any detected callbacks correlated with a source host and destination IP/domain.
- Any host flagged repeatedly is cross-checked against EDR (e.g., Kaspersky) for infection confirmation.
- Sandstorm-analyzed files show a verdict (Clean/Malicious) in `Email Protection → Sandstorm Activity` or equivalent log.

## Best Practices

- ✔ Set ATP action to **Drop and Log**, not Log Only, once tuned — detection without blocking still leaves the host communicating with C2 infrastructure.
- ✔ Investigate every ATP-flagged internal host immediately; it is a strong indicator of existing compromise, not just a policy violation.
- ✔ Correlate ATP alerts with EDR/SIEM data (Section 15) for full incident context rather than treating firewall logs in isolation.
- ✔ Keep Sandstorm enabled for all inbound email attachments and web downloads where the subscription allows.
- ✔ Review exclusion list quarterly — stale exclusions for decommissioned systems create blind spots.

---

# 9. SSL/TLS Inspection

## Overview

SSL/TLS Inspection (also called HTTPS Decryption) allows SFOS to decrypt, inspect, and re-encrypt HTTPS traffic so that Web Control, Application Control, IPS, and ATP can act on the actual content rather than just SNI metadata. This requires deploying a CA certificate to client devices.

## Prerequisites

- Web/Application Control configured (Section 6).
- Ability to distribute a CA certificate to endpoints (via AD Group Policy, MDM, or manual install).

## Configuration Steps

### Step 1: Generate or Import the Decryption CA Certificate

Navigate: `Rules and Policies → SSL/TLS Inspection Rules → Certificate Authority`.

| Field | Value |
|---|---|
| Certificate Name | `<value, e.g., Goinnovior-SFOS-CA>` |
| Type | Generate Locally / Import Existing Internal CA |
| Validity | `<value>` |

### Step 2: Distribute the CA Certificate to Clients

Export the certificate (`.pem`/`.crt`) and deploy via:

| Method | Use Case |
|---|---|
| AD Group Policy | Domain-joined Windows endpoints |
| MDM Profile | Managed mobile/laptop fleets |
| Manual Install | Small number of unmanaged devices |

### Step 3: Create an SSL/TLS Inspection Rule

Navigate: `Rules and Policies → SSL/TLS Inspection Rules → Add Rule`.

| Field | Value |
|---|---|
| Rule Name | `<value>` |
| Source Zone/Network | `<value>` |
| Destination | Any / `<value>` |
| Decrypt Action | Decrypt |
| Exclude Categories | `<value, e.g., Banking, Health — for privacy/compliance>` |

### Step 4: Configure Exclusions

Navigate: `Rules and Policies → SSL/TLS Inspection Rules → Exclusion List → Add`.

| Field | Value |
|---|---|
| Domain/Category | `<value, e.g., online banking domains>` |
| Reason | `<value>` |

## Verification

- On a client with the CA certificate installed, browsing HTTPS sites shows no certificate warning, and the certificate viewer shows the Goinnovior/SFOS CA as issuer.
- On a client without the CA installed, HTTPS sites show a certificate warning (expected, confirms inspection is active) — used to validate scope, not left in production.
- `Reports → Web` shows full category detail (not just "HTTPS") for decrypted traffic.

## Best Practices

- ✔ Always exclude legally/privacy-sensitive categories (banking, healthcare, government) from decryption per Goinnovior compliance policy.
- ✔ Deploy the CA certificate via centralized management (GPO/MDM) — never ask end users to manually trust a certificate, which trains them to click through security warnings.
- ✔ Pilot SSL inspection on a single test VLAN before enabling organization-wide, to catch app-pinning breakage (e.g., some mobile apps reject inspected certs).
- ✔ Keep an exclusion process/ticket workflow for legitimate certificate-pinned business applications that break under inspection.
- ✔ Rotate/renew the internal CA certificate before expiry and re-push to all endpoints in advance.

---

# 10. Site-to-Site VPN

## Overview

Site-to-Site VPN connects two SFOS appliances (or SFOS to a third-party firewall) over IPsec or SSL VPN tunnels, allowing transparent routing between remote office networks and Goinnovior's core site.

## Prerequisites

- Public/static WAN IPs (or FQDN via DDNS) on both ends.
- Agreed subnets for local and remote networks (no overlap).
- Shared pre-key or certificate strategy agreed between both sides.

## Configuration Steps

### Step 1: Define Local and Remote Subnets

Navigate: `Hosts and Services → IP Host → Add` — create host/network objects for local LAN and remote LAN subnets.

### Step 2: Create the IPsec Connection

Navigate: `Site-to-Site VPN → IPsec → Add`.

| Field | Value |
|---|---|
| Name | `<value>` |
| Connection Type | Site-to-Site |
| Gateway Type | Respond Only / Initiate the Connection |
| Remote Gateway IP/FQDN | `<value>` |
| Authentication Type | Preshared Key / Digital Certificate |
| Preshared Key | `<value>` |
| Local Subnet | `<value>` |
| Remote Subnet | `<value>` |

### Step 3: Configure Phase 1 and Phase 2 Parameters

| Field | Phase 1 | Phase 2 |
|---|---|---|
| Encryption | `<value, e.g., AES256>` | `<value, e.g., AES256>` |
| Authentication | `<value, e.g., SHA256>` | `<value, e.g., SHA256>` |
| DH Group | `<value, e.g., 14>` | `<value, e.g., 14>` |
| Key Life | `<value>` | `<value>` |
| PFS | Enabled | Enabled |

### Step 4: Create Matching Firewall Rules

Navigate: `Firewall → Add Firewall Rule` — create rules for `VPN → LAN` and `LAN → VPN` zones permitting the required services between the two subnets (per Section 4).

## Verification

- `Site-to-Site VPN → IPsec Connections` shows tunnel status as **Active/Up**.
- Ping/traceroute between a host on the local subnet and a host on the remote subnet succeeds.
- `Log Viewer → IPsec` shows successful Phase 1/Phase 2 negotiation with no repeated retransmits.

## Best Practices

- ✔ Match Phase 1/Phase 2 parameters exactly on both ends — mismatches are the most common cause of tunnels failing to establish.
- ✔ Use certificate-based authentication over pre-shared keys for permanent site links where PKI is feasible.
- ✔ Never use "Any-Any" firewall rules across the VPN zone — scope explicitly to required subnets and services, same as any other zone pair.
- ✔ Document all remote subnets and PSKs in a secure, access-controlled record (not plaintext in shared docs).
- ✔ Set up tunnel-down email alerts (`Administration → Notification Settings`) so outages are caught proactively, not by user complaint.

---

# 11. Remote Access / SSL VPN

## Overview

Remote Access VPN allows individual users (e.g., traveling staff, remote employees) to connect securely to Goinnovior's internal network from anywhere, using either SFOS's SSL VPN client or IPsec remote access.

## Prerequisites

- Authentication source configured (local users or AD/RADIUS — see Section 12).
- Public WAN IP/FQDN reachable by remote clients.

## Configuration Steps

### Step 1: Define the SSL VPN IP Lease Range

Navigate: `Configure → VPN → SSL VPN (Remote Access) → Settings`.

| Field | Value |
|---|---|
| IP Lease Range | `<value, e.g., 10.10.100.0/24>` |
| Protocol | UDP / TCP |
| Port | `<value, default 8443>` |
| DNS Server | `<value>` |

### Step 2: Create a User Group / Policy for Remote Access

Navigate: `Configure → VPN → SSL VPN (Remote Access) → Add`.

| Field | Value |
|---|---|
| Policy Name | `<value>` |
| Permitted Users/Groups | `<value>` |
| Permitted Network Resources | `<value, e.g., specific internal subnets only>` |

### Step 3: Assign Policy to Users

Navigate: `Authentication → Users → [User] → VPN Access` — attach the SSL VPN policy created above.

### Step 4: Distribute the SSL VPN Client Configuration

Navigate: `Configure → VPN → SSL VPN (Remote Access) → Download Client Configuration` and distribute the `.ovpn`/installer bundle to the end user, along with their credentials.

### Step 5: (Optional) Configure IPsec Remote Access

Navigate: `Configure → VPN → IPsec Connections → Add → Remote Access`, using similar authentication and lease-range settings for clients using native IPsec clients instead of SSL VPN.

## Verification

- Remote user connects via the SSL VPN client and receives an IP from the lease range.
- Remote user can reach only the internal resources explicitly permitted in the policy (test both allowed and disallowed targets).
- `Log Viewer → VPN` shows successful authentication and session establishment with correct username.

## Best Practices

- ✔ Scope each remote access policy to only the specific subnets/services a user role needs — never grant blanket LAN access by default.
- ✔ Require multi-factor authentication (via RADIUS/AD integration — Section 12) for all remote access users, not just local password auth.
- ✔ Disable or remove VPN access promptly for offboarded staff — tie this into the Goinnovior HR offboarding checklist.
- ✔ Enforce split-tunnel vs. full-tunnel deliberately based on data-handling policy, not by default configuration.
- ✔ Review active VPN sessions periodically (`Monitor & Analyze → Current Activities → VPN`) for anomalous login times or locations.

---

# 12. Authentication: AD, RADIUS, STAS

## Overview

SFOS supports multiple authentication backends for user-based policy enforcement (firewall rules, web policy, VPN access): Active Directory (AD) for domain credential lookups, RADIUS for centralized/MFA-capable authentication, and STAS (Single Try Authentication System) / SATC agent for transparent, single sign-on-style user identification without prompting for credentials.

## Prerequisites

- Reachability (routing/firewall rule) between SFOS and the AD/RADIUS server.
- Service account credentials for AD bind.
- STAS/SATC agent installer available for deployment to domain controllers (for transparent auth).

## Configuration Steps

### Step 1: Configure Active Directory (AD) Server

Navigate: `Authentication → Servers → Add → Active Directory`.

| Field | Value |
|---|---|
| Server Name | `<value>` |
| Server IP | `<value>` |
| Port | `389 / 636 (LDAPS)` |
| Bind DN (Service Account) | `<value>` |
| Password | `<value>` |
| Domain Name | `<value>` |
| Search Base DN | `<value>` |

Test connectivity using the **Test Connection** button before saving.

### Step 2: Configure RADIUS Server

Navigate: `Authentication → Servers → Add → RADIUS`.

| Field | Value |
|---|---|
| Server Name | `<value>` |
| Server IP | `<value>` |
| Authentication Port | `1812` |
| Shared Secret | `<value>` |
| Group Name Attribute | `<value, if using group-based policy>` |

### Step 3: Set Authentication Order

Navigate: `Authentication → Services → Add Server to List` — order servers (e.g., AD first, RADIUS second, Local last) matching Goinnovior's intended fallback behavior.

### Step 4: Deploy STAS for Transparent Authentication

1. Download the SATC (Sophos Authentication for Thin Client) or STAS agent from `Authentication → STAS`.
2. Install on the domain controller(s) — the agent monitors AD login events and relays user-to-IP mapping to SFOS.
3. Navigate: `Authentication → STAS → Add Client` on SFOS and register the DC's IP as an STAS collector source.

| Field | Value |
|---|---|
| Client Name | `<value, e.g., DC01>` |
| Client IP | `<value>` |
| Shared Key | `<value>` |

### Step 5: Enable User-Based Policy Enforcement

Navigate: `Firewall → [Rule]` — set **Source** as a user/group (via matched AD/RADIUS identity) instead of only network-based, once STAS/AD identification is active.

## Verification

- `Authentication → Servers → [Server] → Test Connection` returns success.
- A domain user logging into a workstation is transparently identified in `Monitor & Analyze → Current Activities → Live Users` without a captive portal prompt (STAS working).
- RADIUS-based VPN or admin logins succeed and correctly reflect group membership in applied policy.

## Best Practices

- ✔ Use a dedicated, least-privilege service account for the AD bind — never use a Domain Admin account for LDAP bind.
- ✔ Prefer LDAPS (636) over plaintext LDAP (389) for the AD bind connection where the DC supports it.
- ✔ Deploy STAS on all domain controllers, not just one, so authentication continues if a DC is unavailable.
- ✔ Enforce RADIUS-based MFA for remote access and admin console logins, not just standard user policy lookups.
- ✔ Periodically audit `Authentication → Users` for stale/local accounts that should be migrated to AD-based identity.

---

# 13. High Availability (HA)

## Overview

HA pairs two identical XGS appliances (Active-Passive or Active-Active, model dependent) so that a hardware failure on the primary unit fails over to the secondary with minimal disruption. This section covers Active-Passive HA, the most common Goinnovior deployment pattern.

## Prerequisites

- Two identical XGS appliances, same model and firmware version.
- A dedicated HA link (dedicated port or crossover cable) between the two units.
- Both units licensed/registered individually before pairing.

## Configuration Steps

### Step 1: Cable the HA Link

Connect a dedicated port (e.g., Port4) directly between both appliances for the HA heartbeat/sync link.

### Step 2: Configure HA on the Primary Unit

Navigate: `System → High Availability → Configure HA`.

| Field | Value |
|---|---|
| HA Role | Primary |
| HA Mode | Active-Passive |
| Dedicated HA Link Port | `<value, e.g., Port4>` |
| HA Link IP | `<value, e.g., 169.254.1.1/30>` |

### Step 3: Configure HA on the Secondary Unit

Repeat with:

| Field | Value |
|---|---|
| HA Role | Auxiliary/Secondary |
| HA Link IP | `<value, e.g., 169.254.1.2/30>` |

### Step 4: Confirm Configuration Sync

Once paired, the secondary syncs configuration automatically from the primary. Navigate: `System → High Availability` on either unit to confirm **Sync Status: In Sync**.

### Step 5: Test Failover

Navigate: `System → High Availability → [Primary] → Force HA Failover` in a maintenance window to validate failover behavior before relying on it in production.

## Verification

- `System → High Availability` shows both units, correct roles (Primary/Auxiliary), and status **Ready/Sync Complete**.
- Forced failover test results in the secondary assuming Active role with no manual intervention required, and traffic resumes within expected downtime window.
- Both units show identical firmware version and licensing status.

## Best Practices

- ✔ Use a dedicated physical HA link, not a shared/switched network segment, to avoid heartbeat delays affecting failover timing.
- ✔ Keep both units on identical firmware at all times — mismatched firmware can block HA pairing or cause instability.
- ✔ Schedule and document a controlled failover test at planned intervals (e.g., quarterly) rather than only discovering failover behavior during an actual outage.
- ✔ Monitor HA status via SNMP/email alerts (`Administration → Notification Settings`) so a silent split-brain or sync failure doesn't go unnoticed.
- ✔ Ensure redundant power and, where possible, redundant WAN uplinks feed each HA unit — HA protects against hardware failure, not upstream link failure, unless paired with link redundancy.

---

# 14. Traffic Shaping / QoS

## Overview

Traffic Shaping policies control bandwidth allocation per user, group, application, or firewall rule — preventing any single traffic type (e.g., backups, streaming) from starving business-critical applications (e.g., VoIP, ERP access).

## Prerequisites

- Firewall rules configured (Section 4) — shaping policies attach to rules.
- Baseline understanding of available WAN bandwidth for Goinnovior's site.

## Configuration Steps

### Step 1: Create a Traffic Shaping Policy

Navigate: `Rules and Policies → Traffic Shaping → Add`.

| Field | Value |
|---|---|
| Policy Name | `<value>` |
| Type | Total Bandwidth / Per User / Per Rule |
| Guaranteed Bandwidth (Upload/Download) | `<value>` |
| Maximum Bandwidth (Upload/Download) | `<value>` |
| Priority | `<value, 1 (highest) – 7 (lowest)>` |

### Step 2: Attach Policy to a Firewall Rule

Navigate: `Firewall → [Rule] → Security Features → Traffic Shaping Policy` — select the policy created above.

### Step 3: Configure Application-Based Shaping (Optional)

Navigate: `Applications → Application Filter` — combine with a Traffic Shaping policy so specific applications (e.g., cloud backup, streaming) are capped independent of the general rule bandwidth.

### Step 4: Set Global Bandwidth Limits (if using multiple WAN links)

Navigate: `Network → Interfaces → [WAN Interface] → Bandwidth` — define the actual physical link capacity so SFOS calculates shaping percentages correctly.

## Verification

- `Reports → Traffic Shaping` shows bandwidth allocation matching configured guarantees/limits during peak usage.
- A bandwidth-heavy test transfer (e.g., large file download) on a shaped rule does not exceed the configured maximum, and priority traffic (e.g., VoIP test call) remains unaffected.
- `Monitor & Analyze → Current Activities → Bandwidth` shows per-user/per-app consumption in real time.

## Best Practices

- ✔ Set WAN interface bandwidth values accurately (`Network → Interfaces`) — shaping percentages are meaningless if the base link capacity is misconfigured.
- ✔ Prioritize latency-sensitive traffic (VoIP, video conferencing) with higher priority values over bulk transfer traffic (backups, updates).
- ✔ Apply per-user shaping on Guest/WiFi zones to prevent a single device from consuming the shared uplink.
- ✔ Avoid over-provisioning guaranteed bandwidth across all policies combined beyond actual link capacity — guarantees are only meaningful if the sum stays within real capacity.
- ✔ Review shaping reports monthly and adjust as business application needs change (e.g., new cloud ERP rollout).

---

# 15. Logging, Reporting, and SIEM Integration

## Overview

SFOS generates detailed logs across firewall, IPS, ATP, web, and VPN subsystems. This section covers configuring log storage, on-box reporting, and forwarding logs to an external SIEM (e.g., Wazuh, Sentinel) for centralized correlation — a core part of Goinnovior's managed security services offering.

## Prerequisites

- Correct time zone/NTP configured (Section 1) — critical for accurate log correlation.
- Destination SIEM/syslog server reachable from SFOS over the network.

## Configuration Steps

### Step 1: Configure On-Box Log Settings

Navigate: `Configure → System Services → Log Settings`.

| Log Type | Enable |
|---|---|
| Firewall Rule Logs | Yes |
| IPS Logs | Yes |
| Web Filter Logs | Yes |
| ATP Logs | Yes |
| VPN Logs | Yes |
| Admin/System Events | Yes |

### Step 2: Configure Syslog Forwarding to SIEM

Navigate: `Configure → System Services → Log Settings → Add Syslog Server`.

| Field | Value |
|---|---|
| Name | `<value>` |
| Server IP | `<value, e.g., Wazuh manager IP>` |
| Port | `<value, default 514>` |
| Protocol | UDP / TCP / TLS |
| Facility | `<value>` |
| Log Format | Device Standard / CEF |

### Step 3: Select Which Log Types Forward to Syslog

Within the same syslog server configuration, enable individual log categories to forward (Firewall, IPS, Web, ATP, VPN, Authentication) matching what the SIEM parser expects.

### Step 4: Configure On-Box Reports

Navigate: `Reports → My Reports` — bookmark commonly reviewed reports (Top Blocked Web Categories, Top IPS Attacks, VPN Usage) for scheduled email delivery.

Navigate: `Reports → Report Scheduling → Add` to automate periodic report delivery to Goinnovior's security team inbox.

## Verification

- SIEM (e.g., Wazuh) shows incoming events from the SFOS device IP, correctly parsed and timestamped.
- Test event (e.g., a deliberate blocked connection) appears in both `Log Viewer` on SFOS and the SIEM dashboard within expected latency.
- Scheduled reports arrive via email at the configured interval with correct date ranges.

## Best Practices

- ✔ Use TLS-encrypted syslog forwarding where the SIEM supports it, rather than plaintext UDP, especially over any network segment beyond a trusted local link.
- ✔ Forward all major log categories (Firewall, IPS, ATP, Web, VPN, Auth) to the SIEM — partial logging creates investigation blind spots during incident response.
- ✔ Set on-box log storage retention as long as disk allows, but treat the SIEM as the authoritative long-term retention source, not the appliance itself.
- ✔ Confirm NTP sync across SFOS, the SIEM server, and all correlated systems (AD, EDR) — time drift breaks event correlation during investigations.
- ✔ Schedule regular (e.g., weekly) review of Top Attacks / Top Blocked reports as part of Goinnovior's managed service deliverables, not just ad hoc lookups during incidents.

---

# 16. Backup, Restore, and Firmware Upgrades

## Overview

This section covers configuration backup scheduling, disaster restore procedures, and safe firmware upgrade practices — critical maintenance operations that must be handled carefully to avoid unplanned downtime on production Goinnovior client deployments.

## Prerequisites

- Administrative access to SFOS WebAdmin.
- Secure storage location for backup files (encrypted, access-controlled — e.g., an internal file share or Sophos Central).

## Configuration Steps

### Step 1: Configure Scheduled Backups

Navigate: `Backup & Firmware → Backup → Schedule Backup`.

| Field | Value |
|---|---|
| Frequency | Daily / Weekly |
| Time | `<value>` |
| Backup Mode | Send via Email / FTP / Cloud (Central) |
| Encryption Password | `<value>` |

### Step 2: Take a Manual Backup Before Any Change Window

Navigate: `Backup & Firmware → Backup → Backup Now` — always take a manual backup immediately before firmware upgrades or major rule/policy changes.

### Step 3: Restore from Backup

Navigate: `Backup & Firmware → Restore → Upload Backup File`.

| Field | Value |
|---|---|
| Backup File | `<value>` |
| Encryption Password | `<value>` |

Confirm restore, expecting an automatic reboot.

### Step 4: Firmware Upgrade Procedure

1. Navigate: `Backup & Firmware → Firmware` and review available versions.
2. Take a manual backup (Step 2) and download it locally before proceeding.
3. Click **Upload & Boot** or **Upload & Flash** on the target firmware version.
4. **For HA pairs:** upgrade the Auxiliary unit first, confirm sync, then fail over and upgrade the former Primary.

### Step 5: Confirm Post-Upgrade Health

After reboot, check `System → Health` and confirm all subscriptions, interfaces, and HA (if applicable) show normal/expected status.

## Verification

- Scheduled backup files arrive at the configured destination on schedule and are restorable (test-restore periodically in a lab unit if available).
- Post-upgrade, `Backup & Firmware → Firmware` shows the new version as active and the previous version retained as fallback.
- All firewall rules, NAT rules, and VPN tunnels remain intact and functioning identically after upgrade.

## Best Practices

- ✔ Always take a manual backup immediately before any firmware upgrade or major configuration change, even with scheduled backups already in place.
- ✔ Store backup encryption passwords in Goinnovior's password manager, never in the backup filename or an unencrypted note.
- ✔ Read the firmware release notes fully before upgrading — check for known issues affecting features in active use (VPN, HA, specific hardware models).
- ✔ Upgrade during a planned maintenance window, never during business hours, even for HA pairs where downtime is expected to be minimal.
- ✔ Retain at least the last 2–3 firmware versions' backups and keep an offline (non-appliance-stored) copy of the most recent backup at all times.

---

# 17. Sophos Central Management

## Overview

Sophos Central provides centralized visibility and management across multiple XGS appliances — useful for Goinnovior's managed services model where multiple client firewalls need unified monitoring, policy templates, and alerting from a single console.

## Prerequisites

- Device registered to Sophos Central (Section 1).
- Sophos Central Firewall Manager (or equivalent licensing tier) access.

## Configuration Steps

### Step 1: Confirm Device Enrollment

In Sophos Central: navigate to `Devices → Firewall` and confirm the appliance appears with status **Managed** and correct device group assignment.

### Step 2: Organize Devices into Groups

Navigate: `Devices → Firewall → Device Groups → Add Group` — group by client or site (e.g., "Client A – HQ", "Client B – Branch").

### Step 3: Apply Central-Managed Policies (if using centralized policy push)

Navigate: `Firewall Management → Templates` — create reusable policy templates (Firewall Rules, IPS, Web Policy baselines) and apply to relevant device groups for consistent baseline configuration across client deployments.

### Step 4: Configure Central Alerting

Navigate: `My Products → Firewall → Alerts` — set thresholds and notification channels (email, integrations) for events like HA failover, license expiry, or high threat activity.

### Step 5: Review Central Dashboard

Navigate: `Home` or `Firewall Dashboard` in Central for a cross-device summary view — useful for daily managed-service health checks across all Goinnovior client firewalls.

## Verification

- All managed appliances show **Online** and **Managed** status in Central with recent last-seen timestamps.
- A test policy template pushed from Central correctly appears on the target appliance's local WebAdmin.
- Alert test (e.g., simulated license-expiry threshold) generates the expected notification.

## Best Practices

- ✔ Maintain a clear device group/naming convention per client to avoid cross-client configuration errors when using templates.
- ✔ Use Central for cross-device monitoring and alerting even if local WebAdmin remains the primary configuration point for granular per-client rules.
- ✔ Restrict Central console access (RBAC) so only authorized Goinnovior engineers can push templated changes across client fleets.
- ✔ Regularly review the Central dashboard as part of daily managed-service operations, not only when an alert fires.
- ✔ Keep a documented change log for any template pushed via Central, since it affects multiple client appliances simultaneously.

---

# 18. Troubleshooting and CLI Diagnostics

## Overview

This section covers the SFOS Command Line Interface (CLI) console and common diagnostic workflows for troubleshooting connectivity, VPN, HA, and performance issues beyond what the WebAdmin GUI surfaces directly.

## Prerequisites

- Console/SSH access to the appliance (`Administration → Device Access` must permit SSH from the admin's network).
- Admin credentials.

## Configuration Steps

### Step 1: Access the CLI Console

Connect via SSH (`ssh admin@<firewall IP>`) or the physical console port. On login, the SFOS CLI menu appears with numbered options (Network Configuration, System Configuration, Diagnostics, etc.).

### Step 2: Common Diagnostic Commands (via CLI Console → Advanced Shell where applicable)

| Task | Command / Menu Path |
|---|---|
| Check interface status | CLI Menu → Network Configuration → Show Interfaces |
| Ping test | CLI Menu → Diagnostics → Ping |
| Traceroute | CLI Menu → Diagnostics → Traceroute |
| Packet capture | CLI Menu → Diagnostics → Packet Capture |
| View routing table | CLI Menu → Network Configuration → Show Routes |
| Check HA status | CLI Menu → System Configuration → Show HA State |
| Restart network service | CLI Menu → System Configuration → Restart Network Service |

### Step 3: Use WebAdmin Packet Capture (GUI alternative)

Navigate: `Diagnostics → Packet Capture → Start Capture`.

| Field | Value |
|---|---|
| Interface | `<value>` |
| Filter | `<value, e.g., host <IP> and port <port>>` |
| Duration | `<value>` |

Download the resulting `.pcap` for analysis in Wireshark.

### Step 4: Check System Health and Resource Usage

Navigate: `System → Current Activities → System Health` (GUI) or CLI Menu → **System Diagnostics → Show CPU/Memory Usage** for identifying resource exhaustion during performance issues.

### Step 5: Review Consolidated Logs for the Incident Window

Navigate: `Log Viewer` (GUI) and filter by module (Firewall, IPS, VPN, System) and timestamp range matching the reported issue.

## Verification

- CLI console commands return expected output without permission or connectivity errors.
- Packet captures confirm expected traffic flow direction and content for the diagnosed issue.
- Root cause identified and documented before applying a fix — avoid changing multiple variables (rules, routes, NAT) simultaneously during troubleshooting.

## Best Practices

- ✔ Always take a configuration backup (Section 16) before making any troubleshooting-driven changes, in case a rollback is needed.
- ✔ Use packet captures with specific filters (host/port) rather than capturing all traffic, to keep capture files manageable and relevant.
- ✔ Document root cause and resolution for every significant incident in Goinnovior's internal ticketing/knowledge base — recurring issues should trigger a permanent fix, not repeated manual workarounds.
- ✔ Restrict CLI/SSH access to specific admin source IPs (`Administration → Device Access`) rather than leaving it open to the full LAN.
- ✔ Escalate to Sophos Support with a diagnostic file (`Help → Generate Diagnostic File`) for issues beyond standard troubleshooting steps, rather than prolonged trial-and-error on a production appliance.

---

*End of document — Sophos XGS Firewall Configuration (SFOS), Goinnovior Internal Infrastructure.*
