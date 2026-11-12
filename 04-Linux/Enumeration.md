# Linux local enumeration

[Home](../README.md) · [Privilege escalation](Privilege-Escalation.md) · [Passwords](../03-Access/Passwords.md) · [Tunnels](../07-Pivoting/Tunnels.md)

## Establish context

**Target / Bash:**

```bash
id
hostname
uname -a
cat /etc/os-release
cat /etc/passwd
sudo -l
```

Record real/effective identity, groups, architecture, distribution and kernel. `/etc/passwd` fields are `name:password:UID:GID:GECOS:home:shell`. `x` in the password field normally points to shadow authentication; UID 0 is privileged regardless of the account's name. Service/user UID allocation ranges vary by distribution.

## Fast investigation order

| Area | Check | What makes it useful |
|---|---|---|
| Delegated commands | `sudo -l`, relevant doas policy | An allowed privileged command with an exploitable operation or input. |
| User/application files | Home, history, webroot, backups, configuration | Credentials or writable files consumed by another identity. |
| Services/processes | Executable, version, owner, command line | A privileged process using controllable code/configuration. |
| Scheduled execution | Cron, timers, scripts and dependencies | A privileged job will execute a writable component. |
| SUID/SGID/capabilities | Ownership, mode, mount and program behavior | A program actually retains useful privilege. |
| Network/filesystems | Loopback listeners, routes, exports, disks | Another reachable service, credential store or trust boundary. |
| Kernel | Exact build, patch/configuration, available PoC | All exploit prerequisites match, beyond just a version banner. |

## Users and credentials

```bash
ls -la ~ /home
env
ls -la /var/backups /opt /srv 2>/dev/null
```

Inspect relevant shell/database history and application configuration. Environment variables, process arguments and backup scripts can expose credentials. Avoid dumping every readable file into one unsearchable output; target a product or path first. Verify recovered credentials against the intended account/service.

## Processes and software

```bash
ps auxww
ps -eo user,pid,ppid,args
dpkg -l
rpm -qa
```

Use the package manager present on the target. Match executable paths and package versions; vendor backports can patch an old upstream version string. `watch -n 1 'ps auxww'` can reveal periodic processes, while pspy may capture short-lived executions without root, subject to visibility restrictions. A privileged process is a lead only when its inputs or implementation can be abused.

## Network and filesystems

```bash
ip addr
ip route
ss -lntup
cat /etc/fstab
mount
lsblk
lsmod
```

Missing process details in `ss` may be a permissions limitation. Inspect local-only services through [forwarding](../07-Pivoting/Tunnels.md). Configuration such as `/etc/iptables/rules.v4` may be readable, but it may not reflect all active firewall state. For an interesting loaded module, `/sbin/modinfo <MODULE>` supplies metadata.

## Permissions and privileged executables

```bash
find /etc -maxdepth 1 -type f -writable 2>/dev/null
find /opt /srv /usr/local -type d -writable -executable 2>/dev/null
find / -type f \( -perm -4000 -o -perm -2000 \) -ls 2>/dev/null
getcap -r / 2>/dev/null
getfacl <PATH>
```

Expand the path scope when the focused checks leave a gap. Inspect both the file and every relevant parent directory: replacing a file can depend on directory permissions even when the file itself is not writable. ACLs can grant access beyond the basic mode bits. Mount options such as `nosuid` and `noexec` can change the usable path.

## Scheduled execution

```bash
crontab -l
cat /etc/crontab
ls -la /etc/cron* /var/spool/cron 2>/dev/null
systemctl list-timers --all
```

System crontab fields include a user; user crontabs do not. Record the actual execution account, schedule, environment and invoked script. Check `/etc/anacrontab`, systemd unit definitions and logs where applicable. Lack of read access to another user's crontab does not prove that no job exists.

## Enumeration aids

LSE, LinEnum, LinPEAS, unix-privesc-check and Linux Exploit Suggester summarize different evidence. Save output and manually verify a promising result. LSE verbosity progresses from level 0 to 2 and supports selecting test groups such as users, sudo, filesystems, scheduled tasks, services and containers. Use its local help for exact IDs.

Download a script, inspect it and choose options deliberately. A broad `-a` flag can do more than the checks you intended. A highlighted result is not an exploited path or proof of current exam permission.

## Focused file and group checks

```bash
find /home -type f -user <USER> -readable 2>/dev/null
find /opt -type f -group <GROUP> 2>/dev/null
find /var -type f -cmin -60 2>/dev/null
find /var -type f -amin -60 2>/dev/null
capsh --print
```

`ctime` is inode/status change time, not creation time; `mtime` concerns file content and `atime` access, subject to mount/update policy. GNU find `-size 50M` uses rounded units rather than a universal exact-byte comparison.

Interesting groups need a concrete resource: `shadow` can expose hashes, `disk` raw block access, `docker` a daemon socket, `lxd` container/image/device operations and `video` device/framebuffer access where configured. `wheel`/`sudo` still require reading the applicable delegation policy. Group names alone do not prove a usable path.

Existing tmux/screen session sockets and same-user process injection/ptrace are additional source leads. Check socket/process permissions, ownership, namespace and ptrace policy; a same-user injection can affect a more useful session but does not intrinsically create root rights. Tracing a SUID process may suppress its elevated behavior.
