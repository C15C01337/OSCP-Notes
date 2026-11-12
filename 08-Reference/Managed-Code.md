# Managed-code inspection and serialization leads

[Home](../README.md) · [Server-side requests and deserialization](../02-Web/Server-Side-Requests.md) · [Credentials](../06-Active-Directory/Credentials.md)

## Inspect a recovered .NET application

The supplied workflow downloads an application from a share, identifies it with `file` and opens it in ILSpy. Prefer static inspection before executing an unknown program.

**Kali / Bash:**

```bash
file application.exe
strings -a application.exe > application.strings.txt
ilspycmd application.exe > application.cs
```

`ilspycmd` must already be installed with a compatible runtime; GUI ILSpy is another route. Select the binary matching the host architecture, rather than copying a source instruction that downloads x64 and then changes into an ARM64 directory. Decompiled source may reveal hardcoded LDAP credentials, endpoints, decoding logic or resource files. Follow the actual transformations and test any recovered secret against its intended service.

Wine runs compatible Windows programs on Linux; it does not turn a Windows executable into a Linux command. Executing the supplied program is separate from decompiling it and may require configuration or have side effects.

## Serialization prerequisite checklist

The source `.NET` page is mainly a set of research links. Preserve its distinct entry points as investigation leads:

| Surface | Evidence needed |
|---|---|
| JSON/type metadata | Attacker-influenced type/object construction by an unsafe serializer. |
| .NET remoting over HTTP | Reachable compatible endpoint, serialization path and any authentication. |
| ViewState | Actual ASP.NET page settings, validation/encryption keys where required and a compatible formatter/gadget. |
| `.resx` resource loading | A program loads the resource through an unsafe object-deserialization path. |
| XAML/XAMLX/workflows | A component interprets attacker-controlled markup with dangerous construction/execution behavior. |
| Uploaded `web.config` | Writable relevant directory and IIS/ASP.NET configuration processing that permits the intended change. |
| Managed DCOM | Eligible activation/serialization component and rights. |

ysoserial.net, ViewStatePayloadGenerator, viewgen and RCEvil.NET appear as payload/helper tools. A gadget generator needs the right serializer, dependency/runtime versions and delivery context. It cannot create an unsafe deserialization sink where the application only treats input as text.

The linked product examples include SharePoint workflows/BDC, Exchange, SSRS, SmarterMail, Sitecore, CyberArk and Milestone XProtect. Only their names/links are supplied in those sections; use a matching local PoC/reference when present and do not infer that every deployment is affected.
