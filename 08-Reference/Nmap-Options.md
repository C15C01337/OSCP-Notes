# Nmap option reference

[Home](../README.md) · [Practical scan workflow](../01-Enumeration/Scanning.md)

Use the practical workflow for the first pass. This page consolidates the longer option list from the imported notes and corrects mislabeled flags. Options select different probes; none guarantees a complete or undetected scan.

## Targets and names

| Option | Meaning |
|---|---|
| `-iL targets.txt` | Read explicit targets from a file. |
| `--exclude <TARGETS>` / `--excludefile file` | Exclude hosts from the supplied target set. |
| `-sL` | List selected targets; consider whether reverse DNS lookups are desired. |
| `-6` | IPv6 target mode. |
| `-n` / `-R` | Disable / force reverse DNS resolution. |
| `--dns-servers <SERVERS>` | Select resolvers for supported lookup operations. |
| `--system-dns` | Use the system resolver. |
| `--randomize-hosts` | Change scan order within selected targets. |

Random target generation (`-iR`) is present in the original option dump but is unnecessary when working from an assigned target list. A list of hosts separated by whitespace is different from an IP octet expression containing commas.

## Discovery and scan type

| Option | Meaning / limit |
|---|---|
| `-sn` | Discovery only; legacy examples use `-sP`. |
| `-Pn` | Skip discovery, treat selected hosts as online. |
| `-PS<PORTS>` / `-PA<PORTS>` | TCP SYN / ACK discovery probes. |
| `-PU<PORTS>` | UDP discovery probe. |
| `-PY<PORTS>` | SCTP INIT discovery probe. |
| `-PE` / `-PP` / `-PM` | ICMP echo / timestamp / address-mask probes. |
| `-PO<PROTOCOLS>` | IP-protocol discovery probes. |
| `-PR` | ARP discovery where the target is on the relevant local link. |
| `-sS` / `-sT` | SYN / TCP connect port scan. |
| `-sU` | UDP port scan. |
| `-sN` / `-sF` / `-sX` | TCP NULL / FIN / Xmas probes; interpretation depends on the responding stack/firewall. |
| `-sA` | ACK scan for filtering behavior; does not directly distinguish open from closed ports. |
| `-sI <ZOMBIE>` | Idle scan using a suitable idle intermediary and predictable IP-ID behavior. |
| `--scanflags <FLAGS>` | Custom TCP flags; use a clear hypothesis for interpretation. |
| `-sO` | IP protocol scan, distinct from TCP/UDP port enumeration. |

The source labels `-sA` as Xmas; the correct Xmas flag is `-sX`. Discovery is not selected by `-p`: `-p` selects ports. Several old ping flags in the source are obsolete or inconsistent; use the supported discovery options above.

## Ports, identification and scripts

| Option | Meaning |
|---|---|
| `-p 22,80,445` | Selected numeric ports; service names may also be accepted. |
| `-p U:53,161,T:22,445` | Separate UDP/TCP port sets when those scan types are enabled. |
| `-p-` | Ports 1–65535 for enabled scan protocols. |
| `--top-ports <N>` / `-F` | Most common N ports / reduced fast set. |
| `-r` | Sequential port order. |
| `-sV` / `--version-trace` | Service probing / detailed probe trace. |
| `-O` / `--osscan-guess` | OS fingerprinting / more aggressive guessing. |
| `-sC` / `--script <NAME>` | Default scripts / selected script or expression. |
| `--script-args ...` | Script-specific arguments; inspect the script's help. |
| `--traceroute` | Trace supported routes after scanning. |
| `-A` | Bundles OS/version detection, default scripts and traceroute. |

Historical RPC scan `-sR` examples are superseded in ordinary service identification by integrated version probing. A script's `smbusername`, `smbuser`, password/hash and domain arguments must match its actual library/version; copying inconsistent names can silently omit authentication.

## Timing and diagnostics

| Option | Meaning |
|---|---|
| `-T0` through `-T5` | Timing templates, from conservative to aggressive; `-T3` is the ordinary default. |
| `--min-rate` / `--max-rate` | Lower / upper sending-rate bound. |
| `--min-parallelism` / `--max-parallelism` | Probe parallelism bounds. |
| `--min-hostgroup` / `--max-hostgroup` | Target-group sizing bounds. |
| `--initial-rtt-timeout` / `--max-rtt-timeout` | Initial / maximum probe-response timeout. |
| `--max-retries` | Retransmission limit. |
| `--host-timeout` | Bound time spent on a host. |
| `--scan-delay` / `--max-scan-delay` | Probe-delay controls. |
| `--defeat-rst-ratelimit` | Trade accuracy/closed-port handling for speed under RST rate limiting. |
| `--reason`, `-v`, `--packet-trace` | More information about scan decisions and packets. |

The source reverses initial/max RTT labels and uses lowercase `-t` for timing; those are corrected here. Very low timeouts can produce false negatives. Repeat a disputed result under a more conservative setting.

## Packet construction references

`-f`/`--mtu` fragment eligible raw probes; `-D` adds decoys; `--source-port` selects a source port; `--data-length` adds padding; `--spoof-mac` changes eligible link-layer frames; `--ttl` selects TTL; `--send-eth`/`--send-ip` choose sending paths; `--badsum` deliberately sends invalid checksums. Their applicability depends on protocol, privilege and network path. In particular, spoofed local MAC values do not carry through a routed VPN, and these options do not make a TCP connect scan anonymous.
