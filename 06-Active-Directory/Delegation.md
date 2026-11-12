# Kerberos delegation

[Home](../README.md) · [Authentication](Authentication.md) · [ACLs](ACLs.md) · [Lateral movement](Lateral-Movement.md) · [Trusts](Trusts-Persistence.md)

## Distinguish the delegation model

| Model | Where permission lives | Main prerequisite |
|---|---|---|
| Unconstrained | Delegation-enabled service account/host | Control of that context and a usable delegated client TGT. |
| Traditional constrained | Delegating account's allowed service list | Control of the delegating identity and eligible S4U flow to allowed SPNs. |
| Resource-based constrained (RBCD) | Resource computer's `msDS-AllowedToActOnBehalfOfOtherIdentity` | Control of that property plus a usable controlled service principal. |

Delegation is not a universal ability to impersonate any account to every service. Sensitive/protected accounts, service keys, account flags, KDC policy and trust settings can restrict the path.

## Unconstrained delegation

Enumerate eligible accounts/computers with the loaded PowerView version, for example `Get-DomainComputer -Unconstrained` or its supported legacy equivalent. Control of the machine alone is not proof that a high-value TGT is present.

The source path is: privileged access on delegation host → eligible user authenticates with delegated credentials → accessible TGT → ticket export/import → authorized access using that identity. Rubeus `monitor`/`harvest` and Mimikatz ticket export are tools for available ticket material, not passive decryption of every network ticket.

Coercing a DC to authenticate adds another dependency: the service/protocol must delegate the expected credential and policy must allow it. Cross-forest delegation restrictions can invalidate historical chains.

## Traditional constrained delegation

Inspect `msDS-AllowedToDelegateTo` and the protocol-transition flag/context. `Get-DomainUser -TrustedToAuth` and `Get-DomainComputer -TrustedToAuth` find a subset of relevant configurations; also examine the actual allowed-service attributes.

With control of an eligible account, a source-style Rubeus request is:

```text
Rubeus.exe s4u /user:<DELEGATING_ACCOUNT> /rc4:<ACCOUNT_NT_HASH> /impersonateuser:<USER> /msdsspn:<ALLOWED_SPN> /ptt
```

Use matching key/ticket options for the installed version and encryption policy. The resulting service ticket still needs to match the service and usable authorization. Kekeo's `tgt::ask` and `tgs::s4u` represent the same stages with different syntax.

`/altservice` substitution depends on which service account key protects the ticket and how the receiving service validates it. It does not grant arbitrary access to unrelated services/hosts with different keys.

## Resource-based constrained delegation

**Required evidence:** write control over the target resource's delegation attribute/security descriptor; a controlled account with a usable SPN/key; an eligible KDC/target configuration; and a useful impersonated account.

The source workflow:

1. Obtain or create a controlled service/computer principal. Machine-account creation depends on quota and delegated permissions; it is not guaranteed for every domain user.
2. Record its SID and key material.
3. Save the target's original `msDS-AllowedToActOnBehalfOfOtherIdentity` value.
4. Add a descriptor/ACE authorizing the controlled principal without discarding legitimate entries.
5. Request the S4U ticket for the target service and verify access.
6. Restore the attribute and remove only the lab-created principal/artifacts.

Powermad `New-MachineAccount`, PowerView object queries/updates, Impacket delegation tools and Rubeus S4U are alternatives. The original raw security-descriptor example overwrites the whole attribute; preserve existing access instead. A machine account's trailing `$` is part of its account name and requires careful shell quoting.

## Delegation tickets and context

Rubeus `tgtdeleg` or Kekeo `tgt::deleg` may obtain a usable TGT in an eligible **current** authentication context. They do not produce a TGT for an arbitrary account whose secret/context is unavailable. A ticket request, ticket import and remote service authorization are separate verification steps.
