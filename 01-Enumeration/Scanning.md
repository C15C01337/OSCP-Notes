# Host discovery and port scanning

[Home](../README.md) · [Workflow](../00-Start/Workflow.md) · [DNS](DNS.md) · [SMB](SMB-RPC.md) · [Web](../02-Web/Enumeration.md)

## First pass

**Kali / Bash.** Use an IP or explicitly scoped range in place of `<IP>` or `<SUBNET>`.

```bash
mkdir -p scans
nmap -sn <SUBNET> -oA scans/discovery
nmap -sT --top-ports 1000 --reason <IP> -oA scans/tcp-quick
sudo nmap -sS -p- --reason <IP> -oA scans/tcp-all
# Replace this list with the discovered ports.
nmap -sT -sV -sC -p 21,22,80,445 --reason <IP> -oA scans/services
sudo nmap -sU --top-ports 20 --reason <IP> -oA scans/udp-top
```

Use `-Pn` for a known target that does not answer discovery probes. A full TCP scan prevents uncommon listeners from being missed. A short UDP pass is only a starting point; expand it when evidence supports another service.

## Read the results correctly

| Option or result | Meaning and practical limit |
|---|---|
| `-sn` | Host discovery without a port scan; silence does not prove absence. |
| `-sS` | SYN scan; requires raw-packet privileges. |
| `-sT` | TCP connect scan; works without raw sockets and through TCP proxies. |
| `-sU` | UDP scan; unrelated to whether a TCP port of the same number is open. |
| `-sV` | Service probing; banners can be incomplete or misleading. |
| `-sC` | Default NSE scripts; inspect what they do for the service. |
| `-O` | OS fingerprinting; confirm with other evidence. |
| `-A` | Bundles version/OS detection, scripts and traceroute; not needed for every scan. |
| `-p-` | All ports 1–65535 for the selected protocol. |
| `-T4`, `-T5`, `--min-rate` | Timing controls; higher rates can miss services on a lossy link. `--min-rate` sets a minimum send rate. |
| `-oA prefix` | Normal, XML and grepable output; also available separately as `-oN`, `-oX`, `-oG`. |
| `open` | A service responded as listening. |
| `closed` | The host responded that no service is listening. |
| `filtered` | Probes were blocked or did not get a decisive response. |
| `open\|filtered` | Common with UDP; inconclusive until a service-specific response is obtained. |

## Targeted scripts

Locate scripts, inspect their descriptions and select individual checks:

```bash
ls /usr/share/nmap/scripts/smb*
nmap --script-help smb-os-discovery
nmap -p 445 --script smb-os-discovery,smb2-security-mode <IP>
nmap -sU -p 161 --script snmp-info <IP>
```

Broad expressions such as `--script vuln`, `ftp-*` or `smb-vuln*` may include intrusive checks. Select a script to answer a specific question. Source examples using `safe or smb-enum-*` select more than only SMB scripts because `or` is a union.

## Discovery without Nmap

**Target / CMD:** use `%%i` instead of `%i` in a batch file. Replace the example subnet prefix.

```batch
for /L %i in (1,1,254) do @ping -n 1 -w 200 10.10.10.%i >nul && echo 10.10.10.%i
```

**Target / Bash:** test a known service one host at a time.

```bash
for i in $(seq 1 254); do nc -zv -w 1 10.10.10.$i 445; done
```

Netcat implementations vary; a CIDR range is not a portable `nc` target. UDP Netcat silence does not prove an open port. Local-network discovery with Netdiscover depends on link-layer visibility and may not work across a routed VPN.

## Alternative scanners and unusual filtering

Masscan provides fast discovery; confirm its results with Nmap. Set the correct interface and a rate appropriate to the path. AutoRecon and NmapAutomator wrap common enumeration tasks; inspect their saved results and what each component runs. A custom scan wrapper is not a separate discovery technique.

The sources also mention Nmap fragmentation (`-f`) and decoys (`-D`). Neither makes a connection anonymous or guarantees a firewall bypass. Ordinary accurate scans and service-specific probes are the useful baseline.

If port knocking is indicated by configuration or recovered notes, preserve the sequence, protocol and timing. `knock <IP> 159:udp 27391:tcp 4:udp` is a syntax example; its numbers are not a universal sequence.

## Troubleshooting

Check target scope, VPN route, interface, host discovery and packet loss. Repeat a disputed port at a lower rate. For a TCP SOCKS proxy, use a connect scan with host discovery and DNS behavior considered explicitly; raw SYN and ordinary UDP scans are not carried by a TCP proxy.
