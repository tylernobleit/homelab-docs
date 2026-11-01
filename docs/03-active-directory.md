# 03. Active Directory, DNS and DHCP

## 1. Prepare the server

- Windows Server VM on VLAN 20 with a static IP (for example `10.10.20.10`)
- Rename the server (for example `DC01`) and set DNS to itself after promotion; use `127.0.0.1` or its own IP
- Install updates before promoting

## 2. Install AD DS and promote

PowerShell (elevated):

```powershell
Install-WindowsFeature AD-Domain-Services, DNS, DHCP -IncludeManagementTools

Install-ADDSForest `
    -DomainName "lab.example.local" `
    -DomainNetbiosName "LAB" `
    -InstallDns `
    -SafeModeAdministratorPassword (Read-Host -AsSecureString "DSRM password")
```

Use a name you control or a clearly non-routable lab name. Avoid `.local` in production; it conflicts with mDNS. For a lab it is acceptable, but note the reasoning in your write-up.

## 3. OU structure

```
lab.example.local
  Staff
    Accounting
    Sales
    Operations
  Computers
    Workstations
    Servers
  Groups
  Service Accounts
  Disabled Users
```

Create it with PowerShell so it is repeatable:

```powershell
$base = "DC=lab,DC=example,DC=local"
"Staff","Computers","Groups","Service Accounts","Disabled Users" |
    ForEach-Object { New-ADOrganizationalUnit -Name $_ -Path $base }
"Accounting","Sales","Operations" |
    ForEach-Object { New-ADOrganizationalUnit -Name $_ -Path "OU=Staff,$base" }
"Workstations","Servers" |
    ForEach-Object { New-ADOrganizationalUnit -Name $_ -Path "OU=Computers,$base" }
```

## 4. DHCP

1. Authorize the DHCP server in AD: `Add-DhcpServerInDC`
2. Create the VLAN 30 scope (10.10.30.100-200), gateway option 003 = `10.10.30.1`, DNS option 006 = `10.10.20.10`, domain name option 015
3. If the DHCP server is on a different subnet than clients, configure a DHCP relay (helper address) on the firewall for VLAN 30

## 5. Group Policy baseline

| GPO | Linked to | Settings |
|---|---|---|
| Password and lockout | Domain | min length 12, lockout after 5 attempts, 15 minute lockout duration |
| Workstation baseline | Workstations | screen lock after 10 minutes, Windows Update schedule, disable guest account |
| Drive mapping | Staff | map shared folders by department group |
| Removable media | Workstations | restrict USB storage if policy requires |

Test GPOs on a pilot OU or a single test machine before linking widely, and check results with `gpresult /r` on the client.

## 6. Join a client and test

- Join a Windows client in VLAN 30 to the domain
- Sign in as a standard user, confirm GPO and drive maps apply
- Use the onboarding and offboarding scripts from [it-admin-toolkit](https://github.com/tylernobleit) to create and disable a user, and record the result

## Results

**TODO:** screenshots of ADUC/OU tree, the DHCP scope, `gpresult` output, and issues hit along the way (DNS being the usual suspect).
