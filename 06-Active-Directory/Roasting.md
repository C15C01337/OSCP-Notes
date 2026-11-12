# Kerberos roasting and account discovery

[Home](../README.md) · [Authentication](Authentication.md) · [Enumeration](Enumeration.md) · [Password cracking](../03-Access/Passwords.md) · [ACLs](ACLs.md)

## Kerberos username discovery

**Kali / Bash:**

```bash
kerbrute userenum -d <DOMAIN> --dc <DC_IP> users.txt
```

Different Kerberos errors can reveal whether a principal exists. Username discovery is not proof that a password is valid or that pre-authentication is disabled. Record exact domain, spelling and result. The source NSE alternative is `krb5-enum-users` with an explicit realm and username database; include a target and verify local script arguments.

## AS-REP roasting

**Prerequisite:** a user has `DONT_REQ_PREAUTH` / "Do not require Kerberos preauthentication" set. For a known username, an attacker may request an AS-REP without knowing the password and obtain client-key-encrypted material suitable for offline guessing. It is **not** the krbtgt-encrypted TGT that is cracked for the user's password.

**Kali / Bash:**

```bash
impacket-GetNPUsers '<DOMAIN>/' -usersfile users.txt -no-pass -dc-ip <DC_IP> -format hashcat -outputfile asrep.txt
```

With a valid credential, enumerate/request eligible accounts using that installation's authenticated `GetNPUsers` options. **Windows alternatives:**

```powershell
Get-DomainUser -PreauthNotRequired
Get-ADUser -Filter 'DoesNotRequirePreAuth -eq $true' -Properties DoesNotRequirePreAuth
.\Rubeus.exe asreproast /format:hashcat /outfile:asrep.txt /nowrap
```

For an RC4 (`etype 23`) AS-REP in the matching format:

```bash
hashcat -m 18200 asrep.txt <WORDLIST>
```

Do not insert `23$` into every output indiscriminately. Preserve the actual encryption type and obtain the format expected by the chosen cracker. If no response is roastable, that does not imply the username is invalid.

## Kerberoasting

**Prerequisites:** normally a valid domain authentication context and a target account with an SPN. A requested service ticket includes material encrypted using the service account's key, enabling offline password guessing. This does not return the account's NT hash directly.

**Kali / Bash:**

```bash
impacket-GetUserSPNs '<DOMAIN>/<USER>' -dc-ip <DC_IP> -request -outputfile tgs.txt
```

**Target / PowerShell:**

```powershell
Get-DomainUser -SPN
.\Rubeus.exe kerberoast /outfile:tgs.txt /nowrap
```

PowerView `Invoke-Kerberoast`/`Request-SPNTicket` and a .NET `KerberosRequestorSecurityToken` are alternatives in the source collection. They request/export material; they do not crack it on the target simply because a function is named "Kerberoast".

For matching RC4 TGS output:

```bash
hashcat -m 13100 tgs.txt <WORDLIST>
```

AES tickets require a different mode. Rubeus `/aes`, `/rc4opsec`, account filters and formatting options are version-dependent; an option containing "opsec" does not guarantee invisibility. Computer and managed-service accounts usually have long random passwords, making offline cracking less promising than weak human-chosen service passwords.

## Targeted changes when ACLs permit them

Control of a user object may allow adding an SPN to make it roastable or changing pre-authentication settings. These are **directory modifications** requiring the relevant property rights. Record the original attribute values and restore only the values introduced by the test.

Do not blindly XOR a UAC bit: repeating XOR toggles it back, and replacing `userAccountControl` can alter unrelated flags. Likewise, preserve existing SPNs rather than overwriting the whole list. See [ACL paths](ACLs.md).

## Password spraying

Use [online password testing](../03-Access/Passwords.md#online-tests) with domain policy, fine-grained policies and existing failures considered. DomainPasswordSpray, Invoke-CleverSpray, Kerbrute, NetExec and the historical Spray script are alternatives in the collection. A single candidate across users can still lock accounts; "low and slow" is not a guarantee.

After cracking or validating a secret, confirm the account and its real service rights before attempting a privileged remote execution method.

PowerShell AD-module policy queries include `Get-ADDefaultDomainPasswordPolicy`, `Get-ADFineGrainedPasswordPolicy -Filter *` and `Get-ADUserResultantPasswordPolicy -Identity <USER>`. The default domain threshold need not be the policy applied to a particular user.

The AD map also mentions **blind/no-preauth Kerberoasting** using an eligible no-preauth principal and guessed SPNs with tool support. This is a distinct edge case from normal authenticated Kerberoasting; establish the KDC behavior, supported encryption and real service principals. Its copied `keberoast` spelling and split arguments are not valid universal invocations. CVE-2022-33679 is retained as a separate encryption/preauthentication lead in the registry.

## Timeroasting reference

The large command reference includes Timeroast and a `timecrack.py` implementation for legacy Windows NTP authenticator data associated with machine accounts. This is a different format/path from Kerberos AS-REP or TGS roasting.

The supplied cracker expects lines shaped as `RID:$sntp-ms$<32_HEX_AUTHENTICATOR>$<96_HEX_SALT>` and computes MD5 over the candidate's NT-hash bytes followed by the salt. It is a dictionary check against captured output, not a method to read plaintext passwords from any NTP server.

The modified code uses Latin-1 when opening files to avoid decoder failures and an MD4 fallback when OpenSSL lacks MD4. Encoding a wordlist differently changes candidates; preserve the intended password encoding. Its claim about Hashcat/John support is historical and has not been verified online. Use a local tool that explicitly accepts the captured format rather than assigning an unrelated Kerberos mode.
