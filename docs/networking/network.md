# Network Architecture

This document describes the logical network design for the ACME Corporation enterprise environment.

The network is implemented virtually using VMware networking and firewall routing. It represents an enterprise network model without requiring a physical managed-switch environment.

---

## 1. Network Goals

The network design covers:

- IPv4 addressing
- Subnetting
- VLANs
- Routing
- NAT
- Firewall policy
- Network segmentation
- DMZ design
- VPN access
- DHCP
- DNS
- Traffic logging
- Security boundaries
- Network troubleshooting

The main security principle is:

> **Traffic should be explicitly permitted when required rather than broadly trusted by default.**

---

## 2. Topology

```text
                         INTERNET
                             |
                             v
                     HOME ROUTER / NAT
                             |
                             v
                    VMware Virtual Network
                             |
                             v
                        +---------+
                        |  FW01   |
                        | Firewall|
                        | Router  |
                        | VPN     |
                        +----+----+
                             |
                 +-----------+-----------+
                 |           |           |
                 v           v           v
             MANAGEMENT   SERVERS      CLIENTS
                 |           |           |
                 +-----------+-----------+
                             |
                    Additional Zones
                             |
          +----------+-------+-------+----------+
          |          |               |          |
         DMZ      SECURITY        BACKUP      VPN
          |          |               |          |
        WEB01     MON/WAZUH       BACKUP01   Remote Users

                         GUEST
                           |
                           v
                    Internet only
```

The physical network is abstracted through VMware virtual networking.

---

## 3. VLANs and Subnets

| VLAN | Name | Subnet | Purpose |
|---:|---|---|---|
| 10 | MANAGEMENT | `192.168.10.0/24` | Administrative systems and management access |
| 20 | SERVERS | `192.168.20.0/24` | Core infrastructure and application servers |
| 30 | CLIENTS | `192.168.30.0/24` | Domain-joined workstations and endpoints |
| 40 | DMZ | `192.168.40.0/24` | Public-facing or externally exposed services |
| 50 | SECURITY | `192.168.50.0/24` | Monitoring, logging, and security systems |
| 60 | BACKUP | `192.168.60.0/24` | Backup infrastructure and protected backup traffic |
| 70 | VPN | `192.168.70.0/24` | Remote-access VPN clients |
| 80 | GUEST | `192.168.80.0/24` | Untrusted guest access |

These networks are logical security zones.

---

## 4. Security Zones

### Management - VLAN 10

Used for administrative access.

Examples:

- Administrative workstation
- Management interfaces
- Windows Admin Center
- RSAT
- Infrastructure administration

Access is restricted to authorized administrators.

### Servers - VLAN 20

Contains core enterprise services:

- Domain controllers
- File services
- Application services
- Database services
- Management services

The server network is not treated as a flat trusted network.

### Clients - VLAN 30

Contains domain-joined endpoints such as WIN11-01 and WIN11-02.

Clients require selected services including:

```text
DNS
DHCP
Kerberos
LDAP
SMB
HTTPS
Configuration Manager
WSUS
```

They do not receive unrestricted access to every server.

### DMZ - VLAN 40

Used for services with possible external exposure.

Example:

```text
WEB01
```

A compromised DMZ host should not automatically have access to internal servers.

### Security - VLAN 50

Used for:

- Prometheus
- Grafana
- Wazuh
- Syslog
- Security monitoring

Monitoring systems receive only the access required to collect their data.

### Backup - VLAN 60

Used for backup infrastructure and protected backup traffic.

Backup systems are treated as a separate security boundary rather than ordinary file servers.

### VPN - VLAN 70

Remote users terminate into the VPN network.

VPN access is controlled by firewall rules and, where applicable, RADIUS/NPS authentication.

### Guest - VLAN 80

Guest devices are untrusted.

```text
Guest
  |
  +----> Internet
  |
  X----> Management
  X----> Servers
  X----> Clients
  X----> Security
  X----> Backup
```

---

## 5. Firewall Architecture

FW01 is the central security boundary.

Responsibilities:

- Inter-VLAN routing
- NAT
- Firewall policy
- VPN termination
- DHCP relay where required
- DNS forwarding
- Traffic logging

Conceptual default policy:

```text
Internet -> Internal          DENY
Guest -> Internal             DENY
DMZ -> Internal               DENY by default
Clients -> Management         DENY by default
VPN -> Internal               ALLOW only required resources
Servers -> Internet            ALLOW only required traffic
Management -> Infrastructure  ALLOW required administration
```

Firewall rules should record:

- Source
- Destination
- Protocol
- Port
- Direction
- Purpose
- Security justification

---

## 6. Required Service Flows

### Client to DNS

```text
CLIENTS -> DC01
UDP/TCP 53
```

### Client to DHCP

```text
CLIENTS -> DHCP service
UDP 67/68
```

### Domain Authentication

Typical services include:

```text
Kerberos
LDAP
DNS
SMB
RPC / dynamic RPC
```

The exact ports depend on the Windows service being exercised.

### Client to File Server

```text
CLIENTS -> FILE01
TCP 445
```

### Client to Web Service

```text
CLIENTS -> WEB01
TCP 443
```

### VPN to Internal Services

VPN users receive access only to the networks and services required by their role.

---

## 7. NAT

FW01 provides NAT between the internal virtual environment and the upstream network.

```text
Internal networks
      |
      v
    FW01
      |
     NAT
      |
      v
 VMware / Home Network
      |
      v
  Internet
```

NAT is not a replacement for firewall security.

