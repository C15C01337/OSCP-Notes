# Service and port map

[Home](../README.md) · [Scanning](Scanning.md) · [Auxiliary services](Auxiliary-Services.md)

Ports are identification hints; verify the actual protocol and service. Entries without a chapter were **link-only leads** in the supplied material. They are retained for recognition without pretending their external guides were read. TCP and UDP on the same number can be different services.

| Port / transport | Service | Next question / chapter |
|---|---|---|
| 21/TCP | FTP | [Anonymous/known login, readable files, upload consumer](File-Services.md) |
| 22/TCP | SSH | [Account/key, permissions, forwarding](File-Services.md) |
| 23/TCP | Telnet | Banner, authentication and daemon identity (reference lead) |
| 25,465,587/TCP | SMTP / submission / SMTPS | [Capabilities, TLS, user checks and delivery rights](Mail-SNMP.md) |
| 43/TCP | WHOIS | Domain/registration metadata; scope applies (reference lead) |
| 49/TCP | TACACS+ | Authentication service and configuration evidence (reference lead) |
| 53/TCP+UDP | DNS | [Records, explicit resolver and zone-transfer policy](DNS.md) |
| 69/UDP | TFTP | Known filenames and read/write exposure; no directory-listing assumption (reference lead) |
| 79/TCP | Finger | User-information exposure (reference lead) |
| 80,443 and other TCP | HTTP / HTTPS | [Hostnames, applications, API, TLS and parameters](../02-Web/Enumeration.md) |
| 88/TCP+UDP | Kerberos | [Realm, DC, clock, account and ticket type](../06-Active-Directory/Authentication.md) |
| 110,995/TCP | POP3 / POP3S | [TLS, mailbox identity and access](Mail-SNMP.md) |
| 111/TCP+UDP | Unix RPC portmapper | [Registered programs, NFS/mount service](File-Services.md) |
| 113/TCP | Ident | Connection-user metadata if supported (reference lead) |
| 123/UDP | NTP | [Clock alignment; separate Timeroast prerequisites](../06-Active-Directory/Roasting.md) |
| 135/TCP; dynamic TCP | MSRPC endpoint mapper | [Endpoints, bindings and query/method rights](SMB-RPC.md) |
| 593/TCP | RPC over HTTP | Source 539 is a transposition; identify actual service (reference lead) |
| 137/UDP,138/UDP,139/TCP | NetBIOS | [Names/datagrams/session service differ](SMB-RPC.md) |
| 139,445/TCP | SMB | [Protocol/signing, shares, files, identities and ACLs](SMB-RPC.md) |
| 143,993/TCP | IMAP / IMAPS | [Mailbox authentication and TLS](Mail-SNMP.md) |
| 161,162/UDP | SNMP queries / traps | [Version, community or user; trap listener is not query service](Mail-SNMP.md) |
| 194,6667 and other TCP | IRC | Banner, channels and matching backdoor/version lead (reference lead) |
| 264/TCP | Check Point service | Identify product/interface; source provides only a port lead (reference lead) |
| 389/TCP+UDP,636/TCP | LDAP / LDAPS | [RootDSE, base DN, TLS, bind and ACLs](../06-Active-Directory/Enumeration.md) |
| 500,4500/UDP | IKE / IPsec | Source lists 500; identify negotiation and NAT traversal separately (reference lead) |
| 502/TCP | Modbus | Identify protocol/unit and exposed operations before writing anything (reference lead) |
| 512/TCP | rexec | Remote execution authentication and policy (reference lead) |
| 513/TCP | rlogin | Login/trust configuration (reference lead) |
| 514/TCP | rsh | Host/account trust; distinct from UDP syslog (reference lead) |
| 515/TCP | LPD | Print service and queue exposure (reference lead) |
| 548/TCP | AFP | Share authentication and file access (reference lead) |
| 554,8554/TCP | RTSP | Stream URLs, authentication and permissions (reference lead) |
| 623/UDP | IPMI | Management interface and authentication policy (reference lead) |
| 631/TCP | IPP / CUPS | Printer/admin web surface and configuration (reference lead) |
| 700/TCP | EPP | Identify provisioning service; source is a port-only lead (reference lead) |
| 873/TCP | rsync | List modules and authorized read/write paths (reference lead) |
| 1026 / RPC-dependent | rusersd | Confirm registered RPC service instead of assuming a fixed port (reference lead) |
| 1080/TCP | SOCKS | [Proxy version, authentication and allowed destinations](../07-Pivoting/Tunnels.md) |
| 1050,1098,1099/TCP | Java RMI / related Java middleware | Registry/advertised endpoints and actual protocol; ports are hints (reference lead) |
| 1433/TCP | MSSQL | [SQL vs Windows login, identity, permissions and links](Databases.md) |
| 1521–1529/TCP | Oracle TNS | [Listener/SID/service and database authentication](Databases.md) |
| 1723/TCP + GRE | PPTP | Control channel alone does not establish a complete VPN (reference lead) |
| 1883/TCP | MQTT | Broker authentication, topic read/publish policy (reference lead) |
| 2049/TCP+UDP | NFS | [Version, exports, mapping and filesystem rights](File-Services.md) |
| 2301,2381/TCP | HP management web | Product/build and exposed management functions (reference lead) |
| 2375,2376/TCP | Docker API | TLS/client authentication and daemon access; exposure does not imply authorization (reference lead) |
| 3128/TCP | Squid / HTTP proxy | Proxy ACLs and internal reachability (reference lead) |
| 3260/TCP | iSCSI | Target discovery/authentication and block-storage access (reference lead) |
| 3299/TCP | SAProuter | Route service and configured permissions (reference lead) |
| 3306/TCP | MySQL / MariaDB | [Database identity, grants and server OS context](Databases.md) |
| 3389/TCP | RDP | [Remote Desktop rights, NLA and session restrictions](../06-Active-Directory/Lateral-Movement.md) |
| 5432/TCP | PostgreSQL | [Roles and server-side capabilities](Databases.md) |
| 5985,5986/TCP | WinRM HTTP / HTTPS | [Remoting authorization and transport](../06-Active-Directory/Lateral-Movement.md) |
| 6379/TCP | Redis | [Authentication/ACLs, selected keys and configuration](Databases.md) |
| 8021/TCP | FreeSWITCH event socket | [Binding, credentials and permitted commands](../08-Reference/Exploit-Selection.md) |
| 9389/TCP | Active Directory Web Services | [ADWS; source SCCM label is incorrect](../06-Active-Directory/Supplementary-Surfaces.md) |
| 11211/TCP or UDP | Memcached | [Protocol, exposed statistics/items and network policy](Auxiliary-Services.md) |
| 27017/TCP | MongoDB | [Authentication, accessible databases and roles](Databases.md) |
| 3268,3269/TCP | Global Catalog LDAP / TLS | [Forest-wide query scope; not a generic replication port](../06-Active-Directory/Enumeration.md) |
