# Enterprise Infrastructure Lab - ACME Corporation

This checklist tracks the development of the ACME Corporation enterprise infrastructure lab.

The checklist is divided into Current Lab, Advanced / On-Demand, and Future Expansion work.

The logical enterprise architecture describes the target environment. The current implementation is constrained by the available host hardware, primarily the 16 GB RAM limit of the VMware Workstation host.

---

## Enterprise Architecture Checklist

### Phase 1 - Foundation
- [x] Finalize enterprise architecture
- [x] Finalize VM consolidation strategy
- [x] Finalize IP addressing plan
- [x] Finalize VLAN plan
- [x] Finalize VM resource profiles
- [x] Document hardware limitations
- [x] Create architecture diagram
- [x] Create network diagram
- [x] Establish repository structure
- [x] Establish naming conventions
- [x] Establish documentation standards

---

### Phase 2 - Virtualization (VMware Workstation)
- [x] Configure VMware virtual networks
- [x] Configure NAT networking
- [x] Configure isolated networks
- [x] Configure VLAN-aware networking where practical
- [x] Create base Sysprep templates / golden images
- [x] Create current lab VMs (`FW01`, `DC01`, `FILE01`, `LINUX01`, `WIN11-01`, `SCCM01`)
- [x] Document VM resource allocations
- [x] Document VM startup priorities
- [x] Document which workloads are mutually exclusive
- [x] Test VM snapshots
- [x] Test VM restore from snapshot
- [x] Document resource constraints

*Note: `DC02`, `WEB01`, `SQL01`, `WSUS01`, `PKI01`, `NPS01`, `MON01`, `WAZUH01`, `SYSLOG01`, `BACKUP01`, and `WIN11-02` remain logical roles or consolidated workloads unless hardware/resources allow separate deployment.*

---

### Phase 3 — Network Infrastructure & Routing

#### FW01 (OPNsense)
- [x] Install/configure OPNsense (Permanent Disk Mode)
- [x] Configure WAN interface (em0 - NAT)
- [x] Configure LAN Trunk parent interface (em1 - Trunk-Segment)
- [x] Configure VLAN interfaces:
  - `192.168.10.1/24` (VLAN 10 - MANAGEMENT)
  - `192.168.20.1/24` (VLAN 20 - SERVERS)
  - `192.168.30.1/24` (VLAN 30 - CLIENTS)
  - `192.168.40.1/24` (VLAN 40 - DMZ)
  - `192.168.50.1/24` (VLAN 50 - SECURITY)
  - `192.168.60.1/24` (VLAN 60 - BACKUP)
  - `192.168.70.1/24` (VLAN 70 - VPN)
  - `192.168.80.1/24` (VLAN 80 - GUEST)
- [x] Configure routing & NAT
- [x] Configure firewall aliases (`RFC1918_Subnets`, `DC01_IP`, `AD_Admin_Ports`)
- [x] Configure firewall rules:
  - Allow VLAN 10 -> Full Administrative Access
  - Allow VLAN 30 -> VLAN 20 (WinRM `5985/5986`, RDP `3389`, AD/DNS ports)
  - Allow outbound Internet traffic (`!RFC1918_Subnets`)
  - Implement default-deny rules for unauthorized inter-VLAN traffic
  - Isolate DMZ (VLAN 40) and GUEST (VLAN 80) from internal networks
- [x] Configure OPNsense **DHCP Relay** on VLANs 30, 50, and 80 pointing to `192.168.20.10` (DC01)
- [x] Configure OPNsense NTP server across all active VLAN interfaces
- [x] Configure management access (VLAN 10 restricted)
- [ ] Configure VPN (WireGuard / OpenVPN)
- [x] Export XML firewall configuration backup

#### VLANs
- [x] VLAN 10 - Management
- [x] VLAN 20 - Servers
- [x] VLAN 30 - Clients
- [x] VLAN 40 - DMZ
- [x] VLAN 50 - Security / Monitoring
- [x] VLAN 60 - Backup
- [x] VLAN 70 - VPN
- [x] VLAN 80 - Guest

