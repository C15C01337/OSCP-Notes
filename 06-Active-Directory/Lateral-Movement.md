# Remote access and lateral movement

[Home](../README.md) · [AD workflow](Workflow.md) · [Authentication](Authentication.md) · [Credentials](Credentials.md) · [Tunnels](../07-Pivoting/Tunnels.md)

## Select a service that the identity can use

| Method | Typical network requirement | Authorization to verify |
|---|---|---|
| SMB files | TCP 445 | Share and filesystem access. |
| PsExec/service creation | SMB + service control | Administrative/service-control rights, writable staging path/share. |
| WMI over DCOM | TCP 135 + dynamic RPC; output retrieval may use SMB | WMI namespace/process creation rights, commonly local administration. |
| WinRM / PowerShell remoting | TCP 5985/5986 | Allowed remoting endpoint and account rights. |
| RDP | TCP 3389 | Remote interactive logon rights and session policy. |
| DCOM application | RPC/DCOM plus application endpoints | Activation/launch rights and matching application/configuration. |
| Database/application service | Its listener | That service's role/permissions, independent of OS administration. |

Authentication success and remote execution rights are separate. PTH itself is not limited to administrators; **PsExec's execution path** usually is. The same distinction applies to Kerberos tickets.

## Plain credential and NT hash examples

**Kali / Bash; tools may prompt for an omitted password:**

```bash
impacket-wmiexec '<DOMAIN>/<USER>@<HOSTNAME>'
impacket-psexec '<DOMAIN>/<USER>@<HOSTNAME>'
impacket-wmiexec -hashes ':<NT_HASH>' '<DOMAIN>/<USER>@<HOSTNAME>'
impacket-psexec -hashes ':<NT_HASH>' '<DOMAIN>/<USER>@<HOSTNAME>'
evil-winrm -i <HOSTNAME> -u '<USER>' -H '<NT_HASH>'
```

Impacket's `-hashes` uses `LM:NT`; a leading colon supplies no LM hash. Evil-WinRM's `-H` takes the NT hash. Set local/domain context deliberately. Remote UAC restrictions can affect local administrator accounts even when their secret is valid.

Historical alternatives include winexe/pth-winexe, smbexec, atexec, SCShell and NetExec execution options. Service creation, scheduled-task execution and service reconfiguration have different artifacts and rights requirements. Check the actual implementation and save any created service/task names.

## Sysinternals PsExec

**Windows / CMD:**

```batch
PsExec64.exe \\<HOSTNAME> -u <DOMAIN>\<USER> cmd.exe
```

PsExec typically stages its service binary, creates/starts a service and launches the requested process. A usable ADMIN$ path, service-control access and firewall configuration matter. Microsoft PsExec does not have the same hash argument as Impacket's tool. Use `-i` only when an interactive session is intended and available.

## PowerShell remoting

**Windows / PowerShell:**

```powershell
$cred = Get-Credential '<DOMAIN>\<USER>'
Invoke-Command -ComputerName <HOSTNAME> -Credential $cred -ScriptBlock { whoami; hostname }
$session = New-PSSession -ComputerName <HOSTNAME> -Credential $cred
Enter-PSSession -Session $session
```

`PSSession` alone is not the session-creation cmdlet. Reuse a saved session for stateful commands:

```powershell
Invoke-Command -Session $session -ScriptBlock { $processes = Get-Process }
Invoke-Command -Session $session -ScriptBlock { $processes }
Remove-PSSession $session
```

`Invoke-Command -Session ... -FilePath <LOCAL_SCRIPT>` runs a local script through that session; avoid combining conflicting `-ComputerName` and `-Session` parameter sets. Administrators/Remote Management Users membership is a lead; endpoint ACLs and policy decide actual permission. A remoting session may not carry credentials for a second network hop.

`winrs -r:<HOSTNAME> -u:<DOMAIN>\<USER> <COMMAND>` is a native WinRM client. Do not enable unencrypted authentication or trust every host simply to troubleshoot one connection; identify the actual authentication/name/trust issue.

### Second-hop access

If A opens a remoting session to B and a command on B accesses C, the first logon does not necessarily delegate credentials for C. Diagnose the failing hop using the identity, selected ticket/logon context, endpoint and network route. Explicit eligible credentials or correctly configured delegation can change the result, but nesting `Invoke-Command` alone does not solve it. A ticket imported into a different logon session may also be invisible to the process making the second-hop request.

## WMI process creation

**Windows / PowerShell, DCOM session:**

```powershell
$cred = Get-Credential '<DOMAIN>\<USER>'
$options = New-CimSessionOption -Protocol Dcom
$session = New-CimSession -ComputerName <HOSTNAME> -Credential $cred -SessionOption $options
Invoke-CimMethod -CimSession $session -ClassName Win32_Process -MethodName Create -Arguments @{CommandLine='cmd.exe /c whoami > C:\Users\Public\identity.txt'}
Remove-CimSession $session
```

A successful method return/process ID is not the command's stdout. Retrieve the expected output through an available channel and verify identity. Legacy `wmic /node:... process call create ...` may be unavailable on newer systems.

## RDP and restricted administration

**Kali:** use the installed FreeRDP client's help (`xfreerdp` or `xfreerdp3`) for syntax:

```bash
xfreerdp /v:<HOSTNAME> /u:<USER> /d:<DOMAIN> +clipboard
```

RDP PTH additionally relies on supported Restricted Admin behavior and suitable account rights. Setting `DisableRestrictedAdmin` is a privileged configuration change, not a bypass for an invalid hash or missing login right. Record/restore the original value if deliberately changed in a lab. Session-policy changes such as `fSingleSessionPerUser` also require existing privilege; they do not establish a new authentication path.

## Overpass-the-hash and pass-the-key

**Kali:** request a TGT using accepted key material, then select its cache:

```bash
impacket-getTGT '<DOMAIN>/<USER>' -hashes ':<NT_HASH>' -dc-ip <DC_IP>
export KRB5CCNAME="$(realpath <USER>.ccache)"
impacket-wmiexec -k -no-pass '<DOMAIN>/<USER>@<HOST_FQDN>'
```

Use `-aesKey` when holding the matching AES key and supported by the tool. RC4 may be disabled; possessing an NT hash then does not guarantee a TGT. Use the target FQDN/SPN and a working time/DNS context. The account still needs the remote method's rights.

On Windows, Mimikatz `sekurlsa::pth /user:<USER> /domain:<DOMAIN> /ntlm:<NT_HASH> /run:powershell.exe` changes credentials used by a spawned logon context under suitable privileges. The displayed local user and network authentication context may differ. Verify with an actual authorized service request.

## Pass-the-ticket

**Mimikatz:** `kerberos::ptt <TICKET.kirbi>`. **Rubeus:** `ptt /ticket:<TICKET>`. On Kali, convert/select a suitable ccache and use a Kerberos-capable client with `-k -no-pass` where supported.

A TGT and a service ticket have different scopes. Exporting/importing a ticket is not automatically Domain Admin access. The source claim that a TGT can only be used on the machine where it was created is too broad; ticket validity, account restrictions, cache/logon context and service matching are the relevant limits.

## DCOM applications

The source MMC example instantiates `MMC20.Application` remotely and calls `ExecuteShellCommand`. Other examples use Excel automation/macros. They require eligible remote activation and application configuration; they are not generic RPC-port-to-shell recipes. Confirm the target component and user context, then record the launched process and its output separately.
