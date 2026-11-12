# Windows and domain credential material

[Home](../README.md) · [AD workflow](Workflow.md) · [Authentication](Authentication.md) · [Lateral movement](Lateral-Movement.md) · [Windows backup rights](../05-Windows/Privileges-Credentials.md#backup-operators-and-registry-hives)

## Select the artifact you actually have

| Source | Material | Required context |
|---|---|---|
| SAM + SYSTEM | Local account hashes | Readable copies or usable registry backup rights. |
| SECURITY + SYSTEM | LSA secrets/cached logon data | Matching hives and appropriate decryptor. |
| LSASS memory/dump | Available logon secrets, keys, tickets | Process access and relevant secrets present; protection may prevent extraction. |
| NTDS + SYSTEM | Domain database credential data | Readable consistent database copy and key material. |
| Replication interface | Selected domain credential data | [Directory replication rights](Trusts-Persistence.md#dcsync). |
| Credential Manager/DPAPI files | Encrypted credentials and master keys | Correct user/system context or usable recovery material. |
| SYSVOL/GPP | Historical `cpassword` values or scripts | Readable relevant policy files; not every policy contains a secret. |
| App/browser/password-manager files | Product-specific saved secrets | Correct files, keys, runtime and account context. |

Do not treat cached logon verifiers, NT hashes, service keys and NetNTLM responses as equivalent formats.

## Mimikatz: current logon material

**Inside Mimikatz on the relevant Windows host:**

```text
privilege::debug
sekurlsa::logonpasswords
sekurlsa::ekeys
sekurlsa::tickets /export
```

`privilege::debug` attempts to enable an available privilege; it does not grant one to an arbitrary account. `token::elevate` can impersonate an accessible higher token under suitable rights; `token::revert` returns to the prior context. Local administrator/SYSTEM commonly provides useful access, but PPL, Credential Guard and other protections can still prevent extraction. Plaintext is not guaranteed to exist.

Other source commands include `vault::list`, `vault::cred`, `lsadump::sam`, `lsadump::secrets`, `lsadump::cache`, `sekurlsa::tspkg` and `sekurlsa::dpapi`. Select the artifact/question rather than running a long modifying command chain by default.

## Offline LSASS dump analysis

**Mimikatz:**

```text
sekurlsa::minidump C:\Path\To\lsass.dmp
sekurlsa::logonpasswords
```

`sekurlsa::minidump` **selects an existing dump for analysis**; it does not create that dump. Acquire a dump only through a separately understood method with sufficient rights. Match analyzer architecture/version to the artifact.

**Kali / Bash:**

```bash
pypykatz lsa minidump lsass.dmp
```

Do not infer a credential is absent solely because one parser fails. Record the dump origin, host/build, acquisition method and parser error. Treat every dump as sensitive material.

## Protected LSASS and loading code

The sources discuss checking `RunAsPPL`, the `lsaiso.exe` process, a Mimikatz driver, PPLdump and custom SSPs. Registry/process indicators help identify protection but are not a complete state assessment. Removing process protection or loading a driver requires existing powerful rights and compatible policy/build. A historical bypass is not universally applicable.

In-memory SSP injection (`misc::memssp`) and custom SSP registration change authentication handling. They are persistent/monitoring techniques, not a way to decrypt every credential already protected by Credential Guard. Keep them with [extended persistence](Trusts-Persistence.md#other-persistence-mechanisms), including cleanup and exact platform requirements.

## SAM and LSA hives

Use the [local backup workflow](../05-Windows/Privileges-Credentials.md#backup-operators-and-registry-hives). On Kali:

```bash
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
impacket-secretsdump -sam SAM -security SECURITY -system SYSTEM LOCAL
```

A SAM extraction provides local account hashes, not every domain account's password. The NTDS database is binary; `type C:\Windows\NTDS\ntds.dit` is not a credential extraction method.

## Volume shadow copies and NTDS

**Prerequisites:** appropriate rights, a usable snapshot/backup mechanism and access to the database/key material. A snapshot can expose a consistent copy of locked files; it does not grant read permission automatically.

```batch
vssadmin list shadows
reg save HKLM\SYSTEM C:\Users\Public\SYSTEM.backup
```

Use `diskshadow` commands inside its console or a reviewed script. Source snapshot tools differ by Windows edition and privilege context. Backup-aware copying may be needed when ordinary `copy` is denied; the SeBackupPrivilege cmdlets are one source example.

After obtaining the matching database and SYSTEM copy:

```bash
impacket-secretsdump -ntds ntds.dit -system SYSTEM.backup LOCAL
```

Record snapshot identifiers, exposed drive letters and created files. Clean up only the specific artifacts you created. Creating a shadow copy is not inherently a persistence method or a guarantee of avoiding detection.

## DPAPI and Credential Manager

Candidate locations include `%APPDATA%\Microsoft\Credentials`, `%LOCALAPPDATA%\Microsoft\Credentials` and `%APPDATA%\Microsoft\Protect\<SID>`. Master-key files and encrypted credential blobs must be matched correctly.

**Mimikatz pattern:**

```text
dpapi::cred /in:<CREDENTIAL_FILE>
dpapi::masterkey /in:<MASTERKEY_FILE>
dpapi::cache
```

Decryption requires suitable context/key material. The source `/rpc` workflow depends on eligible domain recovery and authentication; it is not a universal offline decrypt option. Re-run the credential decode only after obtaining the matching master key.

DonPAPI, LaZagne, Firefox Decrypt, SharpChromium, KeePwn and related tools target different credential stores. Their presence in a tool list does not establish access to every user's secrets. KeePass files belong to the [password-manager cracking workflow](../03-Access/Passwords.md#offline-cracking).

## Group Policy Preferences

Inspect accessible SYSVOL policy XML and scripts for credential material. Historical GPP `cpassword` values can be decoded with a matching GPP decryptor, for example a local `gpp-decrypt` utility. Preserve the account, file path and full value. A policy file may be stale; confirm the credential on the intended host/service.

Internal Monologue uses an SSPI/authentication path to obtain challenge-response material without directly reading LSASS in its described context. Treat its output as the actual NetNTLM format and choose the matching cracker. It does not turn every captured response into an NT hash, and the source's custom challenge/options are not portable across all releases.

## Save evidence before moving

For each recovered item, keep source, target, account, exact format and successful test. Use a selected hash/ticket instead of a domain-wide dump when it answers the current objective. Proceed to [lateral movement](Lateral-Movement.md) only after identifying the account's effective rights.
