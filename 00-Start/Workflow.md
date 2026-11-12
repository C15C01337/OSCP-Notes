# Target workflow

[Home](../README.md) · [Scanning](../01-Enumeration/Scanning.md) · [Web enumeration](../02-Web/Enumeration.md)

## Prepare a notebook

Use one directory per target and one record per finding. Keep a separate inventory of hostnames, credentials, routes and evidence. On Kali:

```bash
mkdir -p target/{scans,web,loot,evidence,exploits}
```

Record the target IP, supplied credentials, known domain, VPN interface and callback address. Check connectivity and routes before interpreting a timeout as a closed service.

## Work from observations

1. [Discover services](../01-Enumeration/Scanning.md). Save a quick TCP scan, a complete TCP scan and a targeted UDP pass.
2. Enumerate every exposed service. Record banners, hostnames, anonymous access, files, application paths and authentication methods.
3. For each lead, write `observation → hypothesis → smallest test → result → next action`.
4. Validate the specific prerequisite before choosing an exploit: product, version, platform, configuration, authentication and required permissions.
5. After access, confirm identity and host. Save the successful request or command before changing anything else.
6. Enumerate local permissions, services, scheduled tasks, credentials and internal listeners. Revisit network services with each new credential or route.
7. Capture the required proof and build a reproducible attack path as you work.

## Choose a lead

| Evidence | Next question |
|---|---|
| Readable share, backup or exposed repository | Does it reveal application source, credentials, users or internal names? |
| Login page | Is there a documented default, recovered credential, reuse or a specific injection hypothesis? |
| Software version | Does an exploit match the exact component and configuration? |
| File upload | Where is it stored, can it be retrieved, and how is it processed? |
| File read | Can configuration, source or a private key connect this to another service? |
| New password or hash | Which host, service and account namespace accept it? |
| Local access | What does this identity control that a more privileged process consumes? |

Avoid treating a version string or scanner label as proof. A writable FTP directory is useful only when a separate mechanism serves, executes or consumes its contents.

## When stuck

Review saved output before launching another scan. Check omitted ports, alternate hostnames, virtual hosts, redirects, API endpoints, comments, backups and credentials. Confirm that the request reaches the intended application and that any listener is reachable from the target.

After a focused block without new evidence, leave a resume note and change hypothesis or service. Record why a path failed: wrong version, missing privilege, blocked route, rejected syntax or confirmed patched behavior. A timeout alone does not distinguish these cases.

## Record every transition

For each access or privilege change, capture:

- Target IP and hostname.
- Account name, domain or local context, and effective privilege.
- Exact exploit/request/command and any modified files.
- Relevant output, proof evidence and screenshot filename.
- Credentials or routes gained and the next smallest test.

Keep a list of changes to undo, including uploaded files, listeners, configuration changes and accounts created during practice.
