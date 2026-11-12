# Active Directory Certificate Services

[Home](../README.md) · [AD workflow](Workflow.md) · [ACLs](ACLs.md) · [Authentication](Authentication.md) · [Relay](Relay-Coercion.md)

## Establish the certificate path

Identify the CA, published templates, enrollment rights, authentication EKUs, subject/SAN control, approval/signature requirements, template/CA ACLs and account-mapping behavior. A certificate is useful only with its corresponding private key and an authentication service that accepts the resulting identity mapping.

Source enumeration tools include Certify, Certipy, PSPKIAudit and certificate-template administration tooling. Example **Windows / Certify console arguments**:

```text
Certify.exe find /vulnerable
```

Check the installed release's help. A tool's "vulnerable" label is a lead that must be verified against CA policy and actual account mapping.

## Subject-supply template path

The supplied detailed example describes an ESC1-style combination:

- A controlled principal can enroll in a published template.
- The template/CA permits a requester-controlled subject or alternative identity.
- The certificate is valid for an authentication purpose.
- Issuance is not blocked by required approval or unavailable authorized signatures.
- The issued identity can map to the intended account under the domain's rules.

`ENROLLEE_SUPPLIES_SUBJECT`, Client Authentication EKU and zero required signatures are useful checks but do not prove the whole path alone. A `Protected Users` membership or a guessed UPN is not a complete explanation for success/failure; inspect certificate contents and mapping errors.

Request only after selecting the actual CA/template and saving the request parameters. The original Certify example adds `/machine` indiscriminately; enrollment context must match the controlled principal and template requirements.

## PEM, key and PFX conversion

**Kali / Bash:**

```bash
openssl x509 -in cert.pem -noout -text
openssl pkcs12 -export -in cert.pem -inkey cert.key -out cert.pfx
```

A certificate file without its private key is not sufficient for client authentication. Keep the key and export password with the credential record. There is no need to concatenate key and certificate merely to use OpenSSL when separate `-in`/`-inkey` arguments are available.

## Authenticate and verify

The source Rubeus command contains the typo `asktht`; the relevant command is `asktgt`:

```text
Rubeus.exe asktgt /user:<USER> /domain:<DOMAIN> /dc:<DC_HOSTNAME> /certificate:<PATH_TO_PFX> /ptt
```

Supply a PFX password where needed using the tool's supported option. PKINIT must be supported and the certificate/account mapping accepted. After obtaining a ticket, inspect it and test a specific authorized service; a successful certificate request is not yet a usable TGT or domain administrator session.

## Certificate path comparison

These labels and examples come from the local source collection. They span different tool and Windows generations; they are not a verified snapshot of current CA/DC behavior.

| Source label | Underlying control/configuration | Evidence to establish |
|---|---|---|
| ESC1 | Requester-supplied subject/alternative identity on an authentication-capable template | Enrollment, publication, issuance and accepted account mapping. |
| ESC2 | Any Purpose/no restrictive EKU behavior | What the issued certificate can actually be used for; enrollment-agent use needs additional conditions. |
| ESC3 | Enrollment Agent certificate | Agent EKU, enrollment-agent restrictions and a target template accepting on-behalf-of enrollment. |
| ESC4 | Writable template object | Exact template ACL, controllable attributes, publication and a resulting usable enrollment path. |
| ESC5 | Vulnerable access control on other PKI objects | Identify the specific CA/PKI object, effective write right and downstream issuance/trust consumer; not every PKI ACE is a template edit. |
| ESC6 | CA accepts SAN request attributes | CA flag plus template/issuance/mapping behavior; the flag alone does not guarantee impersonation. |
| ESC7 | CA management/issuance permissions | Distinguish ManageCA from ManageCertificates/officer roles and identify the actual privileged operation. |
| ESC8 | Relay to HTTP certificate enrollment | NTLM-capable endpoint without effective relay protection, a live authenticating client and an eligible template. |
| ESC9 | Template omits the security extension | Template flag plus weak/eligible identity mapping and control of the required account attributes/context. |
| ESC10 | Weak certificate mapping | Actual KDC/Schannel mapping policy and a controllable identity field accepted by that policy. |
| ESC11 | Relay to RPC enrollment without required request encryption | Reachable enrollment interface, enforcement setting and relayed identity's enrollment rights. |
| ESC12 | Specific CA/HSM key-access configuration | Actual access to signing material or a usable signing operation; a generic `forge` command does not establish this. |
| ESC13 | Issuance policy linked to a privileged universal group | Policy OID/group link, eligible template and the resulting authentication membership behavior. |
| ESC14 | Weak explicit mapping (`altSecurityIdentities`) | Exact mapping type/value and control over a field that can produce a matching certificate. |
| ESC15 / CVE-2024-49019 | V1-template application-policy injection in a vulnerable CA | Matching patch/configuration and a certificate whose injected policy is accepted by the intended consumer. |
| ESC16 | CA globally disables the security extension | CA configuration plus account-mapping conditions; not equivalent to all certificates being universally usable. |
| ESC17 (source label) | Server-authentication certificate combined with WSUS/DNS influence | Eligible certificate name/key, affected update client behavior and control of the network/name/update path. |