#### Network Validation
- [x] Test routing across subnets
- [x] Test NAT / WAN access
- [x] Test DNS connectivity
- [x] Test allowed inter-VLAN traffic
- [x] Test blocked inter-VLAN traffic
- [x] Test management access
- [ ] Test VPN
- [x] Test DMZ isolation
- [ ] Test guest isolation

---

### Phase 4 - Active Directory & Core Services

#### DC01 (`192.168.20.10`)
- [x] Install Windows Server Core
- [x] Configure static IP (`192.168.20.10`), Netmask (`255.255.255.0`), Gateway (`192.168.20.1`), DNS (`127.0.0.1`)
- [x] Install AD DS role via PowerShell (`Install-WindowsFeature -Name AD-Domain-Services`)
- [x] Create domain (`acme.local`) via `Install-ADDSForest`
- [ ] Configure Windows NTP client to sync time from OPNsense (`192.168.20.1`)
- [x] Install and authorize DHCP Server role
- [x] Configure DHCP Scopes:
  - Scope 1: `192.168.30.0/24` (Clients — Gateway: `192.168.30.1`, DNS: `192.168.20.10`)
- [x] Create Organizational Units (OUs)
- [x] Create domain users
- [x] Create security groups
- [x] Create administrative accounts
- [ ] Install AD CS (Root CA) for local TLS trust auto-enrollment
- [ ] Configure domain security policies
- [ ] Document Active Directory structure

#### Active Directory Testing
- [x] Point `WIN11-01` DNS to `192.168.20.10` and join `acme.local`
- [x] Install RSAT tools on `WIN11-01` (`RSAT.ActiveDirectory.DS-LDS.Tools`, `RSAT.Dns.Tools`, `RSAT.DHCP.Tools`)
- [x] Test domain authentication & Kerberos ticket issuance
- [x] Test DNS resolution
- [x] Test DHCP leases via OPNsense DHCP Relay
- [ ] Test user/group permissions
- [ ] Test account lockout
- [ ] Test password policies
- [ ] Test administrative access
- [ ] Document common AD troubleshooting procedures

---

### Phase 5 - Group Policy
- [ ] Create baseline GPO
- [ ] Configure password policy
- [ ] Configure account lockout policy
- [ ] Configure Windows Firewall rules via GPO
- [ ] Configure Windows Defender
- [ ] Configure security auditing
- [ ] Configure workstation policies
- [ ] Configure administrative policies
- [ ] Configure Windows update policies
- [ ] Test GPO inheritance
- [ ] Test GPO security filtering
- [ ] Test GPO processing (`gpupdate /force`, `gpresult /h`)
- [ ] Create GPO troubleshooting procedure

---

### Phase 6 - File Services (FILE01 / DC01 Consolidated)
- [ ] Configure storage
- [ ] Configure SMB
- [ ] Create departmental shares
- [ ] Configure NTFS permissions
- [ ] Configure share permissions
- [ ] Configure Access-Based Enumeration (ABE)
- [ ] Configure quotas
- [ ] Configure shadow copies
- [ ] Configure auditing
- [ ] Test access using different users/groups
- [ ] Test denied access
- [ ] Test file recovery
- [ ] Document permissions model

---

### Phase 7 - Linux Infrastructure (LINUX01)
- [ ] Install Ubuntu Server (Headless) on VLAN 20 (`192.168.20.20`)
- [ ] Configure hostname
- [ ] Configure static IP, gateway (`192.168.20.1`), and DNS (`192.168.20.10`)
- [ ] Configure SSH key authentication
- [ ] Configure SSH keys
- [ ] Configure users/groups
- [ ] Configure sudo
- [ ] Configure firewall (`ufw`)
- [ ] Apply basic hardening
- [ ] Configure system updates
- [ ] Configure systemd
- [ ] Configure logging
- [ ] Document Linux administration

---

### Phase 8 - Web and Containers

#### Nginx
- [ ] Install Nginx
- [ ] Configure web service
- [ ] Configure virtual hosts
- [ ] Configure TLS where appropriate
- [ ] Configure logging
- [ ] Test HTTP/HTTPS
- [ ] Simulate Nginx failure
- [ ] Restore service

