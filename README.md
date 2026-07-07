# Sophos XGS45 Firewall Configuration
## Goinnovior Internal Infrastructure

---

# Overview

This repository contains the complete documentation, deployment notes, configuration references, and operational procedures for the Sophos XGS45 Firewall deployed within the Goinnovior office environment.

The repository serves as:

- Infrastructure documentation
- Disaster Recovery reference
- Firewall deployment guide
- Network inventory
- Operational handbook

---

# Environment Information

| Item | Value |
|------|------|
| Organization | Goinnovior Ltd |
| Firewall | Sophos XGS45 |
| Deployment | Production |
| Firewall OS | SFOS |
| Management | Web Console |
| Wireless Controller | Grandstream GDMS |

---

# Network Overview

## WAN

| Setting | Value |
|---------|---------|
| Interface | Port2 |
| Type | Static |
| Public IP |  |
| Gateway | ISP Gateway |
| DNS 1 | 8.8.8.8 |
| DNS 2 |  |

---

## LAN (Primary)

| Setting | Value |
|---------|---------|
| Interface | Port1 |
| IP Address | 172.16.16.16 |
| Subnet | 255.255.255.0 (/24) |
| DHCP Range | 172.16.16.17 - 172.16.16.254 |

---

## LAN (Secondary)

| Setting | Value |
|---------|---------|
| Interface | Port4 |
| IP Address | 192.168.1.1 |
| Subnet | 255.255.255.0 (/24) |
| DHCP Range | 192.168.1.100 - 192.168.1.250 |

---

# Interface Layout

| Port | Purpose |
|------|----------|
| Port1 | Internal LAN |
| Port2 | Internet WAN |
| Port3 | Unused |
| Port4 | Secondary LAN |

---

# DHCP Configuration

## Port1

```
Start IP : 172.16.16.17
End IP   : 172.16.16.254
Subnet   : 255.255.255.0
Gateway  : 172.16.16.16
```

## Port4

```
Start IP : 192.168.1.100
End IP   : 192.168.1.250
Gateway  : 192.168.1.1
```

---

# DNS

Primary

```
8.8.8.8
```

Secondary

```
103.41.213.2
```

---

# Wireless Infrastructure

## Managed AP

Grandstream Access Point

Controller

GDMS Networking

Configured SSIDs

| SSID | Purpose |
|------|----------|
| GIL ~ Home | Internal |
| GIL ~ Employee | Employee |
| GIL ~ Guest | Guest |
| 360D ~ Employee | Employee |

---

# Legacy Equipment

## Netis Router

Purpose

Layer-2 Switch

Issue Found

The device was unintentionally broadcasting:

```
Codeinnovior
Codeinnovior_2.4G
360DSoul_5G
```

Investigation confirmed that:

- WiFi remained enabled
- DHCP disabled
- Connected as switch
- Broadcast disappeared after powering off Netis

Recommendation

- Disable both WiFi radios
- Disable WPS
- Replace with unmanaged switch

---

# Initial Firewall Deployment

## Step 1

Connect laptop to Port1

Browse

```
https://172.16.16.16:4444
```

Complete

- Accept EULA
- Set Administrator Password
- Set Master Key
- Reboot

---

## Step 2

Configure Interfaces

Network

```
Network
→ Interfaces
```

Configure

Port1

```
172.16.16.16/24
```

Port2

```
WAN
Static IP
```

Port4

```
192.168.1.1/24
```

---

## Step 3

Configure DHCP

```
Network
→ DHCP
```

Configure

Port1

```
172.16.16.17-254
```

Port4

```
192.168.1.100-250
```

---

## Step 4

Configure Firewall Rule

```
LAN → WAN
```

Allow

Initially

```
Application Control
Allow

Web Filter
Allow
```

---

# Sophos Registration

Administration

```
Licensing
```

Register

Activate

Synchronize with Sophos Central

---

# Security Recommendations

✔ Strong administrator password

✔ Store Master Key securely

✔ Enable automatic backup

✔ Configure email alerts

✔ Enable IPS

✔ Enable ATP

✔ Enable Application Control

✔ Enable Web Protection

✔ Configure SSL Inspection

✔ Configure MFA

---

