# Passwords, hashes and wordlists

[Home](../README.md) · [SSH keys](../01-Enumeration/File-Services.md#ssh--tcp-22) · [SMB](../01-Enumeration/SMB-RPC.md) · [SQL injection](../02-Web/SQL-Injection.md)

## Identify the credential before using it

Record account, domain/local namespace, source, format, associated host, tested services and result. A digest's length is only a clue; use its origin and full format to identify it. `hashid`, `hash-identifier` and Hashcat's example hashes help classify candidates.

Distinguish an NT hash from a NetNTLM challenge-response and a Kerberos roast hash. They have different cracking formats and uses. A captured NetNTLM response is not a bare NT hash to pass directly to an arbitrary PTH option.

## Offline cracking

**Kali / Bash:**

```bash
hashcat -m <MODE> -a 0 hashes.txt <WORDLIST>
hashcat -m <MODE> hashes.txt <WORDLIST> -r /usr/share/hashcat/rules/best64.rule
hashcat -m <MODE> --show hashes.txt
john --wordlist=<WORDLIST> hashes.txt
john --show hashes.txt
```

| Format | Common Hashcat mode |
|---|---:|
| Raw MD5 | 0 |
| Raw SHA-1 | 100 |
| Raw MD4 | 900 |
| NT hash / NTLM | 1000 |
| LM | 3000 |
| Raw SHA-256 | 1400 |
| Raw SHA-512 | 1700 |
| MD5crypt (`$1$`) | 500 |
| Apache MD5/APR1 (`$apr1$`) | 1600 |
| Traditional DES crypt | 1500 |
| phpass, including compatible legacy WordPress hashes | 400 |
| sha512crypt (`$6$`) | 1800 |
| bcrypt | 3200 |
| NetNTLMv2 | 5600 |
| NetNTLMv1 | 5500 |
| Kerberos TGS-REP, RC4 | 13100 |
| KeePass supported extraction format | 13400 |
| Kerberos AS-REP, RC4 | 18200 |
| Kerberos TGS-REP, AES128 / AES256 | 19600 / 19700 |
| DCC2 / MSCache v2 | 2100 |

Modes must match the actual extracted format and encryption type. Mode 1800 is **sha512crypt**, not raw SHA-512. Do not use `--force` as a default fix for driver/runtime errors. Optimized kernels (`-O`) can impose password-length limits.

Converters mentioned in the notes include `ssh2john`, `keepass2john` and `zip2john`. Extract into a new file, keep the original artifact, then use the matching cracking format. KeePass databases may also require a key file; recovery is not always just cracking one password. `fcrackzip -u -D -p <WORDLIST> archive.zip` is an alternative ZIP-specific tool.

John's KeePass output can include a leading database label; remove **only that label** for a compatible Hashcat input and retain all hash fields. Mode 13400 is not automatically compatible with every KDBX version, including KDBX 4. `kpcli --kdb=<DATABASE.kdbx>` opens a database with a known password; kpcli is not the cracker.

## Rules and masks

Hashcat rule examples, one candidate transformation per line:

```text
:
$1
$!
c$1$!
```

`:` keeps the word; `$1` appends the character `1`; `$!` appends `!`; `c` capitalizes the first character and lowercases the rest. `$<string>` is not a generic multi-character append: append one character per operation. Operations on one line combine; different lines produce separate candidates.

```bash
hashcat --stdout <WORDLIST> -r custom.rule > candidates.txt
hashcat -m <MODE> -a 3 hashes.txt 'Password?d?d?d?d'
hashcat -m <MODE> -a 6 hashes.txt <WORDLIST> '?d?d'
```

Quote masks so shell wildcards do not expand. `-a 3` is mask attack; `-a 6` appends a mask to dictionary candidates. A wordlist is not a mask argument. Preview before committing resources to a large candidate space. John supports rules too, but its configuration/rule language is separate.

## Build relevant lists

```bash
cewl -d 2 -m 5 -w site-words.txt 'http://<HOSTNAME>/'
sort -u site-words.txt > words-unique.txt
cupp -i
crunch 8 8 -t 'pass%%%%' -o candidates.txt
```

CeWL extracts site vocabulary; CUPP builds candidates from known context; Crunch expands a character/pattern space. Crunch uses `@` lowercase, `,` uppercase, `%` digits and `^` symbols. Its `-p` permutation mode differs from fixed-length generation. Keep the search space realistic.

SecLists, RockYou, FastTrack and targeted application/default-credential lists are starting points. Username Anarchy generates username conventions from known names. Other supplied list tools include bopscrk and COOK. Deduplicate candidates while preserving meaningful capitalization. Avoid removing all digits from a list if evidence suggests numbered passwords.

## Online tests

Online guesses interact with account lockout and rate limiting. Start with recovered credentials and a small evidence-based candidate set. A password spray uses one candidate across accounts; it does not eliminate lockout risk. Obtain password policy where possible, including fine-grained AD policy and prior failed attempts.

```bash
hydra -l <USER> -P <PASSWORD_LIST> -s <SSH_PORT> -t 2 ssh://<IP>
hydra -l <USER> -P <PASSWORD_LIST> -t 2 <IP> ftp
hydra -C <USER_PASSWORD_COMBO_FILE> <IP> ftp
```

`-l`/`-p` supply a single username/password; `-L`/`-P` supply files; `-C` takes `user:password` pairs. Medusa uses a different combo layout (`host:user:password`), while Ncrack is another service-specific alternative.

### Web forms

First capture a known failed and successful request manually, including cookies and tokens. Example for a stable form:

```bash
hydra -l <USER> -P <PASSWORD_LIST> <HOSTNAME> http-post-form '/login.php:username=^USER^&password=^PASS^:F=Invalid login' -t 2
```

The route, field names, transport and failure/success marker must match the application. Basic/Digest HTTP authentication uses different modules from HTML forms. Dynamic CSRF tokens, redirects and a session expiry can make a fixed request invalid. A tool reporting every candidate as valid usually signals a bad result matcher.

The original `Spray-Passwords.ps1` code parses `net accounts` output and ignores fine-grained policies; those assumptions can be wrong across languages and domains. Treat its claimed lockout avoidance as unproven.
