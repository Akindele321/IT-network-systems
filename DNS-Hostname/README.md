# DNS Hostname Resolution (R1/R2/R3)

## Overview
This Packet Tracer lab demonstrates **basic DNS hostname resolution** in a routed environment.

Goal:
- Configure a DNS server
- Configure routers with IP addressing and routing
- Configure R1 to use DNS (`ip name-server`)
- Verify hostname resolution by pinging a router hostname (e.g., `ping R2`)

### Devices
- R1, R2, R3 (routers)
- DNS-Server
- Two switches (SW1, SW2)

## What I Configured

### 1) DNS Server
Configured the DNS server with:
- IP address: `10.10.10.10`
- Default gateway: `10.10.10.2`

---

### 2) R1 DNS Client Settings
On R1, I configured DNS name resolution:

## Screenshots
