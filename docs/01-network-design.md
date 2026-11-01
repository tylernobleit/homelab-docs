# 01. Network Design

## Goals

- Separate untrusted devices (guests, IoT) from servers and workstations
- Make the management plane reachable from one place only
- Keep addressing predictable so problems can be diagnosed quickly

All addresses below are private RFC 1918 examples. Replace them if they clash with your home network.

## VLAN and IP plan

| VLAN ID | Name | Subnet | Gateway | DHCP range | Purpose |
|---|---|---|---|---|---|
| 10 | Management | 10.10.10.0/24 | 10.10.10.1 | 10.10.10.100-150 | Switch, AP and firewall admin interfaces |
| 20 | Servers | 10.10.20.0/24 | 10.10.20.1 | none (static) | Domain controller, file/backup server |
| 30 | Workstations | 10.10.30.0/24 | 10.10.30.1 | 10.10.30.100-200 | Domain-joined client PCs |
| 40 | Guest/IoT | 10.10.40.0/24 | 10.10.40.1 | 10.10.40.100-200 | Internet only, no access to internal VLANs |

Static address conventions: `.1` gateway, `.2-.9` network gear, `.10-.49` servers, `.50-.99` reserved/static clients.

## Inter-VLAN firewall policy

Default deny between VLANs; allow only what is needed. Read as "from row to column".

| From \ To | Management | Servers | Workstations | Guest/IoT | Internet |
|---|---|---|---|---|---|
| Management | n/a | allow | allow | allow | allow |
| Servers | deny | n/a | deny | deny | allow (updates) |
| Workstations | deny | allow only AD, DNS, SMB, print | n/a | deny | allow |
| Guest/IoT | deny | deny | deny | n/a | allow |

Workstation-to-server ports to open (to the domain controller and file server): DNS 53, Kerberos 88, LDAP 389/636, SMB 445, RPC 135 plus the dynamic range if required, NTP 123.

## DNS plan

- Domain controller is the only DNS server handed to VLAN 20 and 30 clients
- DC forwards external queries to the firewall or a public resolver
- Guest/IoT VLAN receives the firewall's resolver, never the internal DNS server

## Documentation to keep current

- [ ] Network diagram (export from draw.io or Mermaid and commit it)
- [ ] Switch port map (port, VLAN, device)
- [ ] IP address register for statics and DHCP reservations
- [ ] Credentials stored in a password manager, **never in this repo**

## Results

**TODO:** add the final diagram, port map, and a note on anything that differed from this plan.
