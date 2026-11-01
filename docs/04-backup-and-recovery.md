# 04. Backup and Recovery

A backup that has never been restored is a guess. This phase builds a backup plan and proves it works.

## Strategy: 3-2-1

- **3** copies of the data (production plus two backups)
- **2** different media or systems (for example the file server and an external disk or NAS)
- **1** copy offsite or offline (cloud bucket, rotated external drive stored elsewhere)

## What gets backed up

| Asset | Method | Frequency | Retention |
|---|---|---|---|
| Domain controller (system state) | Windows Server Backup system state | daily | 14 days |
| File server shares | file-level backup to backup target | nightly | 30 daily, 12 monthly |
| Firewall configuration | exported XML to the file server | after each change and weekly | 12 copies |
| VM images | hypervisor snapshot export | weekly | 4 copies |

## Windows Server Backup example

```powershell
Install-WindowsFeature Windows-Server-Backup

$policy = New-WBPolicy
Add-WBSystemState -Policy $policy
$target = New-WBBackupTarget -NetworkPath "\\backup01\dc-backups"
Add-WBBackupTarget -Policy $policy -Target $target
Set-WBSchedule -Policy $policy -Schedule 02:00
Set-WBPolicy -Policy $policy
```

## Monitoring that it is actually running

Use `Test-BackupFreshness.ps1` or `verify-backups.sh` from [it-admin-toolkit](https://github.com/tylernobleit) on a schedule, and alert on a nonzero exit code.

## Restore tests (do these and write them up)

| Test | Procedure | Pass criteria |
|---|---|---|
| Single file | Delete a file from a share, restore it from last night's backup | file returns intact, permissions preserved |
| Full share | Restore a share to an alternate location | file counts and checksums match |
| Domain controller | Restore system state to a clean VM (authoritative restore of a deleted OU is a good exercise) | domain services start, deleted object reappears |
| Firewall | Rebuild from a fresh install plus the saved config | rules and VLANs identical |

Record for each test: date, how long it took, what went wrong, what the documented procedure was missing.

## Recovery objectives

Set realistic targets for the lab and write them down:

- **RPO** (maximum acceptable data loss): 24 hours for file data
- **RTO** (maximum acceptable downtime): 4 hours for the file server, 8 hours for the domain

Then measure actual restore times against them.

## Results

**TODO:** table of completed restore tests with real timings and the fixes you made to the procedure.
