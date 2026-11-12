# AD object permissions and group paths

[Home](../README.md) · [Enumeration](Enumeration.md) · [Roasting](Roasting.md) · [Delegation](Delegation.md) · [Certificates](Certificates.md)

## Read an edge precisely

Identify the controlled principal, all effective groups, target object, access mask, property/extended-right GUID, inheritance and applicable deny entries. A permission name without its object type/scope is insufficient.

**PowerView / PowerShell:**

```powershell
Get-DomainObjectAcl -Identity '<TARGET_OBJECT>' -ResolveGUIDs
Find-InterestingDomainAcl -ResolveGUIDs
Convert-NameToSid '<DOMAIN>\<USER>'
```

Resolve GUIDs to determine whether an ACE applies to a specific attribute, extended right, child object class or the whole object. Match the controlled SID and effective group SIDs; filtering only the username can miss inherited/group-based control.

## Permission-to-action map

| Right or control | Candidate action | What must still be checked |
|---|---|---|
| GenericAll over a user | Reset password, modify permitted properties, targeted roast | Object restrictions and effect on the account. |
| GenericWrite over a user | Modify eligible attributes such as SPN | Does not by itself mean every extended right/password reset is granted. |
| ForceChangePassword | Set a new password without the old one | Account effect, policy and potential DPAPI/EFS access consequences. |
| WriteDACL | Add an ACE granting a useful right | Owner/protection/inheritance, exact desired right and restoration. |
| WriteOwner | Change object ownership under allowed semantics | Ownership and ability to change DACL are separate checks. |
| Group membership control | Add a controlled principal | Rights on `member`/validated writes and actual group benefit. |
| GenericWrite/GenericAll on a computer | RBCD or other permitted property path | Controlled service identity and delegation/KDC requirements. |
| Replication extended rights on domain root | DCSync | Correct scope and all required replication permissions. |
| GPO control | Change policy consumed by linked objects | GPO edit rights, links, security/WMI filtering and refresh/trigger. |

`Self` is an access-right category; it does not universally allow any object to be added to any group. `AllExtendedRights` also depends on target class and effective ACL. Use the actual permitted operation.

## Modifying a user or group

Before a change, save the original membership, ACL or attribute. Use the loaded module's correct command/parameters and modify the smallest needed value. Confirm the new effective identity with a fresh authentication/token where required; an existing logon token may not immediately contain new group membership.

Password reset, SPN insertion, pre-authentication changes and shadow-credential registration are distinct actions with different side effects. Prefer a reversible path appropriate to the objective, and restore the exact original state afterward.

### Tool operations and property scope

The supplied bloodyAD command family maps `get` to reads, `set` to property/password changes, and `add`/`remove` to membership, UAC, computer, DNS or ACL operations. Use local help because authentication and argument syntax varies across versions.

Useful reads include `get writable --detail`, object attributes such as `ms-DS-MachineAccountQuota`, `userAccountControl`, `msDS-Behavior-Version`, `minPwdLength`, and relevant group members. A base domain DN must be the actual DN; repeating `DC=<DOMAIN>` for every component is not a valid universal placeholder expansion.

`msDS-ManagedPassword` contains a gMSA managed-password blob and needs parsing plus read permission. It is not the same attribute as legacy LAPS `ms-Mcs-AdmPwd`. A script-path update needs an executable/logon consumer and a reachable appropriate script location. Adding a DNS record needs record/zone control and a client that uses the name.

When modifying UAC, change only the intended bit and preserve other flags. Removing `ACCOUNTDISABLE` enables an account; it does not grant new privileges. Creating a computer requires delegated rights or applicable quota. Adding GenericAll requires existing ACL-control rights; a tool command does not manufacture permission.

## Chained access

A low-privilege user may control a group that controls another group or object. Recalculate the effective edge after each transition. Paths through local groups, GPOs and sessions require host/network verification in addition to directory data. ACLPwn/aclpwn.py and graph tools in the collection automate parts of this reasoning; their proposed chain remains subject to verification.

## Special groups

DNSAdmins may expose DNS server configuration, including a server-level plugin DLL on compatible systems, but needs appropriate configuration and a load/restart trigger. Backup Operators relies on usable backup privileges; see [credential artifacts](Credentials.md). Account Operators and application-specific groups may have broad but bounded object rights; inspect actual ACLs instead of treating the group name as Domain Admin.

Exchange delegation, AD-integrated DNS updates and certificate-template control are separate permission surfaces. Use the matching service/attribute evidence before selecting a technique.
