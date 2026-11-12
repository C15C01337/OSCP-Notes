# FTP, SSH and NFS

[Home](../README.md) · [Scanning](Scanning.md) · [SMB](SMB-RPC.md) · [File attacks](../02-Web/File-Attacks.md)

## FTP — TCP 21

**Goal:** identify the server, test anonymous access, collect useful files and determine what any writable directory feeds.

```bash
nmap -sV -p 21 --script banner,ftp-anon <IP>
ftp <IP>
```

Inside the FTP client:

```text
Name: anonymous
Password: anonymous
pwd
ls -la
binary
get <REMOTE_FILE>
put <LOCAL_FILE>
bye
```

Anonymous login may use any email-like password or a blank password depending on the server. Record what worked. Use binary mode for executables and other files whose bytes must remain unchanged. Passive/active data-channel behavior can explain a successful login followed by a failed listing or transfer.

For a deliberate mirror of readable files:

```bash
wget -m ftp://anonymous:anonymous@<IP>/
```

Inspect names, hidden files, backups, application configuration and whether uploaded files appear over HTTP. FTP upload alone does not cause code execution. Local configuration leads include `ftpusers`, `ftp.conf`, `proftpd.conf` and `vsftpd.conf`. The FTP service's version and authentication policy guide any further testing.

## SSH — TCP 22

**Goal:** use a recovered credential or key, then identify the account and permissions it grants.

```bash
chmod 600 recovered.key
ssh -i recovered.key <USER>@<IP>
ssh -p <PORT> <USER>@<IP>
```

`id_rsa` and `id_ed25519` are private keys. Their `.pub` files are public keys; a public key cannot authenticate on its own. `authorized_keys` lists public keys accepted for an account. Installing a public key requires write access to the correct account's SSH directory and a server configuration that accepts it.

If the private key has a passphrase, the source workflow is to convert it with `ssh2john` and crack the resulting file with John. The converter may be installed as `/usr/share/john/ssh2john.py`:

```bash
python3 /usr/share/john/ssh2john.py recovered.key > ssh.hash
john --wordlist=<WORDLIST> ssh.hash
```

Preserve the complete key, line breaks and final newline. Check the username, key type and permissions if login fails. Common account-side permissions are `700` for `.ssh` and `600` for private keys and `authorized_keys`; ownership also matters. Legacy algorithms should be enabled only for the specific host when its handshake error identifies the required algorithm.

## NFS and Unix RPC — TCP/UDP 111, NFS 2049

**Goal:** enumerate exports, inspect accessible files and understand UID/GID mapping.

```bash
rpcinfo -p <IP>
showmount -e <IP>
nmap -sV -p 111,2049 --script rpcinfo,nfs-showmount,nfs-ls,nfs-statfs <IP>
sudo mkdir -p /mnt/nfs
sudo mount -t nfs -o ro,nolock <IP>:/<EXPORT> /mnt/nfs
ls -lan /mnt/nfs
sudo umount /mnt/nfs
```

Use `vers=3` only when the service and negotiation require it. NFSv4 may not expose exports through `showmount`; a failed listing does not establish absence of NFS access.

Inspect file ownership numerically. Traditional NFS access can depend on numeric UIDs/GIDs rather than the local display name. `root_squash` maps remote root to an unprivileged identity; `no_root_squash` can make a writable export dangerous. A privilege-escalation path additionally requires a way for the target to execute the controlled content in the intended context. Check export options and target filesystem permissions rather than assuming that a successful mount grants root.

## Useful transitions

| Finding | Follow-up |
|---|---|
| FTP or NFS application backup | Read source/configuration; test recovered credentials on the matching service. |
| Private SSH key | Recover account name, repair formatting, check passphrase, authenticate. |
| Writable web directory | Verify the web path and server execution handler using [file-upload checks](../02-Web/File-Attacks.md#file-upload). |
| Local-only service through SSH | Forward the service using the [pivoting workflow](../07-Pivoting/Tunnels.md). |