---

## 8. DNS

DNS is a critical dependency for the Windows environment.

The Active Directory domain is:

```text
acme.local
```

DC01 provides internal DNS.

Future redundancy:

```text
DC01
DC02
```

DNS troubleshooting includes:

- Client DNS configuration
- DNS server availability
- Forwarders
- A records
- PTR records
- AD-integrated zones
- Name resolution across VLANs
- Firewall rules
- Time synchronization

---

## 9. DHCP

A logical DHCP scope for the client network:

```text
Network: 192.168.30.0/24
Gateway: 192.168.30.1
DNS:     DC01
Domain:  acme.local
```

Other VLANs may have their own scopes.

DHCP relay can be used when the DHCP server is located on another subnet.

---

## 10. VPN

The VPN network is:

```text
192.168.70.0/24
```

The project covers:

- WireGuard
- OpenVPN / IPsec concepts
- Authentication
- Certificate-based authentication
- RADIUS / NPS integration
- Split tunnel
- Full tunnel
- VPN firewall policies
- Remote administration

Authentication flow:

```text
VPN Client
    |
    v
FW01 / VPN
    |
    v
NPS01 / RADIUS
    |
    v
Active Directory
```

The exact implementation depends on the VPN technology being tested.

---

## 11. DMZ

The DMZ is:

```text
192.168.40.0/24
```

Example placement:

```text
WEB01
```

Traffic from a DMZ workload to internal services is restricted to the specific flows required by the application.

```text
Internet
   |
   v
 FW01
   |
   v
 WEB01 / DMZ
   |
   X
Internal Servers
```

---

## 12. Monitoring and Logging Traffic

Monitoring systems require controlled access to monitored endpoints.

Examples:

```text
MON01 -> Windows exporters
MON01 -> Linux exporters
MON01 -> HTTP endpoints
MON01 -> network devices/services
```

Security monitoring may use:

```text
WAZUH01 -> Windows event collection
WAZUH01 -> Linux agents
SYSLOG01 <- network/system logs
```

These flows are restricted to the required sources and destinations.

---

## 13. Network Security Model

### Least Privilege

Allow only the access required by each service.

### Segmentation

Separate:

```text
Management
Servers
Clients
DMZ
Security
Backup
VPN
Guest
```

### Default Deny

New inter-zone traffic is denied until its purpose is defined.

### Administrative Separation

Administrative access should originate from designated management systems where practical.

### Logging

Important firewall and authentication events are logged centrally.

---

## 14. Troubleshooting Method

Network troubleshooting follows a layered approach:

```text
1. Physical / Virtual Network
        ↓
2. Interface State
        ↓
3. IP Address
        ↓
4. Subnet / Gateway
        ↓
5. ARP / Neighbor Resolution
        ↓
6. Routing
        ↓
7. Firewall
        ↓
8. DNS
        ↓
9. TCP / UDP Port
        ↓
10. Application / Service
```

### Windows

```powershell
ipconfig /all
ping
tracert
nslookup
Test-NetConnection
Get-NetIPConfiguration
Get-NetRoute
Get-NetTCPConnection
```

### Linux

```bash
ip addr
ip route
ping
traceroute
dig
ss
curl
nc
```

### Firewall

Useful information includes:

- Interface status
- Routing table
- ARP table
- Firewall logs
- NAT state
- VPN status
- Packet capture

---

## 15. Failure Scenarios

### DNS Failure

Symptoms:

- Domain authentication problems
- Websites unavailable by name
- `nslookup` failures

Investigation:

```text
Client configuration
    ↓
DNS reachability
    ↓
DNS records
    ↓
Forwarders
    ↓
Firewall
```

### DHCP Failure

Symptoms:

- APIPA address
- Incorrect gateway
- No DNS configuration

Investigation:

```text
Client
    ↓
DHCP broadcast
    ↓
VLAN / relay
    ↓
DHCP server
    ↓
Scope availability
```

### Firewall Rule Failure

Symptoms:

- TCP connection refused or timed out
- Service works from one VLAN but not another

Investigation:

```text
Source
Destination
Route
Firewall rule
NAT
Listening service
```

### VPN Failure

Investigate:

- VPN service status
- Authentication
- RADIUS/NPS
- Certificate validity
- Routing
- Firewall policy
- Client configuration

---

## 16. Network Documentation Standard

Significant firewall and network changes should record:

```text
Change
Purpose
Source
Destination
Protocol
Port
Expected behavior
Security impact
Testing
Rollback
```

---

## 17. Network Diagram

The visual network diagram emphasizes the major trust boundaries and traffic flows:

```text
Internet
   |
FW01
   |
+-------------------------------+
| VLANs / Security Zones        |
|                               |
| 10 Management                 |
| 20 Servers                    |
| 30 Clients                    |
| 40 DMZ                        |
| 50 Security                   |
| 60 Backup                     |
| 70 VPN                        |
| 80 Guest                      |
+-------------------------------+
```

See [`diagrams/network.png`](../../diagrams/network.png).

---

## 18. Future Expansion

The network can later incorporate:

- Additional domain controller
- Dedicated management workstation
- Dedicated monitoring network
- Dedicated backup network
- Additional Linux systems
- Additional clients
- More detailed firewall policies
- IDS/IPS concepts
- Network packet capture
- Advanced VPN policies
- Cloud networking
- Azure VNet / NSG concepts
- AWS VPC / Security Group concepts

The current virtual implementation remains smaller than the logical enterprise design because of the available hardware.