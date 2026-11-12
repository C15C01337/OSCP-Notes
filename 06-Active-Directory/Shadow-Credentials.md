# Shadow credentials and certificate authentication

[Home](../README.md) · [ACLs](ACLs.md) · [Certificates](Certificates.md) · [Authentication](Authentication.md) · [Credentials](Credentials.md)

## Required control

The technique adds a public-key credential to a target object's `msDS-KeyCredentialLink`, then authenticates as **that object** using the corresponding private key. It requires an effective right to write that attribute and an eligible authentication path (including a DC certificate/PKINIT support for the common flow). Control of an unrelated account or ordinary LDAP authentication does not grant the attribute write.

1. Resolve the target's DN/account name and record existing KeyCredential entries and DeviceIDs.
2. Read the object's ACL and verify the effective write path.
3. Add one identifiable credential; preserve the generated certificate, private key/PFX and any password.
4. Request a TGT for the **target account**, using the matching certificate/key.
5. Set `KRB5CCNAME` to the resulting cache, then test the authorized service as that target.
6. Remove only the DeviceID you added; verify the original entries remain.

PyWhisker, Whisker, Certipy's `shadow` functions and tooling exposed through BloodyAD are source tool families for this operation. Match the installed version's help for `list`, `add`, `remove` and authentication arguments. Avoid `clear` as cleanup: it removes legitimate entries too. Password resets, group membership changes and unrelated DACL edits in source transcripts are separate actions, not prerequisites for every shadow-credential chain.

## PKINITtools flow

**Kali / Bash, illustrative local tool paths:**

```bash
python3 gettgtpkinit.py -cert-pfx <TARGET.pfx> -pfx-pass '<PFX_PASSWORD>' \
  -dc-ip <DC_IP> '<REALM>/<TARGET_ACCOUNT>' target.ccache
export KRB5CCNAME="$PWD/target.ccache"
```

The tool returns an **AS-REP encryption key** as well as the cache. For supported U2U/PAC recovery, `getnthash.py` consumes that returned key with `-key`; it is not an arbitrary account AES key or the certificate's private key. Successful TGT issuance does not guarantee all subsequent recovery operations succeed.

## PassTheCert and Schannel

Where a certificate is eligible for LDAP Schannel but PKINIT is unavailable, PassTheCert can authenticate a certificate-bound LDAP connection. Exporting a certificate and key from a PFX preserves the same identity; it does not create permissions. A `modify_user` or password-reset operation still needs the corresponding directory right. Protect exported unencrypted private keys and remove temporary copies.

Do not globally weaken `/etc/ssl/openssl.cnf` to troubleshoot an unexplained reset. Identify certificate mapping, protocol, cipher and binding errors. If an old lab service requires a compatibility setting, scope a separate `OPENSSL_CONF` to that process and restore the original configuration rather than changing every client on the machine.
