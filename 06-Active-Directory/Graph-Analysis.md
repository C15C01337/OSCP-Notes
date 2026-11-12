# BloodHound and relationship analysis

[Home](../README.md) · [Workflow](Workflow.md) · [Enumeration](Enumeration.md) · [ACLs](ACLs.md)

## Collect a compatible dataset

BloodHound visualizes relationships collected by SharpHound, BloodHound.py or a compatible collector. Legacy and Community Edition deployments can use different schemas and ingestion methods. Match collector and analyzer versions; a historical Neo4j/Electron setup is not the configuration for every release.

**Kali / Bash, source-style Python collection:**

```bash
bloodhound-python -u '<USER>' -p '<PASSWORD>' -d '<DOMAIN>' -ns <DC_IP> -c All
```

**Target / PowerShell, source-style Windows collection:**

```powershell
.\SharpHound.exe --CollectionMethods All --Domain <DOMAIN> --OutputDirectory .\collection
```

Use the executable's own help for credential, DC, collection-method and output options. Older PowerShell collectors expose `Invoke-BloodHound`; do not copy the EXE's double-dash arguments into a PowerShell function blindly. `All` can perform broad remote queries; select the collection needed for the question. Excluding DCs can leave relevant data missing.

## Analyze a concrete path

1. Mark the principals and hosts actually controlled.
2. Search paths to the specific objective or useful privilege.
3. Read each edge's meaning and prerequisites.
4. Verify group nesting, object ACLs and service/network access on the current target.
5. Record the successful identity transition and update the map.

A shortest path can contain stale sessions, incomplete local-group data or an edge that cannot be exercised from the current route. A graph path is a hypothesis until each transition is verified.

## Alternative analysis tools

Adalanche collects directory data and analyzes relationships. The source workflow is `adalanche collect activedirectory ...`, then `adalanche analyze` against the saved collection. Resolve CA trust or authentication errors by identifying the actual TLS/bind problem; switching to plaintext/basic authentication is not a neutral equivalent.

Other supplied tools include ACLight (privileged access discovery), ADRecon (directory reporting), ADMiner (graph reporting), RustHound and RustHound-CE (collectors), and adidnsdump (AD-integrated DNS data). Their output complements a verified manual record. Keep collection files private because they can contain usernames, network topology and sensitive relationships.

AzureHound is a separate cloud identity collector; SharpHound's on-premises AD dataset is not a substitute for cloud collection. Collection categories such as DC-only, sessions and GPO/local-group relationships have different coverage. Verify actual supported methods rather than assuming `All` in one release means the same as `All` in another. Custom Cypher queries also depend on the graph schema/version.

### ADRecon reporting

**Windows / PowerShell, source-style examples:**

```powershell
.\ADRecon.ps1 -Protocol LDAP -DomainController <DC_HOSTNAME> -Credential (Get-Credential)
.\ADRecon.ps1 -Protocol ADWS -DomainController <DC_HOSTNAME> -Credential (Get-Credential) -Collect Domain,DomainControllers
.\ADRecon.ps1 -GenExcel <SAVED_OUTPUT_DIRECTORY>
```

ADWS commonly uses TCP 9389; the imported port note incorrectly labels it as an SCCM port. ADRecon can save CSV/XML/JSON/HTML/Excel according to version/options. Excel report generation may need Excel and can be performed later against saved CSVs. `-Collect`, output directory/type, page size and thread count control scope/volume. LAPS passwords, BitLocker recovery keys, sensitive ACLs and service-logon information require the relevant rights; a standard-domain-user run does not guarantee all fields are readable.
