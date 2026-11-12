# Active Directory workflow

[Home](../README.md) · [Authentication](Authentication.md) · [Enumeration](Enumeration.md) · [Graph analysis](Graph-Analysis.md)

## Establish the domain context

Record the DNS domain, domain SID, DC IPs/hostnames, starting account, password/hash/ticket format and reachable subnets. Resolve AD hostnames using the appropriate DNS server. Kerberos commonly fails when the hostname/SPN, realm or time is wrong even when the secret is correct.

| Starting evidence | First useful step |
|---|---|
| No credentials | [SMB/RPC](../01-Enumeration/SMB-RPC.md), DNS, permitted anonymous LDAP and username discovery. |
| Usernames only | Check [AS-REP prerequisites](Roasting.md#as-rep-roasting); distinguish valid usernames from crackable accounts. |
| Valid domain credential | LDAP objects, shares, policy, SPNs and controlled account relationships. |
| Windows foothold | [Local enumeration](../05-Windows/Enumeration.md), token rights, accessible credentials and internal services. |
| Local administrator | Credential material may be available; verify protections and actual artifacts. |
| Object-control edge | Inspect [ACLs](ACLs.md), group membership, delegation or certificate settings. |
| Service ticket/key/certificate | Determine exactly which identity, service and rights it can represent. |

## Iterate on evidence

1. [Enumerate](Enumeration.md) users, nested groups, computers, shares, SPNs, policies, OUs, trusts and ACLs.
2. Build a small map of **controlled principal → usable permission → target object/host**. Use [graph tools](Graph-Analysis.md) where helpful, then verify promising edges.
3. Check [roasting](Roasting.md) and recovered credentials. A crackable response is not guaranteed to crack within your available time.
4. Test access to a specific service using the [lateral-movement matrix](Lateral-Movement.md). Record authentication and authorization separately.
5. Re-enumerate after each new identity, group, local privilege, credential or network route.
6. Follow the exact objective; domain-wide extraction or persistence is not a prerequisite for every useful result.

## Choose the next path

| Finding | Reference |
|---|---|
| SPN account or pre-authentication disabled | [Roasting](Roasting.md) |
| LSASS dump, hives, NTDS, DPAPI or GPP | [Credentials](Credentials.md) |
| Password, NT hash or ticket for another host | [Lateral movement](Lateral-Movement.md) |
| GenericAll/WriteDACL/WriteOwner/group control | [ACLs](ACLs.md) |
| Delegation attributes | [Delegation](Delegation.md) |
| Certificate authority/template | [Certificate services](Certificates.md) |
| Forced authentication, name poisoning or relay | [Relay and coercion](Relay-Coercion.md) |
| Replication rights, krbtgt or trust key | [Replication, trusts and persistence](Trusts-Persistence.md) |

## Keep the credential map usable

For every secret, record the namespace (`HOST\user` versus `DOMAIN\user`), origin, format, tested service and accepted rights. A computer account normally ends in `$`; it is still a principal with specific rights. A compromised domain user is not automatically a local administrator, and local administrator is not automatically Domain Admin.

Before switching paths, save the current identity, verified edges, negative results and one next test. Keep topology and callback routes with the [pivoting notes](../07-Pivoting/Tunnels.md).
