# 002 - Public Resolver Connectivity

**Status:** Resolved. DNS resolution is working in the current lab configuration.

## Problem

Initially, `DC01` and other lab machines could not reach public resolver addresses such as `8.8.8.8`, `1.1.1.1`, `google.com`; `FW01` could. A firewall rule allowing DNS traffic on port 53 was added during troubleshooting.

## Resolution

After the firewall rule change, DNS resolution was working. Public DNS resolvers are configured as forwarders on `DC01`, and domain clients use the internal DNS server for `acme.local` and Active Directory records.