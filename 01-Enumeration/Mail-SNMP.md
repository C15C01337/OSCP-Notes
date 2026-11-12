# Mail services and SNMP

[Home](../README.md) · [Scanning](Scanning.md) · [DNS](DNS.md)

## SMTP — TCP 25, 465, 587

**Goal:** identify the mail service, capabilities, host/domain information and any user-enumeration behavior.

**Kali / Bash:**

```bash
nc -nv <IP> 25
openssl s_client -starttls smtp -crlf -connect <IP>:587
openssl s_client -crlf -connect <IP>:465
nmap -p 25,465,587 --script smtp-commands,smtp-ntlm-info <IP>
```

Use implicit TLS on 465 and STARTTLS where advertised, commonly on 25/587. Inside the SMTP connection:

```text
EHLO client.example
VRFY <USER>
QUIT
```

`VRFY`, `EXPN` and recipient responses may reveal users, but generic responses and catch-all delivery cause false positives. Compare a known-invalid name. A refusal may reflect disabled enumeration rather than an invalid account. Nmap `smtp-enum-users` and the standalone `smtp-user-enum` tool automate supported enumeration methods; their command-line syntax differs.

SMTP flow for understanding a recovered transcript:

```text
EHLO client.example
MAIL FROM:<sender@example.test>
RCPT TO:<recipient@example.test>
DATA
Subject: Example

Message body
.
QUIT
```

`DATA` ends with a line containing only a dot. Message acceptance is not proof that the recipient read or executed anything. Source client-side delivery examples require a separate user/application action. Configuration leads include `sendmail.cf` and `submit.cf`.

## POP3 — TCP 110, 995

```bash
nc -nv <IP> 110
openssl s_client -crlf -connect <IP>:995
nmap -p 110,995 --script pop3-capabilities,pop3-ntlm-info <IP>
```

Inside the POP3 connection:

```text
CAPA
USER <USER>
PASS <PASSWORD>
STAT
LIST
RETR 1
QUIT
```

`+OK` indicates success and `-ERR` failure. `STAT` summarizes the mailbox; `LIST` lists message sizes; `RETR` retrieves a message. `DELE` marks messages for deletion, and `RSET` resets deletion marks in the session. Read relevant messages for credentials, internal names and application context.

## IMAP — TCP 143, 993

```bash
nc -nv <IP> 143
openssl s_client -crlf -connect <IP>:993
nmap -p 143,993 --script imap-ntlm-info <IP>
```

IMAP uses tagged commands. A minimal authenticated read session is:

```text
a1 CAPABILITY
a2 LOGIN <USER> <PASSWORD>
a3 LIST "" "*"
a4 SELECT INBOX
a5 FETCH 1 BODY.PEEK[]
a6 LOGOUT
```

Quote credentials containing spaces or protocol-significant characters. NTLM information scripts for SMTP, POP3 and IMAP can expose host/domain metadata when the service supports NTLM. That response is metadata, not a recovered password or proof of successful authentication.

## SNMP — usually UDP 161

**Goal:** discover a valid community and query management data. Versions 1/2c do not encrypt the community string or responses. Read and write capabilities are separate; `public` and `private` are candidate strings, not guaranteed defaults.

```bash
sudo nmap -sU -p 161 <IP>
onesixtyone -c <COMMUNITY_WORDLIST> -i <HOST_LIST>
snmpwalk -v2c -c '<COMMUNITY>' <IP> 1.3.6.1.2.1.1
snmpwalk -v2c -c '<COMMUNITY>' <IP> 1.3.6.1.2.1.25.4.2.1.2
snmpwalk -v2c -c '<COMMUNITY>' <IP> NET-SNMP-EXTEND-MIB::nsExtendOutputFull
```

Use `-v1` when needed. Named MIB lookups require local MIB definitions; a missing local MIB is different from a remote access denial. UDP 162 commonly receives traps rather than ordinary management queries. SNMPcheck, snmp-check and snmpenum are alternative enumeration tools in the sources; check the installed executable's syntax.

| Numeric OID | Data to inspect |
|---|---|
| `1.3.6.1.2.1.1` | System description and identity |
| `1.3.6.1.2.1.25.1.6.0` | Number of processes |
| `1.3.6.1.2.1.25.4.2.1.2` | Running program names |
| `1.3.6.1.2.1.25.4.2.1.4` | Executable paths |
| `1.3.6.1.2.1.25.2.3.1.4` | Storage descriptions |
| `1.3.6.1.2.1.25.6.3.1.2` | Installed software |
| `1.3.6.1.4.1.77.1.2.25` | Windows user accounts where exposed |
| `1.3.6.1.2.1.6.13.1.3` | TCP local ports in the legacy TCP connection table |

NET-SNMP extended output may reveal script output, credentials or internal paths. Writing an extension table can execute a command only when the agent exposes the required writable objects and permissions. The source `snmpset` example omits portable type arguments, so it is not presented as a ready-to-run command. Establish the agent implementation and use its actual MIB schema; read-only community access does not grant execution.
