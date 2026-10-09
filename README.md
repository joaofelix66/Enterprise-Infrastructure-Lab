# Enterprise Infrastructure Lab

A personal enterprise infrastructure project based on a fictional company, ACME Corporation, with approximately 100 employees.

The project covers the design, implementation, administration, security, monitoring, automation, troubleshooting, backup, and recovery of an enterprise environment using Windows, Linux, networking, security, and cloud technologies.

The environment is primarily implemented with VMware Workstation on a 16 GB laptop.

> **Logical enterprise architecture ≠ current VM implementation.**

The target architecture contains more infrastructure roles than the current host can run as separate VMs at the same time. Services are consolidated where practical, and heavier workloads are started only when required.

The objective is to build and operate an enterprise environment rather than simply deploy a collection of unrelated VMs.

---

## Project Goals

- Systems administration
- Windows Server administration
- Linux administration
- Active Directory
- DNS and DHCP
- Group Policy
- File services and permissions
- Networking and segmentation
- Firewalls, NAT, routing, and VPN
- PKI and certificate management
- RADIUS / NPS
- WSUS and Microsoft Configuration Manager
- SQL Server
- Docker and containerized services
- Monitoring and centralized logging
- Security monitoring and hardening
- Backup and disaster recovery
- PowerShell, Bash, and Python
- Ansible and Infrastructure as Code
- Virtualization
- Troubleshooting and incident response
- Cloud and hybrid infrastructure

---

## Enterprise Context

The environment models an enterprise rather than a collection of isolated test systems.

| System | Logical Role |
|---|---|
| FW01 | Firewall, routing, NAT, VPN, traffic control |
| DC01 | Active Directory, DNS, DHCP, GPO |
| DC02 | Secondary domain controller, DNS, replication, resilience |
| FILE01 | SMB, NTFS permissions, ABE, quotas, shadow copies |
| WEB01 | IIS/Nginx, web services, DMZ workload |
| SQL01 | SQL Server database platform |
| SCCM01 | Microsoft Configuration Manager / MECM |
| WSUS01 | Windows patch management |
| PKI01 | Active Directory Certificate Services / internal PKI |
| NPS01 | NPS / RADIUS authentication |
| MON01 | Prometheus, Grafana, infrastructure monitoring |
| WAZUH01 | Security monitoring, FIM, event analysis |
| SYSLOG01 | Centralized logging |
| BACKUP01 | Backup and restore operations |
| LINUX01 | Linux administration, SSH, Docker, Nginx, container workloads |
| WIN11-01 | Corporate endpoint and administrative workstation |
| WIN11-02 | Additional corporate endpoint / testing client |

These are logical roles. A role does not necessarily have a dedicated VM in the current implementation.

See [`docs/architecture/architecture.md`](docs/architecture/architecture.md).

---

## Current Hardware

The current host has **16 GB of RAM**.

The complete logical environment cannot run as one VM-per-role deployment on this hardware. The current implementation therefore uses:

- Role consolidation
- Resource-aware VM profiles
- On-demand workloads
- Heavy services powered off when not required
- Separate scenarios for resource-intensive platforms

The hardware constraint affects the VM layout, not the logical enterprise architecture.

---

## Current VM Footprint

| VM | RAM | Current Purpose |
|---|---:|---|
| FW01 | 1.5 GB | OPNsense, firewall, routing, VLANs, NAT, VPN |
| DC01 | 2 GB | AD DS, DNS, DHCP, GPO, AD CS/PKI, NPS/RADIUS |
| FILE01 | 2 GB | SMB, NTFS, ABE, quotas, shadow copies, WSUS |
| LINUX01 | 2.5 GB | Linux, Docker, Nginx, Prometheus, Grafana, Syslog, security workloads |
| WIN11-01 | 4 GB | Corporate endpoint, administration, RSAT, WAC |
| SCCM01 | 6 GB | Microsoft Configuration Manager / MECM and SQL |

These allocations are not intended to run simultaneously.

The current resource profiles and consolidation decisions are documented in [`docs/architecture/architecture.md`](docs/architecture/architecture.md).

---

## Architecture

- [`diagrams/architecture.png`](diagrams/architecture.png)
- [`docs/architecture/architecture.md`](docs/architecture/architecture.md)

The architecture diagram represents the logical enterprise, including roles that are not currently deployed as individual VMs.

---

## Network

| VLAN | Zone | Subnet |
|---:|---|---|
| 10 | Management | `192.168.10.0/24` |
| 20 | Servers | `192.168.20.0/24` |
| 30 | Clients | `192.168.30.0/24` |
| 40 | DMZ | `192.168.40.0/24` |
| 50 | Security / Monitoring | `192.168.50.0/24` |
| 60 | Backup | `192.168.60.0/24` |
| 70 | VPN | `192.168.70.0/24` |
| 80 | Guest | `192.168.80.0/24` |

