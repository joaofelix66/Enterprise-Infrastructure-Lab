# Enterprise Architecture

This document describes the logical enterprise architecture, the infrastructure roles, service dependencies, and the current VMware implementation.

The environment represents a fictional **ACME Corporation** with approximately 100 employees.

---

## 1. Architecture Principles

1. Separate infrastructure responsibilities logically.
2. Define service dependencies.
3. Apply least privilege and network segmentation.
4. Monitor and log important infrastructure.
5. Automate repetitive administration.
6. Design for backup and recovery.
7. Keep the logical architecture independent from current hardware limitations.

```text
TARGET ENTERPRISE ARCHITECTURE
            |
            v
Logical infrastructure roles
            |
            v
Current virtualization strategy
            |
            v
Resource-constrained VM implementation
```

A logical server role does not necessarily mean a dedicated VM on the current host.

---

## 2. Logical Server Inventory

| System | Logical Role | Main Responsibilities |
|---|---|---|
| FW01 | Firewall / Router | Routing, NAT, firewalling, VPN, traffic control |
| DC01 | Primary Domain Controller | AD DS, DNS, DHCP, GPO |
| DC02 | Secondary Domain Controller | AD replication, secondary DNS, resilience, FSMO/failure exercises |
| FILE01 | File Services | SMB, NTFS, ABE, quotas, shadow copies |
| WEB01 | Web / DMZ | IIS or Nginx, TLS, reverse proxy, web workloads |
| SQL01 | Database | SQL Server and application databases |
| SCCM01 | Endpoint Management | Microsoft Configuration Manager / MECM |
| WSUS01 | Patch Management | Windows Update and patch management |
| PKI01 | Internal PKI | AD CS, certificate templates, auto-enrollment, CRL |
| NPS01 | Network Authentication | NPS / RADIUS, VPN authentication |
| MON01 | Monitoring | Prometheus, Grafana, exporters, alerting |
| WAZUH01 | Security Monitoring | Event analysis, FIM, security alerts |
| SYSLOG01 | Central Logging | Centralized infrastructure logs |
| BACKUP01 | Backup | Backup jobs, retention, restore testing |
| LINUX01 | Linux / Container Platform | SSH, Nginx, Docker, Compose and Linux workloads |
| WIN11-01 | Endpoint / Admin | Corporate endpoint, RSAT, administration and testing |
| WIN11-02 | Endpoint | Additional domain-joined client and testing workload |

These roles form the target logical architecture.

---

## 3. Current VM Implementation

The current host provides 16 GB of RAM, so several logical roles are consolidated.

| VM | RAM | Current Functions |
|---|---:|---|
| FW01 | 1.5 GB | OPNsense, firewall, routing, VLANs, NAT, VPN |
| DC01 | 2 GB | AD DS, DNS, DHCP, GPO, AD CS/PKI, NPS/RADIUS |
| FILE01 | 2 GB | SMB, NTFS, ABE, quotas, shadow copies, WSUS |
| LINUX01 | 2.5 GB | Linux, Docker, Nginx, Prometheus, Grafana, Syslog, security workloads |
| WIN11-01 | 4 GB | Corporate endpoint, administration, RSAT, WAC |
| SCCM01 | 6 GB | Microsoft Configuration Manager / MECM and SQL |

The total allocation is greater than 16 GB, so these machines are used in different resource profiles rather than all running at once.

---

## 4. Role Consolidation

### DC01

Current services:

```text
Active Directory
DNS
DHCP
Group Policy
AD CS / PKI
NPS / RADIUS
```

These roles can be separated as the environment grows.

### FILE01

Current services:

```text
File Services
WSUS
```

File-service work covers SMB, NTFS permissions, ABE, quotas, and shadow copies.

### LINUX01

LINUX01 is the primary Linux platform:

```text
Linux
 ├── SSH
 ├── Nginx
 ├── Docker
 ├── Prometheus
 ├── Grafana
 ├── Syslog
 └── Security workloads
```

Security workloads such as Wazuh are enabled when required rather than treated as part of the normal 2.5 GB baseline.

### WIN11-01

WIN11-01 is used as both a domain-joined endpoint and an administrative workstation.

Typical uses:

- RSAT
- Windows Admin Center
- GPO testing
- Authentication testing
- Configuration Manager client testing
- Administrative tooling

### SCCM01

SCCM01 combines:

```text
Microsoft Configuration Manager / MECM
SQL Server
```

It is treated as a heavy workload and is normally powered off when Configuration Manager is not being studied.

---

## 5. Resource Profiles

