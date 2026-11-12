# DNS and hostname discovery

[Home](../README.md) · [Scanning](Scanning.md) · [Web virtual hosts](../02-Web/Enumeration.md#virtual-hosts)

## Questions to answer

Identify the domain, authoritative nameserver, mail hosts, address records, internal names and reverse mappings. DNS commonly uses UDP and TCP 53; AXFR zone transfers use TCP. TCP DNS is not itself evidence of an allowed zone transfer.

| Record | Meaning |
|---|---|
| A / AAAA | IPv4 / IPv6 address |
| NS | Authoritative nameserver |
| SOA | Zone authority and administrative metadata |
| MX | Mail exchanger |
| CNAME | Alias to another DNS name |
| PTR | Reverse mapping from address to name |
| TXT | Text, sometimes revealing domain or service configuration |
| SRV | Service location, useful when identifying AD services |

## Query a chosen server

**Kali / Bash:**

```bash
dig @<DNS_IP> <DOMAIN> A
dig @<DNS_IP> <DOMAIN> NS
dig @<DNS_IP> <DOMAIN> SOA
dig @<DNS_IP> <DOMAIN> MX
dig @<DNS_IP> <DOMAIN> TXT
dig @<DNS_IP> -x <IP>
dig @<DNS_IP> version.bind TXT CH
dig @<DNS_IP> <DOMAIN> AXFR
host -t A <HOSTNAME> <DNS_IP>
host -l <DOMAIN> <DNS_IP>
```

**Windows / CMD or PowerShell:**

```text
nslookup -type=NS <DOMAIN> <DNS_IP>
nslookup -type=MX <DOMAIN> <DNS_IP>
nslookup <IP> <DNS_IP>
```

Repeat AXFR against each authoritative server and each discovered zone. A transfer must name a zone; omitting it does not enumerate every zone. `ANY` responses may be limited, so query individual record types.

`nslookup -debug -type=A <DOMAIN> <DNS_IP>` exposes query/response details including TTLs; `-timeout=<SECONDS>` adjusts the wait in compatible versions. Use `SOA`, not the source CSV typo `OSA`. Reverse IPv4 PTR queries can explicitly use reversed octets under `in-addr.arpa`; passing the IP directly lets the client construct that lookup. A PTR name is a record value, not proof of host ownership or authorization.

## Name enumeration

```bash
dnsrecon -d <DOMAIN> -t std
dnsrecon -d <DOMAIN> -D <DNS_WORDLIST> -t brt
dnsenum <DOMAIN>
gobuster dns -d <DOMAIN> -w <DNS_WORDLIST>
```

Keep discovered names with their source and resolved IP. Add deliberate static mappings to `/etc/hosts` when necessary. A hosts-file entry helps local name resolution; it does not change the DNS server's records.

## Passive leads in the supplied material

The original notes cover WHOIS, Netcraft, public repository searches and search-engine operators such as `site:<DOMAIN> intitle:"Index of"`. These are supplementary reconnaissance ideas. No external lookups were performed for this handbook.

For an acquired repository, inspect files and history for credentials and configuration. Gitleaks and Gitrob are mentioned as secret-discovery aids. A web-exposed `.git` directory usually needs a repository recovery tool rather than assuming `git clone http://<IP>/.git` will work.

## Distinguish DNS from web routing

DNS discovery tests whether a name resolves. HTTP `Host` fuzzing tests how a web server routes a request and can reveal a virtual host with no public DNS record. Use the [web enumeration procedure](../02-Web/Enumeration.md#virtual-hosts) for that case.
