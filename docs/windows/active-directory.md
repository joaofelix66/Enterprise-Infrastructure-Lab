# Active Directory

## Domain

| Setting | Value |
|---|---|
| Domain | `acme.local` |
| Domain controller | `DC01` |
| Directory service | Active Directory Domain Services |

`DC01` is currently the only domain controller in the lab. Active Directory provides domain authentication and centralized management of users, computers, security groups, and Group Policy.

## OU structure

The domain uses the following OU layout to separate users and computers by role and to provide scopes for Group Policy.

```text
acme.local
├── OU=Users
│   ├── OU=IT
│   ├── OU=HR
│   ├── OU=Finance
│   └── OU=Management
├── OU=Groups
│   ├── GG=IT-Admins
│   ├── GG=HR
│   ├── GG=Finance
│   └── GG=Management
├── OU=Computers
│   ├── OU=Workstations
│   └── OU=Laptops
├── OU=Servers
│   ├── OU=Infrastructure
│   └── OU=Applications
├── OU=Admins
```

## Group Policy

The GPOs are being configured with the CIS Benchmark as a reference, with individual recommendations selected according to the lab's requirements and the role of each system. Current settings cover authentication, account lockout, LDAP and NTLM, SMB, Windows Firewall, Microsoft Defender, auditing, PowerShell logging, user rights, UAC, and Windows LAPS.

The full GPO export, links, scope, and effective settings are documented separately in [`group-policy.md`](group-policy.md).
