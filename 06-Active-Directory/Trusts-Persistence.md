# Replication, forged tickets, trusts and persistence

[Home](../README.md) · [AD workflow](Workflow.md) · [Authentication](Authentication.md) · [Credentials](Credentials.md) · [Delegation](Delegation.md)

These extended techniques retain useful material from the supplied collections. Many require control beyond the first privilege objective and introduce changes that need explicit tracking in a lab.

## DCSync

**Prerequisite:** effective replication rights over the relevant domain naming context, typically `DS-Replication-Get-Changes` and `DS-Replication-Get-Changes-All`; additional filtered-set rights can matter for specific data. Membership in an administrative group is one way to obtain rights, not the only one.

DCSync uses the replication protocol to request data. It does not require installing a real new DC or reading the DC's LSASS process.

**Mimikatz:**

```text
lsadump::dcsync /domain:<DOMAIN> /user:<DOMAIN>\<USER>
```

**Kali / Bash, selected account:**

```bash
impacket-secretsdump -just-dc-user <USER> '<DOMAIN>/<CONTROLLED_USER>@<DC_HOSTNAME>'
```

Use the tool's Kerberos/hash options when holding those formats. A generic remote `secretsdump` can attempt additional techniques; choose the intended method and output scope. Save the ACL evidence and successful replication result, then use recovered material according to its actual account rights.

## Golden and silver tickets

| Ticket | Key required | Intended scope |
|---|---|---|
| Golden | Domain `krbtgt` key | Forged TGT in that domain, subject to validation/policy. |
| Silver | Key of the account protecting the selected service | Forged service ticket for the matching SPN/host/key context. |
| Inter-realm trust | Matching directional trust key | Referral/trust authentication subject to trust filtering and authorization. |

Both require correct realm, domain SID, identity/RID and compatible encryption/validation. The service account SID is not a substitute for the domain SID. Do not put a service key into the `/krbtgt` field and call the result a silver ticket.

**Mimikatz source-style examples:**

```text
kerberos::golden /user:<USER> /domain:<DOMAIN> /sid:<DOMAIN_SID> /krbtgt:<KRBTGT_NT_HASH> /id:<RID> /ticket:golden.kirbi
kerberos::golden /user:<USER> /domain:<DOMAIN> /sid:<DOMAIN_SID> /target:<HOST_FQDN> /service:<SERVICE> /rc4:<SERVICE_NT_HASH> /ticket:silver.kirbi
kerberos::ptt <TICKET.kirbi>
```

Impacket `ticketer` is another generator in the collection. Select supported key types/options from the installed version. Ticket import is not an OS-wide administrator-token change; verify a matching service request. Modern account/ticket validation can invalidate a historical example even when the basic keys are known.

## Trust direction, SID history and forest boundaries

A **diamond ticket** modifies a legitimately obtained TGT and re-protects the modified ticket using the required `krbtgt` key material. Merely requesting a ticket before generating a golden ticket does not make it a diamond ticket. Tool, encryption and PAC validation requirements remain relevant.

Enumerate trust direction, type, transitivity, SID filtering, selective authentication and delegation policy. A bidirectional trust is not a grant of Domain Admin in the other forest.

Historical SID-history/ExtraSids examples use a controlled child domain's key and a privileged SID from the forest root. Applicability depends on the actual trust/filtering/validation configuration; do not assume filtering is disabled. Root-domain RID 519 identifies Enterprise Admins in its appropriate domain context, not every domain's local administrators.

An unconstrained-delegation + printer-coercion chain across forests needs an eligible forwarded TGT and usable cross-trust delegation settings. A simple trust listing or one coerced network connection does not prove that chain.

Source trust flags include NON_TRANSITIVE (1), UPLEVEL_ONLY (2), QUARANTINED_DOMAIN (4), FOREST_TRANSITIVE (8), CROSS_ORGANIZATION (16), WITHIN_FOREST (32), TREAT_AS_EXTERNAL (64) and PIM_TRUST (1024). Flags are a bit field; a numeric value such as 72 alone does not prove a SID-filtering bypass. Determine direction, object type, filtering and effective authorization separately. The map's “Saphire” means **Sapphire ticket**, retained as a source-only forged/PAC-ticket lead without a complete local workflow.

## SQL links across trust boundaries