FW01 provides routing, NAT, firewall policy, and VPN services. Inter-zone traffic follows a least-privilege and default-deny approach where practical.

- [`diagrams/network.png`](diagrams/network.png)
- [`docs/networking/network.md`](docs/networking/network.md)

---

## Technologies

### Virtualization

- VMware Workstation
- Nested virtualization
- Hyper-V laboratory module

### Windows

- Windows Server
- Active Directory
- DNS
- DHCP
- Group Policy
- SMB
- AD CS
- NPS
- WSUS
- Microsoft Configuration Manager
- SQL Server
- PowerShell

### Linux and Containers

- Ubuntu Server / Debian
- SSH
- systemd
- Nginx
- Docker
- Docker Compose
- Bash

### Networking and Security

- IPv4 and subnetting
- VLANs
- Routing
- NAT
- Firewalling
- VPN
- Windows Firewall
- Linux hardening
- Certificate management
- Authentication monitoring

### Monitoring and Operations

- Prometheus
- Grafana
- Node Exporter
- Windows Exporter
- Centralized logging
- Wazuh
- Backup and restore testing

### Automation

- PowerShell
- Bash
- Python
- Ansible
- Terraform

---

## Repository Structure

```text
docs/
├── architecture/
│   └── architecture.md
├── networking/
│   └── network.md
├── windows/
│   ├── active-directory.md
│   ├── group-policy.md
│   ├── file-services.md
│   ├── wsus.md
│   ├── sccm.md
│   ├── pki.md
│   └── nps.md
├── linux/
│   ├── linux.md
│   ├── docker.md
│   └── nginx.md
├── checklist/
│   ├── checklist.md
├── security/
│   ├── security.md
│   ├── hardening.md
│   └── wazuh.md
├── monitoring/
│   ├── monitoring.md
│   └── logging.md
├── backup/
│   ├── backup.md
│   └── disaster-recovery.md
├── automation/
│   └── automation.md
├── operations/
│   ├── change-management.md
│   ├── asset-management.md
│   └── runbooks.md
└── incidents/
    └── README.md

diagrams/
├── architecture.png
└── network.png

scripts/
├── powershell/
├── bash/
└── python/

ansible/
terraform/
docker/
configs/
templates/
```

---

## Operating Model

Major infrastructure components are handled through the same operational cycle:

```text
DESIGN
   ↓
IMPLEMENT
   ↓
TEST
   ↓
DOCUMENT
   ↓
AUTOMATE
   ↓
MONITOR
   ↓
TROUBLESHOOT
   ↓
IMPROVE
```

The project includes normal operations and failure scenarios such as:

- DNS failure
- DHCP failure
- Active Directory authentication failure
- GPO processing failure
- File permission problems
- Disk exhaustion
- Nginx failure
- Docker/container failure
- VPN failure
- Firewall rule mistakes
- Certificate expiration
- WSUS failure
- Configuration Manager deployment failure
- SQL connectivity failure
- Backup failure
- Suspicious authentication
- SSH access problems
- Unreachable services

Incidents use the following structure:

```text
Problem
   ↓
Symptoms
   ↓
Impact
   ↓
Investigation
   ↓
Evidence
   ↓
Root Cause
   ↓
Resolution
   ↓
Prevention
```

---

## Security

The environment uses:

- Least privilege
- Network segmentation
- Default deny where practical
- Separate administrative accounts
- Secure remote administration
- Strong authentication
- Centralized logging
- Security monitoring
- Patch management
- Backup and recovery testing
- Controlled administrative access

Security documentation is kept under `docs/security/`.

---

## Backup and Disaster Recovery

The project includes:

- Backup policy design
- Retention
- Restore procedures
- Restore testing
- RTO and RPO
- Service recovery
- Disaster scenarios

A completed backup job is not considered sufficient by itself. Recovery is tested through restore procedures.

---

## Status

🚧 **Active Development**

Major implementations follow:

```text
Implemented
    ↓
Tested
    ↓
Documented
    ↓
Committed
```

---

## Project Checklist

The project implementation and progress are tracked in the [project checklist](docs/checklist/checklist.md).

---

## Career Objective

The project is being developed as a practical portfolio for:

- Systems Administration
- Infrastructure Engineering
- Windows / Linux Administration
- Networking
- Cloud Infrastructure
- DevOps / Platform Engineering

The repository focuses on practical administration, troubleshooting, security, automation, and recovery rather than a list of installed technologies.

---

## Disclaimer

This is a fictional enterprise environment created for educational and portfolio purposes.

All users, domains, IP addresses, credentials, and organizational data are fictional.