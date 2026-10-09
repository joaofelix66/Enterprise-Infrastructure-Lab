# DNS

DNS runs on `DC01` (`192.168.20.10`) and hosts the Active Directory DNS zones for `acme.local`.

## Zones

Output from `Get-DnsServerZone -ComputerName DC01`:

| Zone | Type | AD-integrated | Reverse lookup zone |
|---|---|---|---|
| `acme.local` | Primary | Yes | No |
| `_msdcs.acme.local` | Primary | Yes | No |
| `0.in-addr.arpa` | Primary | No | Yes |
| `127.in-addr.arpa` | Primary | No | Yes |
| `255.in-addr.arpa` | Primary | No | Yes |

The reverse zones shown above are automatically created zones. Reverse lookup zones for the lab subnets have not been added.

## Forwarders

Output from `Get-DnsServerForwarder -ComputerName DC01`:

- `1.1.1.1`
- `8.8.8.8`

Root hints are enabled. Public resolvers are currently configured as forwarders on `DC01`. DNS resolution is working in the current lab configuration.

Domain-joined clients use the internal DNS server for domain and Active Directory records. `DC01` forwards external queries to the configured forwarders when needed.

## Current scope

The lab currently has one DNS server, so DNS redundancy is not configured. The DNS setup is intentionally basic at this stage; reverse lookup zones for the lab networks and more advanced DNS configuration may be added as the network design develops.

## Configuration checks

```powershell
Get-DnsServerZone -ComputerName DC01
Get-DnsServerForwarder -ComputerName DC01
Resolve-DnsName acme.local -Server 192.168.20.10
Resolve-DnsName example.com -Server 192.168.20.10
```