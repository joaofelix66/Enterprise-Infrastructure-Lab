# 001 - VLAN Tagging and Gateway Connectivity

**Status:** Resolved. Gateway connectivity was restored after setting the VLAN IDs on the affected adapters.

## Problem

`DC01` could not communicate with its default gateway. `WIN11-01` had the same issue.

## Resolution

The VLAN ID was set on the network-adapter registry entries, then the adapters were restarted. VLAN 20 was set on `DC01`; VLAN 30 was set on `WIN11-01`.

On `DC01`:

```powershell
Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\00*" | Where-Object { $_.DriverDesc } | ForEach-Object {
    Set-ItemProperty -Path $_.PSPath -Name "VlanID" -Value "20"
}
Restart-NetAdapter -Name *
```

On `WIN11-01`:

```powershell
Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\00*" | Where-Object { $_.DriverDesc } | ForEach-Object {
    Set-ItemProperty -Path $_.PSPath -Name "VlanID" -Value "30"
}
Restart-NetAdapter -Name *
```

Gateway connectivity returned after the change. The exact underlying cause was not found.. so i'm not sure what caused the issue, if it was a **Firewall Rule** mistake on my part or not.