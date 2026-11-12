# Windows privileges, credentials and other elevation paths

[Home](../README.md) · [Enumeration](Enumeration.md) · [Services](Services-Tasks.md) · [Passwords](../03-Access/Passwords.md) · [Transfers](../03-Access/File-Transfers.md)

## Token privileges

```batch
whoami /priv
whoami /groups
```

A listed privilege can be present but disabled; whether it can be enabled/used depends on the token, API and operation. Absence from the current token differs from being disabled. A primary token supplies a process identity; an impersonation token can change a thread's security context.

| Privilege | Investigation path | Limit |
|---|---|---|
| `SeImpersonatePrivilege` | Impersonate an eligible client token through a suitable service/IPC path. | Needs a usable privileged client and compatible implementation. |
| `SeAssignPrimaryTokenPrivilege` | Assign a usable primary token to a process. | Does not conjure a privileged token. |
| `SeBackupPrivilege` | Backup-aware reads of protected files/registry. | Ordinary reads need not use backup semantics. |
| `SeRestorePrivilege` | Restore-aware writes/ownership operations. | Requires appropriate API/flags and a later privileged consumer. |
| `SeTakeOwnershipPrivilege` | Take ownership, then adjust a DACL where permitted. | Ownership changes and resulting write access are separate steps. |
| `SeDebugPrivilege` | Access eligible processes/tokens. | Protection such as PPL and other controls can still block access. |
| `SeLoadDriverPrivilege` | Driver-loading control. | Needs a usable driver and policy/signature conditions. |
| `SeTcbPrivilege`, `SeCreateTokenPrivilege` | Advanced token/trusted-system operations. | Rare and API-dependent; inspect the precise primitive. |

## Impersonation tools

PrintSpoofer uses a Print Spooler/named-pipe path; JuicyPotato uses compatible COM activation/authentication behavior and a suitable CLSID. GodPotato, JuicyPotatoNG, RoguePotato, SharpEfsPotato, SweetPotato and other variants have distinct requirements. Do not select solely from an OS-major-version label.

Source examples for a matching lab:

```batch
PrintSpoofer.exe -i -c cmd.exe
JuicyPotato.exe -l 1337 -p C:\Path\To\payload.exe -t * -c <CLSID>
```

Verify privileges, architecture, required services/endpoints, patch/configuration and the installed tool's options. A COM receive error or missing CLSID is not evidence of a working path. The classic JuicyPotato behavior is limited on later Windows builds; PrintSpoofer is not guaranteed to replace it on every new build.

Hot Potato's older spoofing/NTLM-relay chain, Rotten Potato token capture, named-pipe impersonation and token duplication are related concepts with different mechanics. Meterpreter `getsystem` implements particular techniques; historical technique/architecture limits in old source descriptions should not be treated as a current exhaustive implementation list.

## Backup Operators and registry hives

**Prerequisite:** sufficient registry backup rights in the current token. Ensure the output directory exists and is writable.

```batch
reg save HKLM\SAM C:\Users\Public\SAM.backup
reg save HKLM\SYSTEM C:\Users\Public\SYSTEM.backup
```

Transfer both hives to Kali, then:

```bash
impacket-secretsdump -sam SAM.backup -system SYSTEM.backup LOCAL
```

SAM holds local account credential data; SYSTEM supplies key material for offline decryption. Reading LSA secrets/cached domain credentials also involves SECURITY and separate artifact handling. A domain controller's domain database is NTDS, not its local SAM.

Legacy backups may exist in `C:\Windows\Repair` or `C:\Windows\System32\config\RegBack`; they may be absent, empty or stale. A group membership alone does not guarantee `reg save` works in a filtered token. Alternative source tools include pypykatz, samdump2 and creddump7, with artifact/version limits.

## Search for stored credentials

```batch
cmdkey /list
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s
findstr /si password C:\Path\To\App\*.xml C:\Path\To\App\*.ini C:\Path\To\App\*.config
```

Inspect unattended setup files, application configs, PowerShell history, backups and user files. Registry values can reveal AutoLogon credentials or connection/proxy details; not every PuTTY session contains a password. Broad recursive registry searches generate much noise, so begin with relevant locations.

`cmdkey /list` identifies stored targets/accounts rather than revealing plaintext. `runas /savecred /user:<ACCOUNT> <COMMAND>` is usable only when a suitable saved credential and logon policy exist. Running a lab `savecred.bat` does not recover a missing credential from nowhere. Keep alternate-account execution, local elevation and network-only credentials distinct.

## AlwaysInstallElevated

