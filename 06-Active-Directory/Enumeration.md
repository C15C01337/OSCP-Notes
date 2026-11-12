# AD objects, LDAP and access enumeration

[Home](../README.md) · [Workflow](Workflow.md) · [Graph analysis](Graph-Analysis.md) · [ACLs](ACLs.md) · [SMB/RPC](../01-Enumeration/SMB-RPC.md)

## Native domain queries

**Domain-connected Windows / CMD:**

```batch
net user /domain
net user <USER> /domain
net group /domain
net group "Domain Admins" /domain
net accounts /domain
setspn -L <ACCOUNT>
```

Use `/domain` when you need domain policy; plain `net accounts` describes local policy. Simple group listings can miss nested/derived access. Record custom groups and users with unusual descriptions, logon settings or delegated rights.

**DNS:**

```text
nslookup -type=SRV _ldap._tcp.dc._msdcs.<DOMAIN> <DC_IP>
nslookup -type=SRV _kerberos._tcp.<DOMAIN> <DC_IP>
```

## LDAP from Kali

LDAP commonly uses 389/636 and AD global catalog 3268/3269. Retrieve RootDSE naming contexts, then query the actual base DN:

```bash
ldapsearch -x -H ldap://<DC_IP> -s base -b '' namingContexts defaultNamingContext
ldapsearch -x -H ldap://<DC_IP> -D '<USER>@<DOMAIN>' -W -b '<BASE_DN>' '(objectClass=computer)' dNSHostName operatingSystem
ldapsearch -x -H ldap://<DC_IP> -D '<USER>@<DOMAIN>' -W -b '<BASE_DN>' '(sAMAccountType=805306368)' sAMAccountName description memberOf servicePrincipalName
```

Use LDAPS/STARTTLS or a supported signed/authenticated method when the server requires it. Simple binds over unencrypted LDAP expose secrets. A RootDSE read does not prove that anonymous subtree queries are allowed. "A successful bind must be completed" can mean the operation needs authentication; it does not always mean that a tested password was wrong.

DNs such as `CN=Users` and `CN=Computers` are not universal object locations. Query the domain subtree with a filter rather than assuming every account is in the default container. Resolve group members recursively where needed. `userPassword` or a description containing a secret is a lead, not guaranteed current plaintext.

Other supplied clients include ldap3, windapsearch and ldapdomaindump. Configure server, TLS, authentication and search base explicitly. A successful `bind()` and the permissions of a particular query are separate results.

## PowerView

**Target / PowerShell, after loading the reviewed script:**

```powershell
. .\PowerView.ps1
Get-Domain
Get-DomainSID
Get-DomainController
Get-DomainPolicy
Get-DomainUser -Properties SamAccountName,Description,MemberOf,ServicePrincipalName
Get-DomainGroup
Get-DomainGroupMember -Identity 'Domain Admins' -Recurse
Get-DomainComputer -Properties Name,DnsHostName,OperatingSystem
Get-DomainOU
Get-DomainGPO
Get-DomainTrust
Get-ForestDomain
```

Some source versions use `Get-NetUser`, `Get-NetComputer`, `Get-NetGroup` or `Get-NetDomain`. Do not mix parameter sets between versions. Check `Get-Command <NAME> -Syntax` in the loaded script.

| Question | PowerView function family |
|---|---|
| Readable domain shares/files | `Find-DomainShare -CheckShareAccess`, `Find-InterestingDomainShareFile` |
| Local administrative access | `Find-LocalAdminAccess`, `Test-AdminAccess` |
| Local group members | `Get-NetLocalGroupMember`, `Find-DomainLocalGroupMember` |
| Active/logged-on sessions | `Get-NetSession`, `Get-NetLoggedon`, `Find-DomainUserLocation` |
| GPO-driven local groups | `Get-DomainGPOLocalGroup`, `Get-DomainGPOComputerLocalGroupMapping` |
| GPOs affecting a computer | `Get-DomainGPO -ComputerIdentity <HOST>` |
| Object/file ACLs | `Get-DomainObjectAcl -ResolveGUIDs`, `Find-InterestingDomainAcl`, `Get-PathAcl` |
| Domain/forest relationships | `Get-DomainTrustMapping`, `Get-ForestTrust` |

Session and local-group queries often need extra remote rights and may be incomplete. A discovered privileged session does not guarantee extractable credentials. Test a useful edge rather than assuming that every graph relationship can be exercised.

## ActiveDirectory module

Requires the module/RSAT or a compatible installed provider. **PowerShell:**

```powershell
Import-Module ActiveDirectory
Get-ADDomain
(Get-ADDomain).DomainSID
Get-ADDomainController -Filter *
Get-ADUser -Filter * -Properties Description,MemberOf,ServicePrincipalName
Get-ADUser -Identity <USER> -Properties *
Get-ADGroup -Filter *
Get-ADGroupMember -Identity 'Domain Admins' -Recursive
Get-ADComputer -Filter * -Properties OperatingSystem,DNSHostName
Get-ADTrust -Filter *
(Get-ADForest).Domains
```

Use `-Filter` or `-Identity` in the appropriate parameter set, not both as the original example does. `Get-DomainSID` belongs to PowerView, not the AD module. `Get-ADObject -Filter 'badPwdCount -gt 0'` can expose prior failures where readable, but count replication and fine-grained policies still matter for lockout decisions.

## .NET LDAP without extra modules

**Domain-connected Windows / PowerShell:**

```powershell
$root = [ADSI]'LDAP://RootDSE'
$base = [ADSI]("LDAP://" + $root.defaultNamingContext)
$search = New-Object DirectoryServices.DirectorySearcher($base)
$search.PageSize = 1000
$search.Filter = '(sAMAccountType=805306368)'
$search.FindAll() | ForEach-Object { $_.Properties }
```

Change the filter to `(objectClass=group)`, `(sAMAccountName=<USER>)` or `(servicePrincipalName=*HTTP*)` as appropriate. Escape user-controlled LDAP filter values when building queries. `objectClass=user` can include computers; `sAMAccountType` helps distinguish normal user accounts. Paging avoids silently limiting a large result set.

## Preserve usable results

Additional native checks are `nltest /dclist:<DOMAIN>`, `gpresult /r` and the current logon server (`$env:LOGONSERVER` in PowerShell, `echo %LOGONSERVER%` in CMD). `gpresult` shows applied policy in the current context, not every GPO defined in the directory.

In Active Directory Users and Computers, **View → Advanced Features** exposes extra containers and object property tabs such as Attribute Editor/Security where available. Inspect `servicePrincipalName` there or use `setspn -L <ACCOUNT>`. `setspn -Q <PATTERN>` searches registrations; `setspn -S <SPN> <ACCOUNT>` modifies one and requires suitable rights. Registration and a running reachable service are separate facts.

Managed service-account setup commands (`New-ADServiceAccount`, `Add-ADComputerServiceAccount`, `Install-ADServiceAccount`) describe administration, not a default attack path. Managed passwords are different from a blank password chosen by a person.

```powershell
Get-DomainUser | Export-Clixml .\domain-users.xml
$users = Import-Clixml .\domain-users.xml
$users | Select-Object SamAccountName,Description
```

Exported objects support later offline filtering; they are a snapshot and do not confer live AD rights. Record query time, DC, identity and limitations. Re-query the exact object before a modification.
