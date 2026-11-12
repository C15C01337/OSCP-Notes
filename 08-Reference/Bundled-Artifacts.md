# Bundled artifact reference

[Home](../README.md) · [Exploit registry](Exploit-Registry.md) · [Exploit selection](Exploit-Selection.md) · [Tools](Tools.md)

The original folders remain beside the handbook. Executables, exploit source, archives, macro documents and shortcuts are **not included in the Notion import**. Their methods are consolidated into the linked chapters; retain original code and license notices when adapting an artifact. No bundled exploit, binary, macro or installer was executed during this work. The separate source audit records original paths and SHA-256 hashes.

## Local PoC families

Original paths below are relative to the supplied collection, not commands to execute.

| Folder beneath `OSCP/exploits/` | Artifact / practical note | Guide |
|---|---|---|
| CVE-2014-5301-ManageEngine-ServiceDesk-Plus-7.6.0-Arbitrary-File-Upload | `CVE-2014-5301.py`; authenticated WAR/EAR packaging and upload path. Needs a compatible deployment/credentials. | [Selection](Exploit-Selection.md) |
| CVE-2015-1328-overlayfs-LPE | `37292`; C source despite extensionless name, namespace/OverlayFS-specific. | [Linux](../04-Linux/Privilege-Escalation.md) |
| CVE-2016-5195-Dirty-COW-PTRACE_POKEDATA-Race-Condition-LPE | `dirty.c`; modifies account-file contents and creates a backup; do not confuse with a read-only check. | [Selection](Exploit-Selection.md) |
| CVE-2017-0144-EternalBlue-MS17-010-RCE | Wrapper, scanner, Python SMB helpers, architecture-specific kernel blobs and historical screenshots. Wrapper/parser/dependency generations must match. | [Selection](Exploit-Selection.md) |
| CVE-2018-7600-Drupal-RCE | `poc7-FireFart.py`; Drupal 7 form/AJAX flow, not all Drupal deployments. | [Web](../02-Web/Enumeration.md) |
| CVE-2019-7214-SmarterMail-RCE | `.NET` remoting/deserialization transport and serialized payload; endpoint/runtime compatibility required. | [Serialization](Managed-Code.md) |
| CVE-2020-11651-CVE-2020-11652-Saltstack-Authentication-Bypass-and-Directory-Traversal-PoC-RCE | `exploit.py`; check/read/upload/master execution/all-minion execution are different operations. `--exec-all` broadens scope. | [Selection](Exploit-Selection.md) |
| CVE-2020-24186-WordPress-Plugin-wpDiscuz-7.0.4-RCE | `49967`; nonce discovery, upload and execution path need matching plugin/page behavior. | [Uploads](../02-Web/File-Attacks.md) |
| CVE-2021-4034-PwnKit-Pkexec-LPE-PoC-3 | Python wrapper embeds amd64 shared-library bytes; not architecture-independent because the launcher is Python. | [Selection](Exploit-Selection.md) |
| CVE-2022-24112-Apache-APISIX-2.12.1-RCE | `50829`; batch/admin API request followed by created route trigger; hardcoded content length/paths need inspection. | [Selection](Exploit-Selection.md) |
| CVE-2023-24078-FuguHub-8.4-Remote-Code-Execution | `51550.py`; bare unquoted first line is a syntax error, title/build labels conflict, later requests hardcode port 8082 despite a port argument. Account creation/login and Lua upload are separate state-changing steps. | [Selection](Exploit-Selection.md) |
| CVE-2023-4911-Looney-Tunables-LPE | `exp.c`, compiled `exp`, `gen_libc.py`, bundled `libc.so.6`; target/build/architecture-specific. | [Selection](Exploit-Selection.md) |
| FreeSWITCH-1.10.1-RCE | `47799.py`; event-socket authentication/command path. | [Selection](Exploit-Selection.md) |
| HP-Power-Manager-Administration-Universal-Buffer-Overflow | `hpm_exploit.py`; legacy Python 2 and embedded return addresses/payload. | [Buffer overflows](Buffer-Overflows.md) |
| Sonatype-Nexus-3.21.1-RCE | `49385.py`; authenticated expression injection, hardcoded target/credentials and repository change. | [Selection](Exploit-Selection.md) |
| Symfony | `secret_fragment_exploit.py`; secret/algorithm/internal signing URL and callable/serializer variants. | [Selection](Exploit-Selection.md) |
| Wifi-Mouse-1.7.8.5-RCE | `49601.py`; sends keyboard commands to download/run a named payload; requires working HTTP delivery. | [Selection](Exploit-Selection.md) |
| GodPotato-LPE, PrintSpoofer, SharpEfsPotato-LPE | Precompiled Windows tools; .NET generation, bitness, privileges and trigger still matter. | [Windows tokens](../05-Windows/Privileges-Credentials.md) |

