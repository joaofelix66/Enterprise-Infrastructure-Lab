# Network

## Network Address

192.168.10.0/24

## Hosts

| Host | IP | Role |
|---|---|---|
| DC01 | 192.168.10.10 | DNS/DHCP |
| LINUX01 | 192.168.10.20 | Linux |
| MON01 | 192.168.10.30 | Monitoring |
| WIN11-01 | DHCP | Client |

## DNS

Primary DNS:
192.168.10.10

Domain:
lab.local

## DHCP

DHCP is provided by DC01.

Address pool:
192.168.10.100 – 192.168.10.200

## VMware Networking

The lab uses an isolated VMware virtual network.