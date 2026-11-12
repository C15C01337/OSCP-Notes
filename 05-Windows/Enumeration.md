# Windows local enumeration

[Home](../README.md) · [Services and tasks](Services-Tasks.md) · [Privileges and credentials](Privileges-Credentials.md) · [Tunnels](../07-Pivoting/Tunnels.md)

## Identity, token and system

**Target / CMD or PowerShell:**

```batch
whoami
whoami /all
hostname
net user
net localgroup
systeminfo
```

Record local versus domain identity, group membership, token integrity, privileges, OS build and architecture. A user in Administrators may hold a filtered token. Service, network and interactive logons can have different capabilities.

**PowerShell alternatives:**

```powershell
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember -Group Administrators
```

These cmdlets may be unavailable in some environments; use native `net` commands where necessary. Avoid assuming English group names on a localized system.

## Network and processes

```batch
ipconfig /all
route print
netstat -ano
tasklist /svc
netsh advfirewall show currentprofile
netsh advfirewall firewall show rule name=all
```

Match a listener PID to its process and service. Local-only services can be reached through [forwarding](../07-Pivoting/Tunnels.md). Reading firewall configuration does not imply permission to change it.

## Services and installed applications

**PowerShell:**

```powershell
Get-CimInstance Win32_Service | Select-Object Name,State,StartMode,StartName,PathName
Get-Process | Select-Object Id,ProcessName,Path
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*' |
    Select-Object DisplayName,DisplayVersion,InstallLocation
Get-ItemProperty 'HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*' |
    Select-Object DisplayName,DisplayVersion,InstallLocation
Get-HotFix
```

Some fields or queries require extra access. A service-query failure through WinRM does not mean RDP will always resolve it. Read accessible registry service configuration as an alternative:

```powershell
Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Services\*' |
    Select-Object PSChildName,ImagePath,ObjectName,Start
```

Prefer uninstall-registry queries to `Win32_Product`, which can trigger installer consistency checks. A hotfix list is useful but incomplete evidence for exploit applicability; record exact component versions too.

## Files, ACLs and history

```powershell
Get-ChildItem C:\Users -Force -ErrorAction SilentlyContinue
Get-ChildItem C:\Users -Recurse -File -Include *.kdbx,*.config,*.xml,*.ini -ErrorAction SilentlyContinue
Get-Acl 'C:\Path\To\File' | Format-List
Get-History
(Get-PSReadLineOption).HistorySavePath
```

`Get-History` lists the current session's history; PSReadLine's history file is separate. Read that path if accessible. PowerShell operational events (for example ID 4104 when script-block logging is enabled) can contain useful script content; access and logging policy vary.

**CMD:**

```batch
dir /a C:\Users
icacls "C:\Path\To\File"
findstr /si password C:\Path\To\App\*.xml C:\Path\To\App\*.ini
```

Focus searches on the current user's files and identified applications before scanning entire disks. Check both a file and its containing directory. Effective permissions include inheritance, group membership and explicit denies; a matching text string in an ACL is only a lead.

## Scheduled tasks, volumes and drivers

```batch
schtasks /query /fo LIST /v
mountvol
driverquery /v /fo csv
```

**PowerShell:**

```powershell
Get-ScheduledTask | Select-Object TaskName,TaskPath,State,Actions,Principal
Get-CimInstance Win32_PnPSignedDriver | Select-Object DeviceName,DriverVersion,Manufacturer
```

A task's principal determines its execution identity; its author field does not. An unmounted volume may hold backups, but access/mounting permissions still apply. Driver names and versions are needed to assess a driver-specific exploit.

## Enumeration tools

WinPEAS, Seatbelt, PowerUp, SharpUp, PrivescCheck, JAWS and windows-privesc-check summarize different checks. Use a matching architecture/runtime, choose categories using local help and save output. PowerUp is a PowerShell script exposing functions after dot-sourcing; PrivescCheck also has script-based distributions. Do not assume every name has a standalone EXE.

WES-NG, Watson, Sherlock and related suggesters compare version/patch evidence to known issues. Confirm each proposed exploit's actual requirements. A source typo references CVE-2018-8210 while linking CVE-2018-8120; use the code's actual target description rather than the copied label.
