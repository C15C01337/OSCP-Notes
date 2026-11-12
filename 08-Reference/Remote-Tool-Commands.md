# Remote administration tool reference

[Home](../README.md) · [SMB/RPC](../01-Enumeration/SMB-RPC.md) · [AD workflow](../06-Active-Directory/Workflow.md) · [Lateral movement](../06-Active-Directory/Lateral-Movement.md)

## Authentication arguments

| Family | Password | NT hash | Kerberos |
|---|---|---|---|
| NetExec | `-u <USER> -p '<PASSWORD>'` | `-u <USER> -H <NT_HASH>` | Installed `-k`/cache options; correct realm, DNS and SPN. |
| Impacket | `'<DOMAIN>/<USER>@<HOST>'` then password prompt | `-hashes :<NT_HASH> '<DOMAIN>/<USER>@<HOST>'` | Usually `-k -no-pass` with a usable cache and service hostname. |
| Evil-WinRM | `-u <USER> -p '<PASSWORD>'` | `-u <USER> -H <NT_HASH>` | Tool-specific realm/cache configuration. |

Use NetExec `-d <DOMAIN>` or `--local-auth` according to the actual account scope. Never place a hash in a password field or use a Net-NTLM response as an NT hash. Password punctuation also needs shell-safe quoting.

## NetExec operation map

**Kali / Bash:** inspect one protocol and one module before running it:

```bash
nxc smb --help
nxc smb -L
nxc smb -M spider_plus --options
nxc smb <HOST> -d <DOMAIN> -u <USER> -p '<PASSWORD>' --shares
```

`--no-bruteforce` pairs corresponding lines of two credential lists; without it, list inputs can form combinations. `--continue-on-success` changes stopping behavior and provides no lockout protection. Save each verified pair and host separately.

| Protocol / feature | Source operations | Prerequisites / interpretation |
|---|---|---|
| SMB inventory | `--shares`, `--users`, `--groups`, `--pass-pol`, `--disks`, `--sessions`/`--smb-sessions`, `--loggedon-users`, RID enumeration | Flag names vary by release; authenticated access and query rights vary by operation. |
| SMB files | Share directory listing, `--get-file`, `--put-file`, `spider_plus` | Check share and NTFS rights. Spider metadata and file downloading are different modes; verify size/read-only options locally. |
| SMB credential sources | `--sam`, `--lsa`, `--dpapi`, `--ntds` | Local administrative, backup or replication rights as appropriate; some methods create services, files or snapshots. |
| SMB modules | `gpp_password`, `gpp_autologin`, `powershell_history`, `eventlogcreds`, `lsassy`, `nanodump`, `ntdsutil`, `backup_operator`, `putty`, `vnc`, `mremoteng`, `notepad`, `notepad++`, `bitlocker` | Each reads a distinct store or invokes a distinct mechanism; no module grants its required privilege. |
| Service/config modules | `enum_av`, `spooler`, `webdav` | Presence/state is a lead; avoid equating service detection with exploitability. |
| Vulnerability/coercion modules | MS17-010, SMBGhost, Zerologon, BadSuccessor, PrintNightmare, noPac, `coerce_plus` | Module behavior may modify state. A name containing “check” does not prove it is read-only. |
| LDAP | Users/groups, `adminCount`, descriptions, DCs, delegation, trusts, custom filters, AD CS, MachineAccountQuota, pre-2000 accounts, SCCM, BloodHound, `daclread`, `whoami`, checker modules | Correct base/domain and effective directory read rights; inspect queried attributes, not only a summary label. |
| gMSA | Managed-password retrieval and parsing | Actual permission to read `msDS-ManagedPassword`; names/identifiers are not the secret blob. |
| WinRM | CMD `-x` / PowerShell `-X` where supported | Remoting authorization; inspect installed protocol options. |
| MSSQL | SQL query, command execution, `mssql_priv` | SQL identity, impersonation and feature rights; record and reverse role/configuration changes. |
| SSH / FTP / NFS | Authentication, file get/put, export inspection | Login or export access does not imply a host-root filesystem escape. |
| RDP / WMI | Sessions/screenshots, WMI queries/execution | Protocol authorization, desktop/session or namespace rights. |

