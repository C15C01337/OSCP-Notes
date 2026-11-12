# Windows services, DLLs and scheduled tasks

[Home](../README.md) · [Enumeration](Enumeration.md) · [Privileges](Privileges-Credentials.md) · [Payload generation](../03-Access/Metasploit.md)

## Prove the complete path

You need a controllable component, a more privileged consumer and an execution trigger. Querying a service does not imply permission to reconfigure or restart it.

**CMD or PowerShell; use `sc.exe` explicitly because `sc` can be a PowerShell alias:**

```batch
sc.exe qc <SERVICE>
sc.exe query <SERVICE>
sc.exe sdshow <SERVICE>
icacls "C:\Path\To\service.exe"
icacls "C:\Path\To"
```

Record service account (`SERVICE_START_NAME`), executable/arguments, start type, current state and ACLs. `qc` shows configuration; `query` shows status. Service-object rights, registry-key rights and executable-file rights are separate.

## Insecure service configuration

**Prerequisite:** permission such as `SERVICE_CHANGE_CONFIG` or equivalent effective control over a privileged service.

Save the original configuration, then a lab reconfiguration pattern is:

```batch
sc.exe config <SERVICE> binPath= "C:\Path\To\reviewed-service-payload.exe"
```

The space after `binPath=` is significant. Confirm the new path and trigger only if permitted. A normal console executable may perform its action and then report a service startup error because it does not implement the service-control protocol. Verify its side effect or returned shell rather than relying solely on the service status.

## Writable service executable or directory

**Prerequisites:** write/replace permission on the actual executable path, a privileged service account and a usable trigger.

Back up the original executable with its path/permissions/hash. Place a compatible payload at the exact path, restart or wait for an appropriate trigger, confirm resulting identity and restore the original. Replacing a file currently held open may fail even when an ACL grants write access.

AccessChk can evaluate file, directory, service and registry permissions; options differ by release. PowerUp functions such as `Get-ModifiableServiceFile` help identify candidates. `Install-ServiceBinary` and `Write-ServiceBinary` are modifying functions and can create accounts or replace files; inspect the loaded script's defaults first.

## Weak service registry ACL

**PowerShell:**

```powershell
Get-Acl 'HKLM:\SYSTEM\CurrentControlSet\Services\<SERVICE>' | Format-List
```

Write access to `ImagePath` can expose a service path even if direct service-manager reconfiguration is denied. Record the original value and registry type. Still verify the service account and trigger; a writable registry value alone is not execution.

## Unquoted executable paths

An unquoted service executable path containing spaces can be interpreted at earlier executable boundaries. For example:

```text
C:\Program Files\Example App\service.exe
```

May cause attempts such as `C:\Program.exe` and `C:\Program Files\Example.exe` before the intended binary, depending on how the process is created.

**Required checks:** the stored command is truly unquoted; the executable portion contains spaces; the selected earlier path is writable; the service runs with useful privilege; and a trigger exists. A space only in an already-separated argument is not enough. Use `icacls` on each relevant directory and confirm actual resolution instead of planting files blindly.

PowerUp's `Get-UnquotedService` identifies candidates, not confirmed exploitation. Record any planted executable so it can be removed.

## DLL replacement and search-order hijacking

**Prerequisites:** a privileged process loads a controllable DLL path/name, matching architecture/exports, and a trigger to load it.

Use Procmon in a controlled reproduction to filter the target process and DLL load/file lookup events. `NAME NOT FOUND` identifies a search attempt; confirm the application will actually load a DLL from a writable searched path. A missing DLL name alone is insufficient.

Search order depends on loader flags, application packaging, KnownDLLs, Safe DLL Search Mode and explicit paths. The source's six-directory list is simplified rather than universal. A FileZilla `TextShaping.dll` example applies only to a matching application behavior.

DLL payloads need the correct bitness and sometimes expected exports. Starting complex work directly in `DllMain` can deadlock under the loader lock; do not assume the source's minimal `system()` example is reliable for every process. Back up an existing DLL before replacement.

## Scheduled tasks

```batch
schtasks /query /fo LIST /v
schtasks /query /tn "\<TASK_PATH>\<TASK_NAME>" /fo LIST /v
```

Inspect action executable/script, arguments, working directory, principal, run level, next trigger and file ACLs. A writable task script is useful if a more privileged principal will run it. Preserve normal script behavior while proving execution. An unlisted task may still leave readable scripts/logs; lack of task visibility is not absence.

## Trigger and recovery

```batch
net stop <SERVICE>
net start <SERVICE>
```

Start/stop rights may be absent. Auto-start configuration plus an available reboot can be another trigger, but rebooting interrupts shells and other work. Check the actual start mode, permissions and restoration plan first. A built-in restart command does not establish authorization to use it in every exam environment.
