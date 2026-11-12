# Consolidation and review notes

[Home](../README.md) · [Tools](Tools.md) · [Exploit registry](Exploit-Registry.md) · [Bundled artifacts](Bundled-Artifacts.md)

## What was consolidated

The source inventory contains 392 files from the supplied folders, including 230 Markdown notes, two CSV exports, a 78-page PDF, a DOCX report template, diagrams/screenshots and bundled tools/PoCs. All 50 complete text bundles were read. The full source hash/disposition ledger is supplied separately as `OSCP-Handbook-Source-Audit.md`; original files remain unchanged.

Repeated scans, shell variants, SQL payloads and AD chains were reduced to shared workflows with links. Distinct prerequisites, tool families, protocol surfaces, source-labelled bugs and useful legacy variants were retained. Port-only and project-link-only entries are labelled as leads. The file-location reference deduplicates 339 supplied candidate paths. The CVE registry preserves 122 literal source identifiers plus name-based normalization of MySQL's NSE identifier and Heartbleed; it does not verify the identifiers' external validity.

## Removed from the reading flow

Personal introductions, exam-result emails, author profiles, sponsor/donation badges, social handles, affiliate shops, unrelated career/resource promotion, contribution policies and empty navigation stubs. Duplicate screenshots, decorative artwork and repeated transcripts were replaced with searchable technical explanations. Original legal notices remain with original artifacts; the handbook does not relicense their code.

Screenshots corroborate existing macro menus, WMI/DCOM descriptors, Process Monitor searches, C2 listener/launcher fields and authentication diagrams. Unique practice names were transcribed into the practice catalogue. The source Kerberos diagrams contain misleading labels: AS-REP versus TGT encryption, TGT versus service ticket, client versus service key and session-key use are clarified in [authentication](../06-Active-Directory/Authentication.md). A TGT is not a universal grant to every service.

The printer/LDAP settings screenshot illustrates a service holding a directory credential: changing a test destination can expose authentication only if the application permits the destination change and initiates a bind. Capture type/protocol and the actual identity matter. PXE/BCD and management-agent repository screenshots are configuration/credential leads with their own access prerequisites, not generic domain compromises.

## Material corrections

- NT hash, Net-NTLM response, cached domain credential, roast format, Kerberos key/ticket and certificate/private key are different artifacts.
- Shell syntax, execution host, SQL dialect, payload architecture and staged/stageless handler must match.
- File read/upload, authentication, remote execution, local privilege and domain replication are separate capabilities.
- Service-control rights, file/registry ACLs and restart/trigger rights are checked separately.
- AD object permissions and certificate issuance/mapping conditions are explicit; ACL/key-credential restoration preserves original values.
- Incorrect labels such as Bad YAML/deserialization, web.config/ASPX, Spring Cloud Function/Spring4Shell, ADWS/SCCM and Nmap ACK/Xmas were corrected.
- Destructive or contradictory source PoCs are identified in [exploit selection](Exploit-Selection.md), including the Zerologon password-reset “tester” and the source-labelled Fragnesia validator.
- Conflicting historical scoring, revert, bonus and tool-policy claims are kept out of operational guidance; use the actual exam instructions.

## Validation boundaries

Local checks cover chapter navigation/anchors, files, code-fence/table structure, duplicate blocks, unwanted imported markup and unchanged original hashes. Commands and artifact behavior were reviewed statically. No target scans, password tests, payload execution, exploitation, bundled installers or binaries were run. Bundled implementation libraries were triaged by interfaces and behavior; this is not a full source-code security audit. External references and current policy were not researched, as requested. Notion's UI import was not exercised, so inspect the first imported page and links before relying on the notebook.