#### Docker
- [ ] Install Docker
- [ ] Configure Docker networking
- [ ] Configure Docker container memory limits (e.g., `mem_limit: 512m`)
- [ ] Create Docker Compose configuration
- [ ] Deploy test containers
- [ ] Configure persistent storage
- [ ] Configure container logging
- [ ] Configure restart policies
- [ ] Simulate container failure
- [ ] Restore container service

---

### Phase 9 - Monitoring and Logging

#### Monitoring
- [ ] Deploy Prometheus
- [ ] Configure `Node_Exporter` on `LINUX01`
- [ ] Configure `Windows_Exporter` on `DC01` / `WIN11-01`
- [ ] Deploy Grafana
- [ ] Configure Prometheus datasource
- [ ] Create infrastructure dashboard
- [ ] Create Windows dashboard
- [ ] Create Linux dashboard
- [ ] Create service-health dashboard
- [ ] Configure useful alerts
- [ ] Test alert generation

#### Logging
- [ ] Configure centralized logging
- [ ] Collect Linux logs
- [ ] Collect Windows events
- [ ] Collect firewall logs
- [ ] Collect application logs
- [ ] Test log ingestion
- [ ] Test log searching
- [ ] Document logging architecture

---

### Phase 10 - Configuration Management (SCCM01)
- [ ] Configure `SCCM01` VM profile (On-Demand execution strategy)
- [ ] Install SQL Server instance / SQL Server Express
- [ ] Extend Active Directory Schema for Configuration Manager
- [ ] Create AD System Management Container and assign permissions
- [ ] Install SCCM / MECM Primary Site
- [ ] Deploy SCCM client agents to `WIN11-01`
- [ ] Test software distribution, boundary groups, and hardware inventory

---

### Phase 11 - Security
- [ ] Apply least privilege
- [ ] Separate administrative accounts (Tiered Administration model)
- [ ] Configure Windows Firewall
- [ ] Configure Linux firewall
- [ ] Review exposed services
- [ ] Review firewall rules
- [ ] Apply SSH hardening
- [ ] Configure centralized logging
- [ ] Monitor authentication events
- [ ] Review privileged accounts
- [ ] Perform basic security review
- [ ] Document security controls

---

### Phase 12 - Backup and Recovery
- [ ] Define backup policy
- [ ] Define retention
- [ ] Define RPO
- [ ] Define RTO
- [ ] Identify critical workloads
- [ ] Back up Active Directory System State (`wbadmin`)
- [ ] Back up FILE01
- [ ] Back up LINUX01 configuration/data
- [ ] Back up important scripts
- [ ] Back up configuration files (OPNsense XML)
- [ ] Test backup jobs
- [ ] Test file restoration
- [ ] Test configuration restoration
- [ ] Document recovery procedures

---

### Phase 13 - Automation

#### PowerShell
- [ ] Create AD administration scripts
- [ ] Create user-management scripts
- [ ] Create file-service scripts
- [ ] Create service-health scripts
- [ ] Create reporting scripts

#### Bash
- [ ] Create Linux administration scripts
- [ ] Create service-management scripts
- [ ] Create backup scripts
- [ ] Create health-check scripts

#### Python
- [ ] Create infrastructure utilities
- [ ] Create log-processing tools
- [ ] Create reporting tools

#### Ansible
- [ ] Configure Ansible
- [ ] Create inventory
- [ ] Create Linux playbooks
- [ ] Test idempotency
- [ ] Document automation workflow

---

### Phase 14 - Troubleshooting Scenarios

#### Windows
- [ ] DNS failure
- [ ] DHCP failure / DHCP Relay issue
- [ ] AD authentication failure
- [ ] GPO processing failure
- [ ] Account lockout
- [ ] File permission problem
- [ ] SMB connectivity failure
- [ ] Disk exhaustion

#### Linux
- [ ] SSH failure
- [ ] DNS failure
- [ ] Disk exhaustion
- [ ] Nginx failure
- [ ] Docker failure
- [ ] Container failure
- [ ] Permission problem

#### Network
- [ ] Incorrect firewall rule
- [ ] Routing failure
- [ ] NAT failure
- [ ] VLAN connectivity failure
- [ ] VPN failure

---

