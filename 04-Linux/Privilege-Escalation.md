# Linux privilege escalation

[Home](../README.md) · [Enumeration](Enumeration.md) · [Shells](../03-Access/Shells.md) · [NFS](../01-Enumeration/File-Services.md#nfs-and-unix-rpc--tcpudp-111-nfs-2049)

For every candidate, establish **control → privileged consumer → trigger → resulting identity**. Back up a file before a deliberate modification and record how to restore it.

## Sudo and delegated commands

```bash
sudo -l
```

Read the complete policy: run-as account, command path, permitted arguments, password requirement and environment handling. `NOPASSWD` applies to the specified command; membership in a group named `sudo` does not replace checking the actual policy.

Look for useful operations in the exact allowed program: command execution, editor/pager escapes, arbitrary file read/write, plugins, configuration loading or controllable arguments. GTFOBins is a reference name in the supplied material; choose its **sudo**, **SUID** or **capability** context correctly. This consolidation does not fetch its contents.

Doas is a separate delegation mechanism with its own policy; `doas -u <USER> /bin/sh` only works if the policy permits that operation. A command can be run as a non-root account and still expose that account's files or credentials.

## SUID, SGID and capabilities

An executable's SUID/SGID bit can change its effective identity on execution. It does not guarantee that the program will preserve that identity for a child shell. Check owner, execute permission, filesystem mount options and the program's privilege-dropping behavior.

```bash
ls -l <BINARY>
getcap <BINARY>
```

`cap_setuid` can allow a process to change UID if its code exposes that operation. Other capabilities grant different, narrower rights. The capability must be usable by the process, and the filesystem/runtime must honor it. A SUID shell may need privilege-preserving behavior; `bash -p` preserves an existing effective identity rather than granting one by itself.

## Scheduled tasks and process inputs

Check the script, executable, imported modules, called utilities, environment, working directory and parent directories. A writable script executed by root is a strong candidate if you can preserve its expected function and trigger/wait for the job.

For relative command names, determine the **job's** PATH. For globs/wildcards, inspect whether a program interprets crafted filenames as options. For relative config/module paths, determine the consumer's working directory. Merely changing your interactive PATH does not change another process's environment.

Confirm the trigger with logs, timestamps or a harmless marker before inserting a more complex command. A job does not run as root just because its file is root-owned; verify the scheduled execution account.

## Weak account-file permissions

| Finding | Potential use | Key limit |
|---|---|---|
| Readable `/etc/shadow` | Offline password cracking | Preserve full hash format, salt and account mapping. |
| Writable `/etc/shadow` | Replace a selected account's hash | Requires correct format and authentication policy; modifies account state. |
| Writable `/etc/passwd` | Insert a compatible hash or a UID-0 account entry | PAM/libc behavior and policy matter; avoid damaging fields or existing entries. |
| Writable sudo policy | Change delegated command rights | Syntax and policy-file validation matter; malformed rules can lock out access. |

Generate a hash with a supported algorithm, for example `openssl passwd -6` or an appropriate `mkpasswd` mode. Do not copy a truncated hash from a transcript. The source's `openssl passwd` default-DES output is not portable across tool versions. Removing the password field does not guarantee an empty-password login; policy may forbid it.

Preserve a backup and permissions, modify only the intended record, test the resulting identity and restore the original after practice. `id` is the verification; a changed file alone is not proof of escalation.

## Stored credentials and local services

Read application configuration, backup scripts, shell history and relevant process arguments. Test reuse thoughtfully across local accounts, databases and remote services. Forward an internal listener if its client/exploit is easier to run on Kali; see [tunnels](../07-Pivoting/Tunnels.md). Execution through a root-owned service still requires a matching flaw or controllable privileged input.

## NFS, containers and unusual groups

For an NFS path, identify export options and numeric ownership. `no_root_squash` on a writable export can let remote root create privileged files, but target execution, filesystem options and binary compatibility still matter. See [NFS enumeration](../01-Enumeration/File-Services.md#nfs-and-unix-rpc--tcpudp-111-nfs-2049).

An unusual group, container socket, mount or device can reveal another control surface. Verify the actual permissions and boundary before calling container-root access host-root access. See [exploitation patterns](Exploitation-Patterns.md) and the [exploit reference](../08-Reference/Exploit-Selection.md).

## Kernel and application exploits

Collect `uname -r`, architecture, distribution/package build, relevant modules and any namespace/configuration prerequisites. Match the local PoC's target assumptions, required compiler and libraries. Searchsploit can search an installed exploit database without browsing:

```bash
searchsploit linux kernel <VERSION> privilege
searchsploit -m <EXPLOIT_ID>
```

Read the copied code before execution. Compiling on the target can reduce library mismatch, but it does not make an exploit compatible. Kernel PoCs may crash or alter the system; prefer a clear permission-based path when available. Version-only suggester output remains a lead, particularly with backported security patches.
