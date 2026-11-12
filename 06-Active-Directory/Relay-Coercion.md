# NTLM capture, coercion and relay

[Home](../README.md) · [Authentication](Authentication.md) · [SMB](../01-Enumeration/SMB-RPC.md) · [Certificates](Certificates.md)

## Separate the stages

1. **Cause authentication:** name-resolution poisoning, a controlled URL/UNC path, an application feature or a coercible service induces an outbound connection.
2. **Capture:** a listener receives a challenge-response suitable for offline password guessing.
3. **Relay:** a live authentication exchange is forwarded to a compatible destination.
4. **Use authorization:** the relayed identity must have rights to perform the selected action.

A saved NetNTLM response is not a bare NT hash. Capturing it and relaying a live exchange are different operations. An outbound connection alone does not prove credentials were sent.

## Capture surfaces in the notes

Responder and Inveigh implement listeners and name-resolution poisoning functions. Configure the intended interface and services. LLMNR, NBT-NS, mDNS, AD-integrated DNS updates and IPv6/DHCPv6 behavior are different causes of traffic, with different network/configuration prerequisites.

The collection also includes UNC paths from web applications, MSSQL `xp_dirtree`, `.url`/`.scf` icon paths, WebDAV/Windows library files and coercible Windows services. Match the consumer and protocol. A visible file icon can trigger a fetch only under compatible client behavior/policy; the old SCF example does not work uniformly on all Windows versions.

## Relay requirements

For SMB relay, required signing commonly prevents the intended unsigned session use. LDAP signing/channel binding, HTTP extended protection and other service protections may stop different relay paths. Do not generalize "SMB signing disabled" into a condition for every relay protocol.

Impacket `ntlmrelayx` takes the intended target/service and action. If using a separate poisoning/capture tool, disable conflicting listeners so the relay tool owns the required port. Record the identity and destination protocol returned by the relay, then verify its granted rights.

Administrative SMB command execution needs sufficient remote rights; LDAP changes and certificate enrollment have their own authorization checks. Self-relay/reflection restrictions and protocol compatibility can invalidate a captured connection's apparent opportunity.

## Coercion and delegation

PrinterBug/SpoolSample, PetitPotam, DFSCoerce and related names in the sources identify different RPC/service paths. They require reachable endpoints and compatible authentication/configuration. A forced NTLM authentication is not automatically a delegated Kerberos TGT; use [delegation](Delegation.md) only when the credential type and policy match.

mitm6 is an IPv6/DNS-positioning technique requiring relevant client/network behavior and a relay destination; it is not a universal way to compromise any IPv4 network. Keep captures, relay attempts and directory modifications distinguishable in the evidence record.

## LDAP relay actions

Source `ntlmrelayx` examples include an interactive LDAP shell, delegation changes, account ACL escalation and DNS updates. The relayed principal must hold the relevant directory rights. Options such as `--delegate-access`, `--escalate-user` or `--add-dns-record` select modifying actions rather than read-only enumeration.

`--remove-mic` relies on specific vulnerable behavior; it is not a general signing/channel-binding bypass. Suppressing dumps or automatic ACL changes limits selected actions but does not change the destination's authentication protections. The interactive shell's listening port and commands vary by tool version.

RBCD and shadow-credential shell helpers should preserve existing values. The source `clear_shadow_creds` command can delete legitimate key credentials and is not a prerequisite for adding one controlled entry. Record and remove only the added device/key entry.

## Source flag and WebDAV corrections

Responder `-A` is an analysis/passive mode in this tool family, not the active poisoning behavior implied by some transcripts. `-w` and `-W` are different version-specific switches; inspect installed help.

Additional coercion families include ShadowCoerce and MSEven/CheeseOunce. The source `-M coerce_plus -M PetitPotam -L` mixes module selection and listing. Inspect `nxc smb -M coerce_plus --options`, then select one supported module and its listener/method options for that build. WebDAV coercion also needs eligible WebClient behavior and the intended UNC network name/port form (such as `name@80/path`). A long encoded hostname copied from one lab is not a universal bypass.
