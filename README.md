# OSCP Handbook

A linked, task-oriented consolidation of the supplied local notes. Repeated techniques share a canonical chapter; personal introductions, badges, store links and duplicated resource lists have been removed.

Start with the [target workflow](00-Start/Workflow.md), then follow the page for the service or behavior you observe. Commands are examples: replace every `<PLACEHOLDER>` and run them in the stated shell. Explanations and examples are consolidated from local files; no websites have been consulted.

## Find the next action

| Observation | Open |
|---|---|
| New target or no useful lead | [Workflow](00-Start/Workflow.md), [scanning](01-Enumeration/Scanning.md) |
| Hostname, domain or DNS service | [DNS](01-Enumeration/DNS.md) |
| SMB shares or Windows RPC | [SMB and RPC](01-Enumeration/SMB-RPC.md) |
| FTP, SSH or NFS | [File services](01-Enumeration/File-Services.md) |
| Mail or SNMP | [Mail and SNMP](01-Enumeration/Mail-SNMP.md) |
| Website or API | [Web enumeration](02-Web/Enumeration.md) |
| SQL errors or database-backed input | [SQL injection](02-Web/SQL-Injection.md) |
| File parameter or upload | [File attacks](02-Web/File-Attacks.md) |
| Input passed to a system command | [Command injection](02-Web/Command-Injection.md) |
| Template expressions | [Template injection](02-Web/Template-Injection.md) |
| Direct database access | [Databases](01-Enumeration/Databases.md) |
| SSRF, XML or serialized data | [Server-side requests](02-Web/Server-Side-Requests.md) |
| Browser script execution | [Browser attacks](02-Web/Browser-Attacks.md) |
| Need a file or command channel | [Transfers](03-Access/File-Transfers.md), [shells](03-Access/Shells.md) |
| Password or hash | [Passwords](03-Access/Passwords.md) |
| Framework or payload reference | [Metasploit](03-Access/Metasploit.md) |
| Internal network or local-only service | [Tunnels and pivoting](07-Pivoting/Tunnels.md) |
| Linux foothold | [Local enumeration](04-Linux/Enumeration.md), [privilege escalation](04-Linux/Privilege-Escalation.md) |
| Domain, AD credential or directory path | [AD workflow](06-Active-Directory/Workflow.md) |
| Windows foothold | [Local enumeration](05-Windows/Enumeration.md), [services and tasks](05-Windows/Services-Tasks.md), [privileges and credentials](05-Windows/Privileges-Credentials.md) |

## Reading conventions

- **Kali / Bash** means a terminal on the attacking machine. **Target / PowerShell**, **Target / CMD** and named tool consoles identify other execution contexts.
- `<IP>` is a target IP; `<LHOST>` is an address the target can reach; `<DOMAIN>` is a DNS domain; `<DC_IP>` is a domain controller address. `<USER>` is a username, never an instruction to overwrite the shell's `USER` variable.
- A working password proves authentication. It does not prove permission to read a share, execute remotely or administer a machine.
- Preserve the exact request, command, output, identity and relevant target when recording a finding. Record negative results too.
- Source examples span different software generations. Version-dependent techniques state their prerequisites. Commands have not been run against targets during this consolidation.

## Exam use

The supplied collections contain conflicting historical exam guidance. They do not establish the rules for your scheduled exam. Use your current official exam instructions for tool permissions, objectives, proof, reporting and submission. Technical reference material is not a statement that a tool is permitted.

## Import and validation

Import `OSCP-Handbook-Notion.zip` using Notion’s Markdown import, then open this home page and follow the relative chapter links. Keep the original folders separately for source code/binaries. Duplicate the [machine worksheet](09-Reporting/Machine-Template.md) into private notes for each target.

[Review notes](08-Reference/Review-Notes.md) explain source coverage, corrections and validation limits. Markdown links/anchors, formatting and source hashes were checked locally. Notion UI import and offensive commands were not tested. External technical links are retained references, not newly researched content.

## Full contents

### Start and preparation

- [Target workflow](00-Start/Workflow.md)

### Service enumeration

- [Auxiliary services and protocol checks](01-Enumeration/Auxiliary-Services.md)
- [DNS and hostname discovery](01-Enumeration/DNS.md)
- [Direct database access](01-Enumeration/Databases.md)
- [FTP, SSH and NFS](01-Enumeration/File-Services.md)
- [Mail services and SNMP](01-Enumeration/Mail-SNMP.md)
- [SMB, NetBIOS and Windows RPC](01-Enumeration/SMB-RPC.md)
- [Host discovery and port scanning](01-Enumeration/Scanning.md)
- [Service and port map](01-Enumeration/Service-Map.md)

