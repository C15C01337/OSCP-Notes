# Port forwarding and pivoting

[Home](../README.md) · [Scanning](../01-Enumeration/Scanning.md) · [Transfers](../03-Access/File-Transfers.md) · [Shells](../03-Access/Shells.md)

## Name the endpoints

- **Kali:** the attacking machine.
- **Pivot:** a host reachable from Kali that can also reach an internal service.
- **Internal target:** a destination reachable from the pivot.

Before building a tunnel, record interfaces, routes and listeners on the pivot (`ip addr`, `ip route`, `ss -lntup` on Linux; `ipconfig`, `route print`, `netstat -ano` on Windows). Decide which machine can initiate the connection. A route to an internal target does not automatically create a reverse-shell path back to Kali.

| Need | Suitable starting point |
|---|---|
| One TCP service through an SSH-accessible pivot | SSH local forwarding |
| Pivot can reach Kali's SSH server but Kali cannot initiate SSH to it | SSH remote forwarding |
| Several TCP destinations | SOCKS proxy via SSH or Chisel |
| Routed access through an agent | Ligolo-ng |
| Simple listener-to-service relay | Socat or Windows portproxy |

## SSH local forwarding

**Run on Kali:**

```bash
ssh -N -L 127.0.0.1:15432:<INTERNAL_IP>:5432 <USER>@<PIVOT_IP>
```

Kali listens on `127.0.0.1:15432`. The SSH server on the pivot connects to `<INTERNAL_IP>:5432`. Connect the database client to **Kali's listening endpoint**:

```bash
psql -h 127.0.0.1 -p 15432 -U <DB_USER> -d <DATABASE>
```

`127.0.0.1` in the destination position means loopback **on the SSH server**, useful for a service bound only on the pivot. Use `-v` and `-o ExitOnForwardFailure=yes` when diagnosing startup failures.

## SSH remote forwarding

**Run on the pivot**, where `<KALI_USER>@<LHOST>` is a usable SSH account:

```bash
ssh -N -R 127.0.0.1:15432:<INTERNAL_IP>:5432 <KALI_USER>@<LHOST>
```

The SSH server on Kali owns the listener. The SSH client on the pivot connects to the destination. Kali again uses `127.0.0.1:15432`. Server forwarding policy may reject the request. Remote wildcard bindings require appropriate server policy; prefer an explicit loopback listener when only local tools need it.

The source's `-R <port>:<port>:<target>:<port>` form has an incorrect field layout. Correct syntax is `[bind_address:]listen_port:destination_host:destination_port`.

## SSH dynamic forwarding

**Run on Kali:**

```bash
ssh -N -D 127.0.0.1:1080 <USER>@<PIVOT_IP>
```

Configure a dedicated Proxychains file with:

```text
socks5 127.0.0.1 1080
```

Then:

```bash
proxychains4 -f ./proxychains.conf nmap -sT -Pn -n -p 445 <INTERNAL_IP>
proxychains4 -f ./proxychains.conf curl 'http://<INTERNAL_IP>/'
```

Use a TCP connect scan. Ordinary SYN, ICMP and UDP probes do not traverse this TCP SOCKS setup. Resolve names on the intended side; `-n` avoids accidental DNS behavior in a scan.

OpenSSH versions supporting remote dynamic forwarding also accept `ssh -N -R 127.0.0.1:1080 <KALI_USER>@<LHOST>` **on the pivot**. That puts a SOCKS listener on Kali and uses the pivot's network. Confirm the installed version supports the shortened `-R` form.

## Chisel reverse forwards

**Kali:**

```bash
./chisel server --port 9002 --reverse
```

**Pivot, for one service:**

```bash
./chisel client <LHOST>:9002 R:127.0.0.1:15432:<INTERNAL_IP>:5432
```

**Pivot, for a SOCKS listener on Kali:**

```bash
./chisel client <LHOST>:9002 R:127.0.0.1:1080:socks
```

Point the client or Proxychains at the resulting Kali listener. Check Chisel help if using a different release's argument syntax. The target's outbound connection goes to the **Chisel server address**, not the internal service address. Source shorthand `R:socks` uses a default listener; the explicit endpoint avoids ambiguity.

## Ligolo-ng routed access

