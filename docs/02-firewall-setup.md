# 02. Firewall and VLAN Setup

Steps written for pfSense or OPNsense; menu names are nearly identical. Do this on a lab network, not the one your household relies on, until you trust the config.

## 1. Install and first boot

1. Install on a dedicated machine or VM with two NICs (WAN and LAN). In a VM, give the WAN a bridged adapter and the LAN an internal network.
2. Assign interfaces, set the LAN address (for example `10.10.10.1/24`), and change the default admin password immediately.
3. Run the setup wizard: hostname, DNS, time zone, NTP.
4. Take a configuration backup (Diagnostics, Backup & Restore) and store it outside the firewall.

## 2. Create VLANs

1. Interfaces, Assignments, VLANs: add VLAN 20, 30 and 40 on the LAN parent interface (VLAN 10 can be the untagged management network).
2. Assign each VLAN as a new interface, enable it, and set the static IPv4 gateway address from the plan.
3. On the managed switch, configure the firewall-facing port as an 802.1Q trunk carrying VLANs 10, 20, 30, 40, and set access ports to the correct VLAN.

## 3. DHCP

Services, DHCP Server: enable per VLAN using the ranges in the design doc. For VLAN 30, set DNS to the domain controller once it exists. Leave VLAN 20 static.

## 4. Firewall rules

Rules are evaluated top-down per interface. Build them in this order for each interface:

1. Allow specific required traffic (for example workstations to the DC on the AD ports)
2. Block access to the firewall's own admin interface from every VLAN except Management
3. Block everything else destined for private ranges (use an alias of RFC 1918 networks)
4. Allow the remainder to the internet

Guest/IoT example:

| Action | Source | Destination | Note |
|---|---|---|---|
| Block | Guest net | Firewall admin ports | stop admin access |
| Block | Guest net | RFC 1918 alias | isolate from all internal networks |
| Allow | Guest net | any | internet access |

## 5. Verify

- From a guest device: internet works, `ping 10.10.20.10` fails
- From a workstation: DNS and AD ports to the DC succeed, SSH or RDP to servers fails
- Check Firewall, Logs to confirm the blocks are being hit for the right reasons

## 6. Harden

- Disable WAN-side admin access
- Restrict admin UI to HTTPS from Management VLAN only
- Enable automatic or scheduled configuration backups
- Keep the firmware updated and note the version in the build log

## Results

**TODO:** paste sanitised rule screenshots (no public IPs or credentials), and record what tests passed.