## Helper resources

| Supplied resource | What it contains / how it is represented |
|---|---|
| `OSCP/resources/php_reverse_shell.php` | Cross-platform PHP process/socket bridge with runtime/OS conditions; [shells](../03-Access/Shells.md). |
| `OSCP/resources/shell.aspx` | Actual ASP.NET C# callback shell using native interop; hardcoded host/port. Distinct from the README's mislabeled web.config example. |
| `OSCP/resources/powercat.ps1` | PowerShell TCP/UDP channel/relay features; choose the intended mode and runtime. |
| `OSCP/resources/config.Library-ms`, `automatic_configuration.lnk` | WebDAV library plus separate PowerShell shortcut. Shortcut downloads Powercat and connects to a fixed lab address; both address and consumer action matter. |
| `OSCP/resources/linux/linux-exploit-suggester.sh` | Local candidate database/feature checks; suggestions are not demonstrated vulnerabilities. Preserve its database rather than copying hundreds of historical package entries into the main workflow. |
| `OSCP/resources/windows/PowerView.ps1` | Directory/search/SID conversion, ACLs, GPO, sessions/shares, trusts, user-location and mutation helpers. Match this module generation's function names and rights. |
| `OSCP/resources/windows/PowerUp.ps1` | Service/file/registry/task, DLL, unattended/GPP/IIS, installer and UAC checks/helpers. Read-only checks differ from Invoke-ServiceAbuse, account/MSI writers and configuration changes. |
| `OSCP/resources/windows/PrivescCheck.ps1` | Gzip/Base64-loaded script modules. Decoded as text for triage; covers token, service/driver, task, policy, credential, network, MSI and leaked-handle checks. Encoded packaging is not proof of a harmless script. |
| `OSCP/resources/windows/spray-passwords.ps1` | Policy/account enumeration and online authentication loops; policy-parsing assumptions and lockout risks remain. |
| Other Windows `.exe` files | PsExec variants, Rubeus, RunasCs variants, Seatbelt, SharpKatz, SharpView, Mimikatz, Netcat and compiled helpers. Indexed by purpose in [tools](Tools.md); no claim of binary integrity beyond recorded source hashes. |
| `adduser.c`, `shell.c` | Hardcoded account-creation and encoded callback launchers. They inherit execution rights; they do not grant privilege by compilation. |
| `OSCP-notes/.gitbook/assets/juicy-potato-master.zip` | 67 archive entries: COM/token source, CLSID lists by historical OS, tests, build files and license. GetCLSID joins AppID/LocalService data, exports CSV/list and opens Out-GridView; omit the GUI step for headless use. |
| `OSCP-notes/.gitbook/assets/test.doc` | Legacy macro-bearing Office document; represented by [client-side prerequisites](../03-Access/Client-Side-Delivery.md), not imported/executed. |
| Field-manual Python helpers | Offline notebook initializer and path-checker/tests. Their behavior is replaced here by plain Markdown worksheets and the handbook's independent audit. |
| SysReptor setup shell script | Installs/fetches/starts software and imports report designs. See [reporting](../09-Reporting/Evidence.md); not executed. |

## Additional checks exposed by helper modules

The bundled enumeration libraries retain leads beyond the short manual triage: MSI custom-action security context, writable extracted MSI binaries, named-pipe permissions, leaked process/file handles, driver hash matches/co-installers, Point-and-Print policy, hardened UNC paths, WLAN profiles, proxy auto-configuration, Defender exclusions, SCCM NAA/cache secrets, PowerShell transcription, LSA protection/Credential Guard, BitLocker/Secure Boot and session/startup history. Identify the specific readable/controllable object and privileged consumer before promoting any result to a path. A protection-status check or known-driver hash alone is not escalation.