### Phase 15 - Operational Maturity
- [ ] Create change-management process
- [ ] Create asset inventory
- [ ] Create service inventory
- [ ] Create IP inventory
- [ ] Create certificate inventory
- [ ] Create startup procedure
- [ ] Create shutdown procedure
- [ ] Create maintenance procedures
- [ ] Create troubleshooting runbooks
- [ ] Document dependencies
- [ ] Document service owners/administrators
- [ ] Document recovery priorities

---

### Phase 16 - Documentation
- [ ] Update architecture documentation
- [ ] Update network documentation
- [ ] Update Active Directory documentation
- [ ] Update Group Policy documentation
- [ ] Update file-services documentation
- [ ] Update Linux documentation
- [ ] Update Docker documentation
- [ ] Update security documentation
- [ ] Update monitoring documentation
- [ ] Update backup documentation
- [ ] Update automation documentation
- [ ] Add troubleshooting documentation
- [ ] Add incident reports
- [ ] Update architecture diagram
- [ ] Update network diagram

---

## Future Expansion

Target architecture tasks deferred until hardware resources expand beyond the current 16 GB host:

### Additional Infrastructure
- [ ] Deploy `DC02` as a dedicated secondary domain controller
- [ ] Test AD replication between `DC01` and `DC02`
- [ ] Test domain-controller failure and recovery
- [ ] Deploy dedicated `WEB01`
- [ ] Deploy dedicated `SQL01`
- [ ] Deploy dedicated `WSUS01`
- [ ] Deploy dedicated `PKI01`
- [ ] Deploy dedicated `NPS01`
- [ ] Deploy dedicated `MON01`
- [ ] Deploy dedicated `WAZUH01`
- [ ] Deploy dedicated `SYSLOG01`
- [ ] Deploy dedicated `BACKUP01`
- [ ] Deploy `WIN11-02`

### Enterprise Resilience
- [ ] Implement redundant domain controllers
- [ ] Implement dedicated infrastructure services
- [ ] Test service redundancy
- [ ] Test failover
- [ ] Test recovery of individual infrastructure roles
- [ ] Test multi-service failure scenarios

### Enterprise Management
- [ ] Expand Configuration Manager deployment
- [ ] Expand WSUS infrastructure
- [ ] Implement dedicated SQL workload
- [ ] Expand endpoint management
- [ ] Implement more advanced monitoring
- [ ] Implement dedicated security monitoring infrastructure

### Future Hardware / Infrastructure
- [ ] Upgrade host RAM
- [ ] Add additional storage
- [ ] Consider dedicated lab server
- [ ] Consider Proxmox/ESXi environment
- [ ] Consider multiple physical hosts
- [ ] Evaluate nested virtualization capacity
- [ ] Evaluate dedicated network hardware
- [ ] Evaluate separate backup storage

### Cloud / Hybrid Expansion
- [ ] Define hybrid-cloud architecture
- [ ] Deploy cloud VM
- [ ] Configure site-to-site VPN
- [ ] Integrate cloud networking
- [ ] Integrate identity
- [ ] Configure cloud monitoring
- [ ] Configure cloud backup
- [ ] Test hybrid connectivity
- [ ] Document hybrid architecture

---

## Completion Criteria

The project is not considered complete simply because every future checkbox is checked. The current lab must demonstrate that deployed infrastructure can be:

$$\text{Designed} \longrightarrow \text{Implemented} \longrightarrow \text{Tested} \longrightarrow \text{Secured} \longrightarrow \text{Documented} \longrightarrow \text{Monitored} \longrightarrow \text{Troubleshot} \longrightarrow \text{Recovered} \longrightarrow \text{Improved}$$

### Current Lab Definition of Done
- [ ] Infrastructure works within available hardware
- [ ] Network is segmented
- [ ] Active Directory is operational
- [ ] DNS/DHCP are operational
- [ ] Group Policy is operational
- [ ] File services are operational
- [ ] Linux services are operational
- [ ] Monitoring is operational
- [ ] Logging is operational
- [ ] Security controls are implemented
- [ ] Backups are tested
- [ ] Restore procedures are tested
- [ ] Troubleshooting scenarios have been completed
- [ ] Automation has been implemented where useful
- [ ] Documentation reflects the actual environment
- [ ] Repository contains no credentials or secrets
- [ ] Changes are committed to version control