Both policy values must be enabled:

```batch
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

If both are `1` and installer policy permits the path, an MSI can run with elevated installation privileges. Generate a matching MSI in a lab and invoke it with `msiexec /i C:\Path\To\package.msi /qn`. Record the resulting identity and installation side effects. One enabled value is insufficient.

## Autoruns, startup folders and privileged GUI applications

Inspect HKLM/HKCU `Software\Microsoft\Windows\CurrentVersion\Run` values and the all-users startup directory `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp`. A writable target can provide execution **when the relevant user logs in**. Run-key placement does not guarantee SYSTEM or a high-integrity token.

A shortcut can point to an executable, but it still needs an actual logon consumer. The source Paint/file-dialog example relies on a GUI application already running with elevated rights; a normal file dialog does not grant administrator access. Confirm the parent token and whether spawned processes retain it.

## UAC, administrator and SYSTEM

UAC bypass techniques such as the source Event Viewer example assume an eligible administrator context and matching settings/build. They do not turn an arbitrary standard account into an administrator. Opening RDP also does not grant rights to add users to Administrators.

With an already elevated administrator context, Sysinternals PsExec can launch a local SYSTEM process, for example `PsExec64.exe -s -i cmd.exe` in a suitable interactive session. This is a change of execution identity using existing administrative control, not a standard-user vulnerability.

## Kernel and application vulnerabilities

Match OS build, architecture, patch state, driver/product version and exploit prerequisites. WES-NG/Watson results are leads. Read and review the exact PoC rather than trusting a filename or precompiled binary. Prefer a verified controllable service/task/credential path when available, and capture the resulting effective identity for every successful attempt.

## Additional privilege and configuration paths

| Source path | Required primitive / correction |
|---|---|
| SeManageVolumePrivilege | A matching exploit may alter volume/file permissions; verify the actual right and build. The PrintNotify/Printconfig.dll COM load path is a separate trigger requiring compatible service behavior and writable load location. |
| SeEnableDelegationPrivilege | Relevant user right plus machine/object, SPN/DNS and delegation control. Kerberos ticket capture also needs eligible authentication. Preserve existing userAccountControl bits rather than replacing them with a copied decimal value. |
| “SeLoadDriver” VSS example | Source commands actually reconfigure the VSS service. That is a service-control path, not evidence of loading a driver. |
| SeTakeOwnership / accessibility executable | Ownership, DACL/write control, protected-file behavior and a privileged logon-screen trigger are separate. Replacing Utilman changes a system component; preserve and restore the original rather than treating it as an ordinary file proof. |
| RogueWinRM | Matching WinRM/BITS/local relay behavior, privileges, bindings and build; not a generic known-password logon. |
| CA key access | Actual key read/export rights, provider/storage and mapping. Certificate forgery is not a Kerberos golden ticket, despite the source heading. |

For GodPotato, source builds target different .NET generations; a matching example is `GodPotato.exe -cmd "cmd /c whoami"`. JuicyPotato/NG selection may require a compatible CLSID/AppID/LocalService registration; enumerate registry data without assuming an interactive `Out-GridView` session exists. An exported DLL function runs only if the loader calls it; exporting `OptionalFunction` alone does not guarantee execution when the DLL is loaded.

### Snapshot collection

Backup privileges, VSS management rights and backup-aware file reads are separate. For a compatible DC lab, a clean DiskShadow script is:

```text
set context persistent nowriters
add volume c: alias examcopy
create
expose %examcopy% x:
```

Run `diskshadow /s <SCRIPT_PATH>` on Windows with the necessary rights and a free drive letter. Read the returned snapshot details; use backup-aware copying where required (for example `robocopy /b` with a selected source directory/file). Preserve a matching SYSTEM hive for offline NTDS decryption. The source's trailing `p`/`X` characters are transcription errors, not DiskShadow syntax. Record the exact snapshot ID/drive mapping and remove only the snapshot and exposure you created.

### IIS configuration and SID reminders

Where readable, `appcmd list apppool /text:*` and application `web.config`/connectionStrings can disclose application-pool identity or connection details. This requires file/configuration access and does not guarantee stored plaintext passwords.

`S-1-0-0` is the Null SID, `S-1-1-0` Everyone, `S-1-5-11` Authenticated Users and `S-1-5-18` LocalSystem. RID 500 identifies a built-in Administrator account within its account domain. `dir /a:d` lists directories, including hidden ones; `dir /a:h` selects hidden attributes. PowerShell `Get-ChildItem -Force` includes hidden/system items where access permits.
