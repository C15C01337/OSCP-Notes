# Auxiliary services and protocol checks

[Home](../README.md) · [Service map](Service-Map.md) · [Scanning](Scanning.md) · [Databases](Databases.md)

## Memcached

On an exposed text-protocol TCP service, connect with `nc <IP> 11211`. Commands inside the session:

```text
version
stats
stats items
stats slabs
stats cachedump <SLAB_ID> <LIMIT>
get <KEY>
quit
```

Use actual IDs/keys returned by the service. `cachedump` availability and completeness vary; exposed metadata does not guarantee every item is retrievable. Binary/SASL deployments differ from the text protocol. Avoid changing or flushing cache contents during enumeration.

## Port knocking

Source examples use `knock` to send a configured sequence of TCP/UDP attempts:

```bash
knock -v <IP> <PORT1> <PORT2> <PORT3>
knock -v <IP> <PORT1>:udp <PORT2>:tcp <PORT3>:udp
```

This requires the **actual sequence, transport and timing**, often discovered in readable `knockd.conf` or application notes. It does not infer a sequence from closed ports. Rescan only the intended service after the sequence and account for source-IP filtering or time-limited access.

## Rsync and TFTP

For a known rsync service, `rsync rsync://<IP>/` lists exposed modules when permitted; `rsync --list-only rsync://<IP>/<MODULE>/` inspects a module. Authentication, module restrictions and write permissions are separate.

TFTP normally transfers a **known filename** over UDP; it does not provide FTP-style directory browsing or ordinary password authentication. Client availability and the server's root directory constrain paths. Compare transferred bytes/checksums and use binary mode where the client exposes it.

## Packet and process observations

`tcpdump -i <INTERFACE> -nn -s0 -w capture.pcap '<CAPTURE_FILTER>'` saves packets; `tcpdump -r capture.pcap -nn` reads them. A capture filter and a Wireshark display filter have different syntax. Loopback capture can expose local cleartext protocols only when traffic exists and the process has capture rights/capabilities. It is not a way for an arbitrary unprivileged user to read every service password.

Process arguments, environment and opened configuration files are separate evidence sources. Preserve the relevant command line/configuration and resulting identity, rather than assuming every occurrence of “pass” is a usable credential.