The collection repeats the same command sequence for ESC2 and ESC3. Their prerequisites differ; do not collapse Any Purpose and Enrollment Agent into an automatic two-step compromise. Likewise, exporting a CA PFX is evidence of signing-key access, not a method that automatically acquires an HSM's private key.

## Certipy request workflow

**Kali / Bash, source-style command names; check the installed `certipy` or `certipy-ad` help:**

```bash
certipy-ad find -u '<USER>@<DOMAIN>' -p '<PASSWORD>' -dc-ip <DC_IP> -vulnerable -stdout
certipy-ad req -u '<USER>@<DOMAIN>' -p '<PASSWORD>' -dc-ip <DC_IP> -target <CA_HOST_FQDN> -ca '<CA_NAME>' -template '<TEMPLATE>'
certipy-ad auth -pfx certificate.pfx -dc-ip <DC_IP>
```

The CA's configuration name, server FQDN and DC address are separate values. Add subject/UPN/SID/DNS options only when the verified path requires them. A filename such as `Administrator.pfx` does not prove the identity encoded in its certificate.

For a verified enrollment-agent path, the second request may use `-on-behalf-of '<DOMAIN>\<TARGET_USER>' -pfx agent.pfx` with a compatible target template. For a template-control path, save the original configuration/ACL before changing it and restore that exact configuration afterward. Template creation also needs directory-create and CA-publication rights; ADCSTemplate or a framework helper does not bypass them.

## Relay, mapping and issued-request handling

HTTP enrollment relay and RPC enrollment relay have different destinations/protection checks. See [relay](Relay-Coercion.md). Record the actual authenticated machine/user and requested template; selecting a DC certificate template for a user account does not guarantee enrollment.

CA-management examples add an officer, publish a template, issue a pending/denied request and retrieve it. Each action requires its own permission and request ID. Preserve the generated private key; retrieving an issued certificate without its matching key may be unusable.

Weak-mapping paths may change UPN, mail or `altSecurityIdentities`. Save the full original attributes. One source section says "revert UPN" but repeats the attack UPN in its command; restoring requires the **recorded original value**. Modern strong mapping and security-extension behavior can prevent the historical chain.

## Troubleshooting by actual error

| Error/result | Investigation |
|---|---|
| Identity not found in certificate | Inspect SAN/subject and select the explicit account/domain only if it matches an accepted mapping. |
| Certificate/account SID mismatch | Compare the issued SID extension and target SID; changing a CLI username cannot override the server's mapping rule. |
| `CERTSRV_E_KEY_LENGTH` | Use the template's minimum key length, such as a supported 4096-bit request where required. |
| PKINIT not supported | Determine whether another eligible certificate authentication protocol exists; possession alone does not add PKINIT support. |
| Request issued but auth denied | Inspect EKUs, trust chain, validity, mapping, key possession and target protocol. |
| Request timeout | Inspect request state/ID before retrying; the source's "run twice" workaround can create duplicate requests. |

Certipy's LDAP-shell/Schannel path and PassTheCert use certificate authentication differently from PKINIT. They still need a trusted chain, eligible mapping and useful directory rights. Do not weaken CA flags or restart Certificate Services simply to make a source example work.
