# Additional AD and Windows attack surfaces

[Home](../README.md) · [AD workflow](Workflow.md) · [ACLs](ACLs.md) · [Trusts and persistence](Trusts-Persistence.md)

The source collection contains many link-only leads. The table keeps their distinct ideas without presenting unread external articles as local instructions.

| Surface | What to investigate | Required evidence before proceeding |
|---|---|---|
| SCCM / Configuration Manager | Deployment roles, client management and distribution paths | A controlled role/account/path with an action that managed clients consume. |
| WSUS | Update metadata, transport and execution/signature policy | A matching vulnerable configuration and control over the relevant update flow. |
| Exchange / PrivExchange | Application authentication and delegated directory rights | Version/configuration, authenticated context and usable relay/ACL destination. |
| SharePoint | Workflow, BDC/type conversion or deserialization endpoints | Exact component/version and accepted attacker-controlled serialization path. |
| SQL Server Reporting Services | Serialized inputs and application keys | Matching endpoint/version and required authentication/key material. |
| AD-integrated DNS | Who can create/change which record objects | Effective record/zone ACLs and a client that uses the influenced name. |
| LAPS / Windows LAPS | Delegated password readers and stored/encrypted password data | Correct attribute, decrypt/read rights and target account mapping. |
| BitLocker recovery objects | Recovery-key attributes | Actual read permissions; directory membership alone is insufficient. |
| GPO/OU links | Policy-edit and link-control rights | Relevant computer/user scope, filtering and refresh/execution consumer. |
| Enterprise administrative tiers | Which identities administer which tiers | Verified trust/ACL/session paths rather than a topology label. |

Named tools in these resource lists include PowerSCCM, WSUSpendu, MailSniper, GPO-Abuse, Grouper, ADACLScanner, zBang, GoFetch, DeathStar and ANGRYPUPPY. Some automate whole attack plans; retain them as supplementary tool references and assess their behavior and current exam permission separately.

The collection's MS14-068, Exchange, SharePoint and other named vulnerability leads are retained in the exploit inventory. Source titles are not proof of exploitability or permission to apply them to a target.
