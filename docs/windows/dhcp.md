# DHCP

DHCP runs on `DC01` (`192.168.20.10`). It currently has two active IPv4 scopes, for the client and guest networks.

## Scopes

Output from `Get-DhcpServerv4Scope -ComputerName DC01`:

| Scope ID | Name | Subnet mask | Address range | Lease duration | State |
|---|---|---|---|---|---|
| `192.168.30.0` | Clients Scope | `255.255.255.0` | `192.168.30.100`–`192.168.30.200` | 8 days | Active |
| `192.168.80.0` | Guest scope | `255.255.255.0` | `192.168.80.100`–`192.168.80.200` | 3 days | Active |

## Current setup

DHCP is hosted on `DC01`; the two scopes correspond to the client (`192.168.30.0/24`) and guest (`192.168.80.0/24`) networks.