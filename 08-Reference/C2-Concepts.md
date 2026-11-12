# Command-and-control framework concepts

[Home](../README.md) · [Shells](../03-Access/Shells.md) · [Metasploit](../03-Access/Metasploit.md) · [Client-side delivery](../03-Access/Client-Side-Delivery.md)

## Covenant terminology

The supplied Covenant walkthrough describes a management application, a listener, a launcher and a **Grunt** agent. A launcher starts an agent; it is not interchangeable with a raw shell payload or a Netcat listener.

| Setting | Meaning |
|---|---|
| Bind address/port | Endpoint owned by the listener process. |
| Connect address/port | Address embedded in the agent/launcher for its callback. |
| HTTP profile | Application-level request/response behavior expected by both sides. |
| Agent/runtime template | Payload and runtime requirements on the target. |
| Management UI | Operator interface, separate from the target-facing callback endpoint. |

Verify the bind with `ss -lntp` and the callback with an actual controlled test. A UI saying "active" does not prove the target can reach the connect address. The walkthrough's 7443 management port and .NET Core 3.1 setup are historical examples, not current installation requirements.

## Launcher families in the supplied material

| Family | Consumer/dependency |
|---|---|
| Binary | Matching native/.NET execution environment. |
| Shellcode | Compatible architecture/loader; the source names Donut for conversion. |
| PowerShell | PowerShell execution and runtime availability. |
| MSBuild | A compatible MSBuild executable consuming the generated project. |
| InstallUtil | Compatible .NET installation utility and assembly format. |
| Mshta | HTA/JScript/ActiveX behavior under client policy. |
| Regsvr32 | Matching scriptlet/COM execution path. |
| Wmic/XSL | WMI client/XSL consumer availability and behavior. |
| Cscript / Wscript | Windows Script Host and the generated script's dependencies. |

Built-in executable presence does not guarantee policy bypass. The source DotNetToJScript references depend on eligible legacy runtime behavior. Command strings must match the consumer's syntax and quoting.

## Setup and operational record

Match the project's required SDK/runtime before attempting a build. `dotnet build` compiles a compatible project and `dotnet run` starts it in the expected directory; the imported `donet` spelling and concatenated shell lines are transcription errors. Do not remove unrelated SDKs or overwrite global configuration to reproduce an old screenshot.

Keep listener configuration, generated payload hash, callback address, agent identity, job list and cleanup record together. Framework file browsers and tasking require an established agent; they do not create permissions beyond its token.

Empire, Cobalt Strike, BeEF and other frameworks appear elsewhere in the local resource lists. Their technical mention is not a statement of exam permission. The primary handbook paths remain explicit service queries, reproducible commands and captured evidence.