`nxcdb` opens the local result database; `help` and `proto smb` select its console context. Stored successes may be stale.

## Impacket operation map

Kali packages commonly provide `impacket-<example>`; upstream checkouts use `<example>.py`. Check the installed parser rather than patching imports or `getMachineName` to suppress an unexplained error.

| Examples | Purpose / boundary | Chapter |
|---|---|---|
| rpcdump, samrdump, lookupsid, netview | RPC endpoints, account/SID/session metadata | [SMB/RPC](../01-Enumeration/SMB-RPC.md) |
| smbclient, smbserver | SMB file client / local share server | [Transfers](../03-Access/File-Transfers.md) |
| services, reg | Service and registry administration | [Windows services](../05-Windows/Services-Tasks.md) |
| psexec, smbexec, wmiexec, atexec, dcomexec | Distinct remote execution transports and triggers | [Lateral movement](../06-Active-Directory/Lateral-Movement.md) |
| GetNPUsers, GetUserSPNs | AS-REP / service-ticket roasting | [Roasting](../06-Active-Directory/Roasting.md) |
| getTGT, getST, describeTicket, ticketConverter, ticketer | Ticket request, inspection, conversion and forgery | [Authentication](../06-Active-Directory/Authentication.md), [delegation](../06-Active-Directory/Delegation.md) |
| findDelegation, addcomputer | Delegation discovery / account creation with effective rights | [Delegation](../06-Active-Directory/Delegation.md) |
| dacledit, owneredit | Read/backup/write DACL or change owner on a selected object | [ACLs](../06-Active-Directory/ACLs.md) |
| changepasswd, smbpasswd | Password change/reset using supported protocols and rights | [Credentials](../06-Active-Directory/Credentials.md) |
| secretsdump | Local artifacts, remote secrets or directory replication depending on options | [Credentials](../06-Active-Directory/Credentials.md) |
| Get-GPPPassword | Retrieve/parse GPP credential XML | [Credentials](../06-Active-Directory/Credentials.md) |

For ACL changes, specify the exact principal and target DN, inspect inheritance, back up the descriptor, then verify effective access. An inherited ACE can affect descendants; restoring only the parent may not restore every deliberately modified child. `ntds.dit` is the filename; `ndts.dit` in a source command is a typo.

## Titanis tools

The supplied source lists this .NET tool family; preserve its functions without treating historical SDK installers as a prerequisite. Do not purge other SDK versions to reproduce a transcript.

| Tool | Operations retained from source | Distinguish |
|---|---|---|
| CredCoerce | Request selected remote authentication behavior | Endpoint, credentials and target service/patch conditions. |
| Epm | Enumerate RPC endpoint registrations | Registered endpoint does not imply authorization to invoke it. |
| Kerb | `asreq`, `getasinfo`, `tgsreq`, `renew`, `select`, conversion, `changepw`, `setpw`, `s2k` | TGT vs service ticket; own password change vs privileged reset; correct realm/salt/enctype for string-to-key. |
| Lsa / Sam | Policy, identity and account database operations | Query rights versus mutation/reset rights. |
| Scm | Service inventory/control/configuration | Query, start/stop and configuration rights differ. |
| Smb2Client | `ls`, `get`, `put`, `enumshares`, sessions/open-files/NICs/snapshots/streams, `watch`; create/remove/link/mount operations | Confirm local/remote argument order and UNC syntax, including optional server port. Removal is not enumeration. |
| Wmi | Query/get/invoke/exec, namespaces/classes/properties/methods; backup/restore/delete | Namespace/method rights; deletion and restore change state. |

## Session and alternate-logon helpers

**Penelope:** source workflow uses F12 for its menu and `i <SESSION_ID>` to interact. Its transfer/task helpers can fetch or execute additional scripts: inspect the selected action. **pwncat-cs:** use its control menu (source Ctrl-D) and explicit upload/download operations; return to the intended session before issuing target commands.

**RunasCs:** runs with supplied credentials and a selected Windows logon type. Callback redirection and UAC-related flags depend on context; they do not create privileges the account lacks. Record local identity, network identity and logon restrictions separately. **PowerShell CLIXML:** exported credentials protected with Windows DPAPI generally require the same user/machine context to decrypt; `Import-Clixml` is not a universal credential decryptor.
