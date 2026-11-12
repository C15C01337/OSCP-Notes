# Web enumeration

[Home](../README.md) · [Workflow](../00-Start/Workflow.md) · [SQL injection](SQL-Injection.md) · [File attacks](File-Attacks.md) · [Command injection](Command-Injection.md) · [Templates](Template-Injection.md)

## Map the application first

Browse every identified HTTP service, including uncommon ports and HTTPS. Record the full base URL, redirects, hostname, server banner, framework, login behavior and session cookies.

- Read source, comments, linked JavaScript, forms and API requests. Inspect the browser network panel and replay interesting requests in a proxy.
- Check `robots.txt`, referenced paths, backup files, exposed `.git`, admin panels, diagnostic pages and configuration disclosures.
- Inspect TLS certificate names as hostname leads. A certificate name still needs to be tested against the service.
- Compare the application response in the browser with the actual HTTP request/response. Cookies, redirects, proxy settings and JavaScript can account for differences.

**Kali / Bash:**

```bash
curl -i 'http://<IP>:<PORT>/'
curl -I 'http://<IP>:<PORT>/'
curl -i -X OPTIONS 'http://<IP>:<PORT>/'
whatweb 'http://<IP>:<PORT>/'
```

`HEAD` and `OPTIONS` may behave differently from `GET`; confirm useful findings with an ordinary request. Use `curl -k` only when a known certificate-validation problem prevents an intentional test.

## Content discovery

Start with a small relevant wordlist. Follow meaningful results and add extensions appropriate to the technology.

```bash
gobuster dir -u 'http://<IP>:<PORT>/' -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak
ffuf -u 'http://<IP>:<PORT>/FUZZ' -w <WEB_WORDLIST> -o ffuf.json
feroxbuster -u 'http://<IP>:<PORT>/' -w <WEB_WORDLIST> -o ferox.txt
wfuzz -u 'http://<IP>:<PORT>/api/FUZZ' -w <WEB_WORDLIST>
```

Send a random nonexistent path first. Some applications return a success status and a generic page for every path. Compare size, words, lines, redirects and content before filtering. A 403 can identify a real resource; it is not proof that bypassing access control is possible.

Common local lists include `/usr/share/wordlists/dirb/common.txt`, `dirb/big.txt`, `dirbuster/directory-list-2.3-medium.txt` and SecLists `Discovery/Web-Content/raft-medium-directories.txt`. Installation paths vary; locate the actual file instead of copying inconsistent `Web_Content`, `Web-Content` or case variants from a source example.

## Virtual hosts

Find names in redirects, certificates, DNS, HTML and configuration. Test an HTTP hostname independently of its DNS record:

```bash
curl -i -H 'Host: <HOSTNAME>' 'http://<IP>/'
ffuf -u 'http://<IP>/' -H 'Host: FUZZ.<DOMAIN>' -w <DNS_WORDLIST> -fs <BASELINE_BYTES>
```

Measure `<BASELINE_BYTES>` from an invalid hostname. Gobuster `vhost` is another option, but its domain-append behavior depends on the version. For HTTPS, preserve both the hostname used for TLS SNI and the HTTP host:

```bash
curl --resolve '<HOSTNAME>:443:<IP>' 'https://<HOSTNAME>/'
```

An `Origin` header/CORS difference is a separate behavior from virtual-host routing. Do not discard a meaningful result just because it returns the same status as the baseline.

## Parameters, APIs and authenticated areas

Enumerate parameters already observed in forms or JavaScript before guessing new ones. Retain content type, cookies, CSRF tokens, method and encoding when replaying a request.

```bash
ffuf -request request.txt -request-proto http -w <WORDLIST>
ffuf -u 'http://<HOSTNAME>/item?name=FUZZ' -w <WORDLIST> -H 'Cookie: PHPSESSID=<SESSION>'
```

Place `FUZZ` in the request file at the value being tested. Wfuzz can vary multiple values; ffuf can vary paths, extensions, headers or JSON/form values. Limit concurrency/rate when the service becomes unstable. Authentication/session expiry can invalidate an otherwise correct enumeration run.

## CMS and product-specific leads

For WordPress, collect core, plugin and theme versions, usernames, exposed backups and uploads. WPScan examples in the sources include:

```bash
wpscan --url 'http://<HOSTNAME>/' --enumerate u,p,t --output wpscan.txt
```

`vp`/`vt` select vulnerable plugins/themes; `ap`/`at` broaden discovery; `cb`/`dbe` look for configuration backups/database exports. Vulnerability data and API-token behavior depend on the installed version and available database. Manual confirmation remains necessary. Joomla/Drupal tools mentioned in the collection include JoomScan and drupwn.

## Scanners and legacy web-server leads

The collection mentions Nikto, Nuclei, Wapiti, W3af, ZAP, WPProbe, WhatWeb and technology-aware wordlist helpers. They serve different purposes; a scanner finding is a lead requiring verification. Keep tool permission decisions with current exam instructions.

Historical TLS/web-server leads include Heartbleed, POODLE, OpenFuck, Apache Struts and Shellshock. Match the actual TLS library, server component and patch level. A server header alone does not prove any of them. Shellshock additionally needs an exposed path that invokes vulnerable Bash with attacker-controlled environment data.

## Fuzzing controls and recovered source

| Tool | Controls | Interpretation |
|---|---|---|
| ffuf | `-mc`, `-fc` | Match/filter status; check actual defaults. |
| ffuf | `-fs`, `-fw`, `-fl` | Filter bytes, words or lines using a measured baseline. |
| ffuf | `-ac` | Automatic calibration; verify it preserves meaningful responses. |
| ffuf | `-e`, `-recursion`, `-recursion-depth` | Extensions and controlled recursion. |
| ffuf | `-t`, `-rate`, `-timeout` | Concurrency, rate and timeout are different controls. |
| ffuf | `-X`, `-d`, `-H`, `-b` | Method, body, headers and cookies; retain form/JSON encoding. |
| Wfuzz | `--hc`, `--hw`, `--hh`, `--hl` | Hide status, words, characters or lines; two dashes matter. |

Wfuzz `FUZZ` and `FUZ2Z` correspond to separate `-z` payload specifications. Choose paired versus product behavior deliberately where supported. PID/numbered-file discovery still needs an accessible resource and an authorization test. A PHPSESSID cookie preserves a session only while it remains valid.

For exposed Git metadata, GitTools' Finder/Dumper/Extractor and git-dumper recover different parts of a repository. Inspect a recovered copy with `git log --all` and `git show <COMMIT>`; old/deleted secrets can remain in history. `.git/config` alone is not every blob/commit. Do not execute recovered hooks/builds during inspection.

WordPress theme editing or plugin upload/activation can execute PHP with the required administrative capability, enabled editor/installation policy, filesystem permissions and handler. A plugin needs a valid header and an actual load/activation trigger; it is not an unauthenticated upload bypass. Restore the original theme/plugin. Droopescan and Joomla-specific scripts are additional source tools with distinct CMS support.
