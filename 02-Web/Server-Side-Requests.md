# SSRF, XXE and unsafe deserialization

[Home](../README.md) · [Web enumeration](Enumeration.md) · [File attacks](File-Attacks.md) · [Template injection](Template-Injection.md)

## Server-side request forgery

**Trigger:** an application fetches a URL, host, document, image or import source influenced by input. Establish which part is controlled: scheme, hostname, port, path, query or redirect target.

1. Compare a known working URL with a controlled endpoint and a nonexistent one.
2. Observe whether the **server** makes the request, using response content or a controlled listener.
3. Identify protocol restrictions, redirects, name resolution and whether responses are returned.
4. Test a relevant internal service only after establishing this behavior.

The source example changes a `server` parameter and uses query delimiters to influence a downstream URL. Its effect depends on how that application constructs the request. An external image loading in your browser does not prove SSRF.

Record the submitted URL, actual destination, callback request and returned response. Reachability from the application server may differ from reachability from Kali.

## XML external entity processing

**Prerequisite:** attacker-controlled XML is parsed with DTD/external entity behavior enabled. Place the entity reference in a field the application actually processes.

```xml
<?xml version="1.0"?>
<!DOCTYPE request [
  <!ENTITY example SYSTEM "file:///etc/passwd">
]>
<request><value>&example;</value></request>
```

For a Windows file-read proof, a matching candidate is `file:///C:/Windows/win.ini`. For a controlled network callback, use an HTTP entity URL pointing at your listener.

Send the XML using the method and content type expected by the endpoint. Let the HTTP client/proxy recalculate `Content-Length`; do not copy the source's fixed value. A remote entity fetching a `.php` URL normally retrieves the web server's response, not PHP source to execute in the XML parser.

Parser errors, an expanded file value and an outbound callback are different evidence levels. Blind XXE may show only a callback. Modern parser configurations often disable external entity resolution.

## Unsafe deserialization and PHPGGC

The collection references PHP Generic Gadget Chains (PHPGGC), including a Guzzle file-write chain. A gadget payload requires all of the following:

- An attacker-controlled value reaches an unsafe deserializer.
- The application's dependencies contain a compatible gadget chain/version.
- Serialization format and transport encoding are correct.
- The chain's side effect is permitted by the process identity and filesystem.

`phpggc -u --fast-destruct Guzzle/FW1 <REMOTE_PATH> <LOCAL_CONTENT_FILE>` is the source's generator pattern; inspect the local tool's chain description before choosing arguments. A serialized payload sent to an ordinary string field has no special effect.

The supplied file titled “Bad YAML” is an **Ansible playbook containing a command task**, not a YAML deserialization payload. Its relevant boundary is permission to submit a playbook to a privileged Ansible process; see [privileged consumers](../04-Linux/Exploitation-Patterns.md#ansible-playbooks).

## Files processed after upload

The original upload matrix lists SVG/XML/PDF/Office/media/archive processing as possible entry points for XXE, SSRF, XSS or file inclusion. Keep the consumer in view: an SVG opened in a browser, an XML parser on the server and an image-conversion library expose different behavior. A file extension by itself does not prove any of those bugs.