PowerUpSQL can enumerate instances and connections (`Get-SQLInstanceDomain`, `Get-SQLConnectionTestThreaded`, `Get-SQLServerInfo`) and inspect/crawl linked servers (`Get-SQLServerLink`, `Get-SQLServerLinkCrawl`). Effective linked-server login mappings may cross a directory boundary independently of a user's ordinary host privileges.

Verify the remote SQL identity, RPC Out and execution permissions at each hop. See [database access](../01-Enumeration/Databases.md#microsoft-sql-server--tcp-1433). Do not enable links/features before establishing permission and recording their original state.

## Other persistence mechanisms

| Mechanism | Existing control needed | Important limit/change |
|---|---|---|
| Skeleton Key | Powerful access to a compatible DC authentication process | Memory patch, platform/protection dependent; not a universal domain password. |
| DSRM account abuse | Access to DSRM credentials and relevant DC policy | DSRM is separate recovery context; changing `DsrmAdminLogonBehavior` is a privileged registry change. |
| Custom SSP / `misc::memssp` | Ability to register/load/inject an SSP | Changes authentication handling; requires compatible process protections and cleanup. |
| Golden/silver tickets | Corresponding keys and valid construction | Key scope, ticket validity and service authorization still apply. |
| Shadow-copy credential access | Snapshot/backup/file rights | Credential access technique, not persistence by itself. |

The source `DsrmAdminLogonBehaviour` spelling is incorrect for the registry value; the intended value name uses **Behavior**. Likewise, the RDP source's delete command truncates `fSingleSessionPerUser`; restoration must use the exact value actually changed.

## DCShadow

DCSync reads replication data; DCShadow attempts to **write** directory changes through replication behavior after registering suitable server/replication metadata. It needs powerful effective permissions and a compatible environment. Registration, changed object attributes and cleanup all belong in the evidence record. The source claim that it generates no logs is not a reliable guarantee.

## AdminSDHolder and SDProp

AdminSDHolder supplies a security-descriptor template for protected accounts/groups. SDProp periodically reapplies protected ACLs, so a deliberately inserted ACE on AdminSDHolder can propagate broader control. The source describes an hourly schedule; verify the actual environment rather than treating that timing as guaranteed.

**Required evidence:** control of the relevant descriptor, an affected protected object and the propagated effective ACE. `adminCount=1` is a useful lead but does not alone prove an account's current protected-group membership or exact ACL state. Removing group membership may not automatically restore prior inheritance.

The source PowerView `Add-ObjectAcl`/`Add-DomainObjectAcl` examples grant FullControl, reset-password, membership or replication rights. Preserve the original descriptor and check the loaded version's parameters. `Set-ADACL` and `Invoke-SDPropagator` in those notes are helper scripts, not standard commands supplied by every ActiveDirectory module.

## Delegated management descriptors

WMI namespaces, PowerShell remoting endpoints and Remote Registry objects have their own security descriptors. An administrator who can change them can delegate future access to another principal. Source helper families include `Set-RemoteWMI`, `Set-RemotePSRemoting` and DAMP's `Add-RemoteRegBackdoor`.

Record which object/namespace changes, the granted operations and exact original DACL. Retrieving machine/local/cached credentials through a delegated registry path still depends on the effective operations and artifacts exposed. Revoke the specific ACE and verify restoration; a generic tool `-Remove` option is not proof that every change was undone.

## Other source leads

- **SeEnableDelegationPrivilege:** a user right affecting delegation configuration; it must be combined with applicable object permissions and policy. It is not the same as already holding a delegated TGT.
- **RID hijacking:** modification of local account/security data to alter effective RID behavior requires powerful local control; preserve it as a separate local account technique rather than a universal domain escalation.
- **Privileged built-in groups:** Account/Server/Print/Backup Operators have bounded abilities that vary by DC/host and policy. A stopped legacy service, readable backup or driver right is only useful when all execution/data prerequisites match.
- **SCCM and WSUS:** delegated deployment/update control can affect managed clients; inspect roles, management traffic, signing and distribution scope. The supplied material contains resource links, not a complete exploitation recipe. See [supplementary surfaces](Supplementary-Surfaces.md).

## Directory vulnerabilities as separate leads

Zerologon, noPac, PrintNightmare, Exchange and SharePoint issues in the collection require matching versions/configuration. Zerologon password-reset PoCs can disrupt a DC's machine-account state and need a recovery plan; "all Windows Servers" is too broad. Preserve those identifiers in the exploit reference rather than treating them as routine steps for every domain.
