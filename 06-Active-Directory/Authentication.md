# AD authentication and ticket handling

[Home](../README.md) · [AD workflow](Workflow.md) · [Roasting](Roasting.md) · [Lateral movement](Lateral-Movement.md)

## Objects and boundaries

A domain stores principals and policy; DCs provide directory/authentication services. A forest can contain multiple domains. OUs organize objects and policy scope; groups grant membership-based access. A SID identifies a principal, while its final RID distinguishes it within the domain. The PDC emulator is one FSMO role holder, not a universally authoritative "main DC" for every operation.

An SPN identifies a service instance, for example `cifs/server.example.test`, `HTTP/web.example.test` or `MSSQLSvc/db.example.test:1433`. The account ending in `$` and the host portion of an SPN are related concepts, not interchangeable strings. Use the actual registered SPN and DNS hostname.

## NT hashes and NTLM

An NT hash is derived from the UTF-16LE password and has no per-account salt. The NTLM network protocol uses challenge-response authentication; the client does **not** send that NT hash directly across the network.

| Material | Meaning | Typical use |
|---|---|---|
| Plain password | Original secret | Authenticate to supported services; derive keys where allowed. |
| NT hash | Password-derived key material | PTH to NTLM-capable services, or RC4-based Kerberos where accepted. |
| LM hash | Legacy password representation | Offline recovery; not normally stored on modern defaults. |
| NetNTLMv1/v2 response | Response tied to challenge and identity | Offline cracking or a live relay chain, not ordinary bare-hash PTH. |
| Cached domain logon verifier | Local cached verifier, often DCC2 | Offline cracking; not the NT hash used for generic PTH. |

NTLM's exchange is negotiation → server challenge → client authentication response → validation by the server or DC. Relay forwards a live exchange; cracking tries password candidates against captured data. See [relay prerequisites](Relay-Coercion.md).

## Kerberos flow

1. **AS-REQ:** client requests initial authentication, normally including pre-authentication data encrypted with its long-term key.
2. **AS-REP:** KDC returns a TGT encrypted for `krbtgt` plus client-readable session-key material protected using the client's key.
3. **TGS-REQ:** client presents the TGT and authenticator to request a ticket for an SPN.
4. **TGS-REP:** KDC returns a service ticket encrypted for the service account and the client/service session-key material.
5. **AP-REQ:** client presents that service ticket and authenticator to the application.
6. **AP-REP:** may provide mutual authentication using the session key.

The service's long-term key and the client/service session key are different. A TGT is not directly a service ticket. Kerberos is primarily symmetric-key authentication; calling a service account key a generic "private key" obscures this. Address checks depend on ticket/configuration; IP binding is not universally enforced as the original simplified flow suggests.

## DNS, realm and time

Use FQDNs for Kerberos-aware clients. Record the DC IP separately for transport overrides. Confirm the realm mapping in `/etc/krb5.conf`, name resolution, time zone interpretation and actual clock offset. Correct the specific time mismatch; disabling every clock-sync service or running an unbounded `ntpdate` loop is unnecessary.

`ntpdate -q <DC_IP>` can query a reachable NTP server; `klist` and Kerberos error messages help identify ticket/realm issues. A virtual machine's host time synchronization can undo a manual adjustment. Process-local `faketime` behavior is tool-dependent and does not change a ticket's validity rules.

## Request, inspect and convert tickets

**Kali / Bash:**

```bash
impacket-getTGT '<DOMAIN>/<USER>' -dc-ip <DC_IP>
export KRB5CCNAME="$(realpath <USER>.ccache)"
klist
impacket-ticketConverter ticket.kirbi ticket.ccache
impacket-ticketConverter ticket.ccache ticket.kirbi
```

Quote literal passwords if supplied on a command line, or use a supported prompt. `KRB5CCNAME='realpath file.ccache'` stores a literal string and is incorrect. A Base64-encoded `.kirbi` must be decoded before format conversion. `kdestroy` removes tickets from the selected cache; preserve any needed artifact first.

`kinit <USER>@<REALM>` is a Kerberos client alternative. `klist -k <KEYTAB>` lists principals in a keytab where readable. Keytabs contain secret key material; `.k5login` is a login-authorization file, not a ticket. Running a KDC or using `kadmin` is not required to use a client ticket.

On Linux, inspect `default_ccache_name` in `/etc/krb5.conf` and the actual selected cache with `klist`. Cache backends can be FILE, DIR, KEYRING, KCM or another provider; do not assume every ticket is `/tmp/krb5cc_<UID>`. Copy a readable FILE cache only when its permissions/context allow it. The supplied `tickey` reference targets eligible keyring/process contexts and is not a generic read of every user's tickets.

Rubeus can export Base64 ticket material; preserve it without console wrapping and decode to bytes before conversion. Deriving an NT hash from a known password requires MD4 over UTF-16LE; Python builds may not expose MD4 through `hashlib`, so the source's Python 2 one-liner is not a portable Python 3 command.

## Hashes, keys and tickets are not universal access

PTH needs a service accepting NTLM and the account's authorization there. Overpass-the-hash/pass-the-key obtains Kerberos tickets from accepted key material. PTT reuses an existing valid ticket. A service ticket is scoped to its service/host and authorization data; a TGT can request additional tickets within its allowed context. Neither grants Domain Admin simply because it was imported.
