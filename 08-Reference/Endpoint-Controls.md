# Endpoint detection and execution controls

[Home](../README.md) · [Shells](../03-Access/Shells.md) · [Windows enumeration](../05-Windows/Enumeration.md) · [C2 concepts](C2-Concepts.md)

## Detection concepts

| Mechanism | Evidence it can use |
|---|---|
| Signatures | Known byte patterns, strings and file structures. |
| Heuristics | Suspicious code structure, imports or combinations of traits. |
| Behavioral analysis | Process, memory, filesystem and network actions; may be live or sandboxed. |
| Statistical/ML classification | Features learned from labeled data and contextual telemetry. |

Changing variable names, encoding a command or packing a binary may alter a signature while leaving behavior visible. An encoder or a renamed file does not establish successful evasion.

## Mechanisms described in the sources

- **Obfuscation:** transforms representation/control flow while retaining behavior.
- **Packing/encryption:** stores content in a transformed form and restores it at runtime; the loader remains executable behavior.
- **Local shellcode loading:** allocates memory in the current process, copies code and starts a thread.
- **Remote process injection:** writes/executes code in a different process with the necessary access.
- **Process hollowing:** changes a created process's execution image/context under suitable implementation conditions.

The duplicated PowerShell source calls `VirtualAlloc`, `memset` and `CreateThread` in the current process. It is **not** a complete remote-process injection or hollowing example. Its pointer arithmetic uses `ToInt32`, which can truncate addresses in a 64-bit process. Do not treat it as portable shellcode-loader code.

Shellter is a PE modification/injection tool in the sources; UPX is a packer; Donut converts supported payloads to shellcode. Each has architecture/runtime and detection limits.

## Distinct Windows controls

PowerShell execution policy, AMSI, script-block logging, AppLocker/WDAC, PPL and Credential Guard enforce or observe different things. `-ExecutionPolicy Bypass` affects script-policy handling; it does not disable AMSI or grant administrator access. Loading a script into memory can still generate telemetry and be scanned.

`Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections` inspects effective AppLocker rules where available. A writable directory is not automatically an execution-policy bypass; the rule condition and executing identity matter.

The collection includes historical references to AMSI patching, script-block logging bypass, argument spoofing, Gargoyle, BYOL, Invisi-Shell, LOLBAS, Sysmon configuration gaps and honeytoken identification. These are separate research leads, not guaranteed current bypasses. Event-log disabling tools such as Invoke-Phant0m are not necessary baseline enumeration steps.

Record the observed error or blocking control before changing payloads. A failed callback can be routing or quoting rather than security software. Do not disable broad controls merely to reproduce a source author's setup.
