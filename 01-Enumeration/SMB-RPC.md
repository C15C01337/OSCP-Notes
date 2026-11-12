# SMB, NetBIOS and Windows RPC

[Home](../README.md) · [File services](File-Services.md) · [Workflow](../00-Start/Workflow.md)

## Start with identity and shares

SMB commonly listens on TCP 445 and may use NetBIOS on 139. MSRPC endpoint mapping commonly uses TCP 135; RPC over HTTP may use 593. An open port does not establish anonymous access.

**Kali / Bash:**

```bash
nmap -sV -p 139,445 --script smb-os-discovery,smb-protocols,smb2-security-mode <IP>
nbtscan <SUBNET>
smbclient -N -L //<IP>
smbclient -N //<IP>/<SHARE>
rpcclient -U '' -N <IP>
enum4linux -a <IP>
```

Test anonymous and guest behavior separately. Some servers map failed logins to guest; a nominal login success is not proof that the supplied account was authenticated. Record the effective identity and actual read/write access.

## Authenticated access

```bash
smbclient -L //<IP> -U '<DOMAIN>\<USER>'
smbclient //<IP>/<SHARE> -U '<DOMAIN>\<USER>'
smbmap -H <IP> -u '<USER>' -p '<PASSWORD>' -d '<DOMAIN>'
enum4linux -a -u '<USER>' -p '<PASSWORD>' <IP>
```

In `smbclient`, use `ls`, `cd`, `get`, `put` and `help`. For multiple downloads:

```text
recurse ON
prompt OFF
mget *
```

Start with relevant directories before recursively copying everything. Look for deployment files, scripts, backups, configuration, password files and usernames. A writable share may feed a web directory, service, scheduled task or user workflow; establish that connection before uploading a payload.

### SMBMap search and file operations

The longer source article adds these options; inspect local help because releases differ:

| Option | Purpose / consequence |
|---|---|
| `--host-file <FILE>` | Explicit host list. |
| `-r` / `-R`, `--depth` | Directory listing / recursive listing and depth control. |
| `--exclude` | Skip selected shares. |
| `-A <REGEX>` | Match/download candidate filenames. |
| `--upload <LOCAL> <REMOTE>` | Upload a file; verify resulting path/bytes. |
| `--download <REMOTE>` | Retrieve a selected file. |
| `-F <REGEX>` | Remote content-search functionality in the source version; needs execution rights and can start remote jobs. |
| `-g` | Machine-readable/grepable listing output. |
| `-L` / `--admin` | Drive information / execution-access filtering where supported. |

Filename matching and file-content search are different. The latter can execute PowerShell and start a callback SMB server in historical versions; it is not simply a read-only share listing. Preserve job/output paths for cleanup. Avoid copied smart quotes and en dashes from the article's commands.

For `smbclient`, `-c 'cd folder; ls'` sends client-console commands noninteractively. `get`, `put`, `mkdir` and `rename` need the corresponding share/file rights. Native SMB1 negotiation and any per-command legacy fallback should be recorded with the specific host.

To mount a share on Kali, using a password prompt:

```bash
sudo mkdir -p /mnt/share
sudo mount -t cifs //<IP>/<SHARE> /mnt/share -o username=<USER>,domain=<DOMAIN>,ro
sudo umount /mnt/share
```

## RPC console queries

Run these inside `rpcclient`; permissions depend on the server:

```text
srvinfo
enumdomusers
enumdomgroups
querydominfo
queryuser <RID>
querygroup <RID>
lookupnames <USER>
lookupsids <SID>
getdompwinfo
getusrdompwinfo <RID>
dsroledominfo
dsenumdomtrusts
```

These expose host/domain identity, users, groups, password policy and trusts. RID cycling resolves candidate SIDs rather than guessing passwords. `enum4linux -r` automates a default RID range; `-R <START>-<END>` selects a range. `-U`, `-G`, `-S`, `-P`, `-o` and `-i` select users, groups, shares, policy, OS and printer information.

`createdomuser`, `deletedomuser`, `lsaaddacctrights` and `lsaremoveacctrights` are modifying RPC operations, not ordinary discovery queries.

## Endpoint enumeration

```bash
impacket-rpcdump <IP>
impacket-samrdump '<DOMAIN>/<USER>@<IP>'
nmap -sV -p 135 --script msrpc-enum <IP>
```

Impacket installations may expose `rpcdump.py` and `samrdump.py` instead of the `impacket-` names. Use local help for that installation. Returned interface identifiers and named pipes identify leads; they do not themselves demonstrate exploitability.

## Protocol and vulnerability checks

Check SMB signing and supported dialects. If a legacy server requires SMB1, use a per-command client option rather than lowering every connection's security settings. A protocol negotiation error can also reflect network or authentication problems.

Historical leads include MS08-067, MS17-010/EternalBlue, Samba username-map script execution (CVE-2007-2447), writable-module loading/SambaCry (CVE-2017-7494), and Samba symlink traversal. Match the actual service, patch level and configuration. A source example that runs `smbclient logon` with shell syntax relies on a specific vulnerable server; it is not a general SMB command-execution feature.

The collections contain both CrackMapExec and NetExec examples. Tool generations and hash argument formats differ. In particular, do not assume every tool accepts `LM:NT`, `NT:LM` or a bare NT hash interchangeably. Read the installed tool's help and keep the exact accepted format with the credential record.