The sources mix interface-management commands from different releases. Use one consistent workflow. A manual TUN setup avoids mixing `route_add`, `add_route`, `start` and `tunnel_start` blindly.

**Kali / Bash:**

```bash
sudo ip tuntap add user "$(id -un)" mode tun ligolo
sudo ip link set ligolo up
./proxy -laddr <LHOST>:11601 -selfcert
```

**Pivot:**

```text
./agent -connect <LHOST>:11601 -ignore-cert
```

On Windows use the matching `agent.exe`. `-ignore-cert` skips certificate validation; use it deliberately for this self-signed lab setup, or configure the supported certificate-pinning/trust option.

**Proxy console:** select `session`, inspect `ifconfig`, and start the selected session's tunnel using that release's help. Older examples use `start`; other releases expose `tunnel_start --tun ligolo`.

**Kali / Bash:**

```bash
sudo ip route add <INTERNAL_SUBNET> dev ligolo
```

Confirm the route and test one internal service. Some releases provide `ifcreate`/`route_add` helpers instead; do not duplicate an already-created interface or route.

### Loopback and callbacks

Ligolo's special loopback mapping in the sources uses `240.0.0.1`. Route that address through the tunnel to reach an agent-local loopback service where supported by the installed version.

A callback listener on the pivot is distinct from Kali-to-internal routing. In the selected agent's proxy console:

```text
listener_add --addr 0.0.0.0:4444 --to 127.0.0.1:4444 --tcp
listener_list
```

The listener binds **on the agent host**; the destination is reached **from the proxy**. Start Kali's shell listener on port 4444 and configure the internal target's callback to the pivot's reachable IP and port. Do not put an IP belonging only to another internal host in the bind position. Stop the specific listener with the syntax shown by the local console help.

## Socat relay

**Pivot:**

```bash
socat -d -d TCP-LISTEN:15432,reuseaddr,fork TCP:<INTERNAL_IP>:5432
```

**Kali:** connect to `<PIVOT_IP>:15432`. The listening port and client port must match; source examples contain a `2345`/`2342` mismatch. Socat is a relay, so the pivot must accept inbound connections and reach the destination.

## Windows SSH, Plink and portproxy

Windows OpenSSH can use the same forwarding principles. Plink is an alternative SSH client:

```batch
plink.exe -ssh -l <KALI_USER> -R 127.0.0.1:13389:127.0.0.1:3389 <LHOST>
```

Run on the Windows pivot to expose its local RDP service on Kali port 13389. Authenticate and verify the SSH host key; unattended authentication configuration is a separate issue.

**Elevated CMD on a Windows pivot, for a direct TCP relay:**

```batch
netsh interface portproxy add v4tov4 listenaddress=<PIVOT_IP> listenport=2222 connectaddress=<INTERNAL_IP> connectport=22
netsh interface portproxy show all
netsh advfirewall firewall add rule name="lab_forward_2222" dir=in action=allow protocol=TCP localip=<PIVOT_IP> localport=2222
```

Kali connects to `<PIVOT_IP>:2222`. The listen address must belong to the Windows machine running `netsh`. Clean up the exact changes:

```batch
netsh interface portproxy delete v4tov4 listenaddress=<PIVOT_IP> listenport=2222
netsh advfirewall firewall delete rule name="lab_forward_2222"
```

Opening RDP does not itself bypass UAC or grant permission to add administrators. Portproxy requires sufficient privilege and supporting Windows networking services. Do not disable the entire firewall to fix one forwarding rule.

## sshuttle and framework routes

`sshuttle -r <USER>@<PIVOT_IP> <INTERNAL_SUBNET>` forwards compatible traffic through an SSH-reachable host with a usable remote Python runtime. Keep subnet scope explicit. If an extra relay leads to another SSH server, identify which server is the actual remote endpoint and which network it reaches.

Metasploit/Meterpreter routes and `portfwd` affect compatible framework sessions and modules; see [Metasploit](../03-Access/Metasploit.md). They are different from an OS TUN route.

## Diagnose each hop

Check listener ownership, local binding, destination reachability from the connecting endpoint, route, DNS, proxy configuration and firewall. Test one service before a subnet scan. For multi-hop paths, document each connection and each callback separately. Remove only the routes/listeners introduced for the exercise.
