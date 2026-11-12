# Client-side files and delivery prerequisites

[Home](../README.md) · [Browser attacks](../02-Web/Browser-Attacks.md) · [Shells](Shells.md) · [Transfers](File-Transfers.md) · [C2 concepts](../08-Reference/C2-Concepts.md)

## Identify the consumer and action

A file must be opened, previewed, rendered or otherwise processed by an eligible client. Record the application/version, required interaction, policy, resulting execution account and callback route. Sending a message or storing a file on a share does not establish execution.

## Word and LibreOffice macros

The source Word workflow creates a macro-enabled document and uses `AutoOpen` / `Document_Open` to call VBA. A minimal controlled proof is:

```vba
Sub Document_Open()
    MsgBox "Macro executed"
End Sub
```

Use the appropriate VBA module/event location and a macro-capable format. `.docx` does not retain VBA macros; `.docm` and eligible legacy formats do. Macro policy, source trust and application settings may block execution; an "Enable" prompt is not guaranteed to be offered.

The supplied execution examples use `WScript.Shell.Run` or `Shell` to launch a command. Long strings need valid VBA concatenation/quoting, and embedded PowerShell requires the correct encoding. Use the [shell reference](Shells.md) after establishing a harmless macro proof rather than copying a wrapped transcript.

LibreOffice's ODT example attaches a Basic macro to **Tools → Customize → Events → Open Document**. Execution remains subject to macro security and user action. `mraptor` can inspect suspicious Office macro content; it does not prove a document will execute on the target client.

## HTA and Windows Script Host

HTA uses the Windows `mshta` consumer and may expose ActiveX/WSH functionality under eligible policy. Ordinary browser JavaScript is a different context. A source HTML file that invokes `WScript.Shell` will not execute OS commands in every browser simply because it contains a script tag.

MSFvenom's `hta-psh` format is a lab delivery option requiring a matching payload, client execution and listener. Establish the exact consumer before testing a callback. WSH/VBScript/JScript availability and policy differ across systems.

## Windows libraries, WebDAV and shortcuts

The supplied `.Library-ms` technique points a library connector at a WebDAV folder. A separate `.lnk` is then launched by a user. Opening the library and executing the shortcut are separate stages.

**Normalized library XML:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
  <name>Shared documents</name>
  <version>6</version>
  <isLibraryPinned>true</isLibraryPinned>
  <templateInfo>
    <folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
  </templateInfo>
  <searchConnectorDescriptionList>
    <searchConnectorDescription>
      <isDefaultSaveLocation>true</isDefaultSaveLocation>
      <isSupported>false</isSupported>
      <simpleLocation><url>http://<LHOST>:8000/</url></simpleLocation>
    </searchConnectorDescription>
  </searchConnectorDescriptionList>
</libraryDescription>
```

Replace the URL placeholder before using the XML. **Kali, for an intentionally served staging folder:**

```bash
wsgidav --host=<LHOST> --port=8000 --auth=anonymous --root=<STAGING_DIRECTORY>
```

A normal Python download server is not a WebDAV server. The Windows client needs compatible WebDAV support, reachability and policy. Shortcut target and arguments must be explicit; a `.lnk` file is a binary shortcut, not merely text renamed with that extension.

## Icon/URL authentication triggers

Historical `.url`/`.scf` examples reference an SMB icon path or file URL. On eligible clients, rendering/processing the file can cause a remote connection. This is an authentication/capture opportunity, not automatic code execution. See [NTLM capture and relay](../06-Active-Directory/Relay-Coercion.md).

## Other link-only ideas retained from the sources

PowerPoint actions, Word subdocuments/DDE, document properties, CSV formula injection, browser-origin payloads and removable-media/USB delivery all require a matching consumer and specific trust/interaction conditions. Gophish, Luckystrike, MailSniper, Bash Bunny and framework delivery modules appear as tools/resources. The supplied links do not provide a complete tested workflow for each.

Keep message/file names, exact content, delivery method, required action and resulting process evidence with the target record. Remove staged files and any created shortcuts after practice.

## SMTP delivery tool fields

Swaks can submit a message using the selected SMTP server, sender, recipient, subject/body, authentication and attachment. Source patterns use `--server`, `--from`, `--to`, `--header`, `--body @<BODY_FILE>` and `--attach @<ATTACHMENT>`. Check local attachment syntax and authenticate only where required. Successful SMTP acceptance proves submission, not client execution. Multiple recipients, external callbacks and real delivery are separate actions to scope deliberately; no messages were sent during this consolidation.
