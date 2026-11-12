# Local toolkit and troubleshooting

[Home](../README.md) · [Tools](Tools.md) · [Workflow](../00-Start/Workflow.md) · [Reporting](../09-Reporting/Evidence.md)

## Prepare a known-working environment

Keep a VM snapshot, required offline wordlists, local tool versions and a tested VPN. Use task-specific variables such as `TARGET`, `LHOST` and `DC_IP`; do not overwrite shell/system variables. A secret read into a variable avoids a literal in shell history but can still appear in process arguments or logs.

```bash
command -v nmap curl smbclient rpcclient nxc evil-winrm john hashcat chisel
nmap --version
python3 --version
ip address
ip route
ip route get <TARGET_IP>
```

For Python dependencies, use an isolated environment suited to the tool:

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install -r requirements.txt
```

The last command can access package servers; use previously prepared dependencies for offline work. `uv` and virtualenv are alternative environment managers in the collection. Do not patch random library files or replace system packages just to hide a parser/import error. Match upstream example names and installed versions.

## Terminal, files and compilation

`tmux new -s <NAME>`, Ctrl-B then D to detach, `tmux ls`, `tmux attach -t <NAME>`; split panes/window navigation depend on local key bindings. Name sessions by host/role to avoid issuing commands on the wrong machine. A socket writable by another identity can expose that identity's existing session; access control, socket path and actual session identity decide the impact.

Use `file`, `sha256sum`, `strings`, `readelf` and `objdump` for initial artifact inspection. `gcc -o <OUTPUT> <SOURCE.c>` compiles a native Linux program; use the appropriate MinGW compiler for Windows. Additional flags/libraries come from the code, not the source OS name alone. See [buffer-overflow adaptation](Buffer-Overflows.md).

`iconv -f UTF-8 -t UTF-16LE script.ps1 | base64 -w 0` is a Bash encoding pattern for a PowerShell EncodedCommand. Base64 is reversible encoding. Decode into a file and inspect it before using an unfamiliar encoded payload. For binary data use byte APIs, not text encodings.

Useful editor operations: in vi, `i` enters insert mode, Esc returns to normal mode, `:wq` saves/exits, `:q!` abandons changes, `/text` searches and `n` repeats. Confirm the path and permissions before editing a target configuration.

## Small data transformations

```bash
sort -u candidates.txt > candidates.unique.txt
sed '/^[[:space:]]*$/d' candidates.txt > candidates.nonempty.txt
tr -d '\r' < input.txt > input.unix.txt
```

Sorting changes candidate order. If prioritization matters, `awk '!seen[$0]++' candidates.txt` preserves first occurrence. The source's `head` plus `sed '/^1/d'` does **not** remove all numeric sequences; it samples lines and drops those starting with 1. Decide whether you mean filter all-digit lines, delete digit characters, or strip a suffix before choosing a transformation.

The JavaScript wordlist bookmarklet extracts text from the current document and deduplicates words; it does not crawl the whole site. CeWL can crawl to a selected depth with a minimum word length, `--lowercase` and optional numbers. Generate username formats from observed naming conventions instead of assuming every permutation is a valid account.

## Encoding and recovery leads

For layered encodings, preserve original bytes, identify alphabet/length, decode one layer and test whether the result has structure. Hex, Base64, URL encoding and compression are not encryption. CyberChef and local Python/OpenSSL can perform transformations. Brainfuck and Ook are esoteric languages in the supplied PDF; recognize their instruction alphabets rather than guessing a password hash. AES decoding additionally requires mode, key, IV/nonce, padding and any authentication tag; an “AES” label alone is insufficient.

Deleted-file recovery references include `extundelete` and `strings` on relevant disk/image data. Recovery depends on filesystem, access and overwritten blocks; it is distinct from reading a live file through LFI. Preserve a copy/image where possible before recovery work.

## Troubleshoot by layer

| Symptom | Controlled check |
|---|---|
| No service response | VPN/interface, route, destination state, one selected TCP connection; UDP silence is inconclusive. |
| Name works only on pivot | Query the actual DNS server, verify route to it and required FQDN/Host header. |
| Kerberos fails but NTLM works | DNS/SPN, realm, time, account scope and selected cache. |
| Login succeeds but shell fails | Service-specific authorization, token policy and required execution primitive. |
| No callback | Test outbound route and listener from the target viewpoint, including pivot listeners. |
| Download will not execute | Compare hashes; check architecture, runtime, execute permission and mount policy. |
| Browser succeeds but script fails | Preserve cookies, CSRF, Host, redirects, encoding and content type. |
| Port in use | Inspect `ss -lntp` or Windows listeners; identify owner before changing a bind. |
| Crash or exploit “success” | Check actual identity, resource access and side effect; save full error/output. |

Change one variable per test and write the expected result first. Disabling a whole firewall/AV stack or globally lowering TLS policy can conceal the cause; inspect the specific failing component.