### Normal Administration

```text
FW01
DC01
FILE01
LINUX01
WIN11-01
```

### Minimal Administration

```text
FW01
DC01
WIN11-01
```

### Linux / Monitoring

```text
FW01
DC01
LINUX01
WIN11-01
```

### Microsoft Infrastructure

```text
FW01
DC01
FILE01
WIN11-01
```

### Configuration Manager

```text
FW01
DC01
SCCM01
WIN11-01
```

### Security Exercises

Security workloads are enabled as required for the exercise.

---

## 6. Service Dependencies

A simplified dependency chain:

```text
Network
   |
   v
FW01
   |
   +----------------------+
   |                      |
   v                      v
DC01                   Linux services
   |
   +--------+---------+
   |        |         |
   v        v         v
 AD       DNS       DHCP
   |
   +--------------------+
   |         |          |
   v         v          v
 GPO      PKI/NPS    Domain Clients
                         |
                         v
                    FILE01 / WSUS
                         |
                         v
                     SCCM01
                         |
                         v
                       SQL
```

The exact dependencies vary by service.

---

## 7. Identity Architecture

The logical domain is:

```text
acme.local
```

OU structure:

```text
acme.local
├── OU=Users
│   ├── OU=IT
│   ├── OU=HR
│   ├── OU=Finance
│   └── OU=Management
│
├── OU=Groups
│   ├── GG=IT-Admins
│   ├── GG=HR
│   ├── GG=Finance
│   └── GG=Management
│
├── OU=Computers
│   ├── OU=Workstations
│   └── OU=Laptops
│
├── OU=Servers
│   ├── OU=Infrastructure
│   └── OU=Applications
│
├── OU=Admins
```

The project covers:

- LDAP
- Kerberos
- DNS dependency
- SYSVOL
- GPO processing
- Group membership
- Delegation
- Service accounts
- LAPS/local administrator management
- Certificates
- AD replication
- FSMO roles
- Sites and Services

---

## 8. High-Level Service Placement

### Identity

```text
DC01
DC02
```

### Network Services

```text
FW01
DC01
NPS01
```

### Application Services

```text
WEB01
SQL01
```

### Endpoint Management

```text
SCCM01
WSUS01
```

### Security

```text
PKI01
NPS01
WAZUH01
```

### Monitoring

```text
MON01
SYSLOG01
```

### Data Protection

```text
BACKUP01
```

### Linux / Containers

```text
LINUX01
```

---

## 9. Enterprise Separation and Current Consolidation

The logical design uses separate roles:

```text
PKI01
NPS01
WSUS01
MON01
WAZUH01
SYSLOG01
BACKUP01
```

The current implementation consolidates them:

```text
DC01       -> AD + DNS + DHCP + GPO + PKI + NPS
FILE01     -> SMB + WSUS
LINUX01    -> Linux + monitoring + logging + containers
SCCM01     -> MECM + SQL
```

The current implementation is not intended to represent a production one-VM-per-role deployment. It provides the same service roles and administration exercises within the available hardware.

---

## 10. Growth Path

Additional hardware can be used to separate roles without changing the logical architecture.

```text
Current

DC01
 ├── AD
 ├── DNS
 ├── DHCP
 ├── PKI
 └── NPS

Future

DC01
DC02
PKI01
NPS01
```

Similarly:

```text
Current

LINUX01
 ├── Monitoring
 ├── Logging
 ├── Docker
 └── Security workloads

Future

MON01
SYSLOG01
WAZUH01
LINUX01
DOCKER01
```

---

## 11. Availability and Resilience

The current 16 GB environment does not provide production high availability.

The project covers HA and resilience concepts through:

- DC02
- AD replication
- Secondary DNS
- Service redundancy
- Backup and restore
- Failure simulations
- Recovery procedures
- RTO/RPO analysis

A failure exercise demonstrates recovery procedures; it does not make the laptop deployment a highly available production environment.

---

## 12. Documentation Standard

1. Purpose
2. Dependencies
3. Design
4. Implementation
5. Configuration
6. Security considerations
7. Monitoring
8. Backup
9. Testing
10. Failure scenarios
11. Troubleshooting
12. Recovery
13. Automation opportunities

---

## 13. Architecture Diagram

The architecture diagram represents the logical enterprise:

```text
Internet
   |
Firewall / VPN
   |
Network Zones
   |
Infrastructure Services
   |
Application Services
   |
Monitoring / Security
   |
Clients / Administration
   |
Backup / Recovery
```

See [`diagrams/architecture.png`](../../diagrams/architecture.png).