### Web applications

- [XSS and browser-side workflows](02-Web/Browser-Attacks.md)
- [OS command injection](02-Web/Command-Injection.md)
- [Web enumeration](02-Web/Enumeration.md)
- [Traversal, inclusion and file upload](02-Web/File-Attacks.md)
- [SQL and NoSQL injection](02-Web/SQL-Injection.md)
- [SSRF, XXE and unsafe deserialization](02-Web/Server-Side-Requests.md)
- [Server-side and client-side template injection](02-Web/Template-Injection.md)

### Access and credentials

- [Client-side files and delivery prerequisites](03-Access/Client-Side-Delivery.md)
- [File transfers](03-Access/File-Transfers.md)
- [Metasploit and payload generation](03-Access/Metasploit.md)
- [Passwords, hashes and wordlists](03-Access/Passwords.md)
- [Shell delivery and stabilization](03-Access/Shells.md)

### Linux

- [Linux local enumeration](04-Linux/Enumeration.md)
- [Linux exploitation patterns](04-Linux/Exploitation-Patterns.md)
- [Linux privilege escalation](04-Linux/Privilege-Escalation.md)

### Windows

- [Windows local enumeration](05-Windows/Enumeration.md)
- [Windows privileges, credentials and other elevation paths](05-Windows/Privileges-Credentials.md)
- [Windows services, DLLs and scheduled tasks](05-Windows/Services-Tasks.md)

### Active Directory

- [AD object permissions and group paths](06-Active-Directory/ACLs.md)
- [AD authentication and ticket handling](06-Active-Directory/Authentication.md)
- [Active Directory Certificate Services](06-Active-Directory/Certificates.md)
- [Windows and domain credential material](06-Active-Directory/Credentials.md)
- [Kerberos delegation](06-Active-Directory/Delegation.md)
- [AD objects, LDAP and access enumeration](06-Active-Directory/Enumeration.md)
- [BloodHound and relationship analysis](06-Active-Directory/Graph-Analysis.md)
- [Remote access and lateral movement](06-Active-Directory/Lateral-Movement.md)
- [NTLM capture, coercion and relay](06-Active-Directory/Relay-Coercion.md)
- [Kerberos roasting and account discovery](06-Active-Directory/Roasting.md)
- [Shadow credentials and certificate authentication](06-Active-Directory/Shadow-Credentials.md)
- [Additional AD and Windows attack surfaces](06-Active-Directory/Supplementary-Surfaces.md)
- [Replication, forged tickets, trusts and persistence](06-Active-Directory/Trusts-Persistence.md)
- [Active Directory workflow](06-Active-Directory/Workflow.md)

### Pivoting

- [Port forwarding and pivoting](07-Pivoting/Tunnels.md)

### Reference

- [Buffer overflows and exploit adaptation](08-Reference/Buffer-Overflows.md)
- [Bundled artifact reference](08-Reference/Bundled-Artifacts.md)
- [Command-and-control framework concepts](08-Reference/C2-Concepts.md)
- [Endpoint detection and execution controls](08-Reference/Endpoint-Controls.md)
- [Exploit and CVE registry](08-Reference/Exploit-Registry.md)
- [Exploit selection, adaptation and source corrections](08-Reference/Exploit-Selection.md)
- [File and configuration locations](08-Reference/File-Locations.md)
- [Managed-code inspection and serialization leads](08-Reference/Managed-Code.md)
- [Nmap option reference](08-Reference/Nmap-Options.md)
- [Remote administration tool reference](08-Reference/Remote-Tool-Commands.md)
- [Consolidation and review notes](08-Reference/Review-Notes.md)
- [Local toolkit and troubleshooting](08-Reference/Toolkit.md)
- [Tool catalogue](08-Reference/Tools.md)

### Reporting worksheets

- [Evidence, reporting and exam preparation](09-Reporting/Evidence.md)
- [Credential, tool and progress inventories](09-Reporting/Inventories.md)
- [Machine worksheet](09-Reporting/Machine-Template.md)
- [Assessment report template](09-Reporting/Report-Template.md)

### Practice

- [Practice and readiness](10-Practice/Readiness.md)
