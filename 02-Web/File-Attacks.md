# Traversal, inclusion and file upload

[Home](../README.md) · [Web enumeration](Enumeration.md) · [Command injection](Command-Injection.md) · [SSH keys](../01-Enumeration/File-Services.md#ssh--tcp-22)

## Distinguish the behavior

| Behavior | What it proves | Additional requirement for execution |
|---|---|---|
| Path traversal | A path escapes its intended directory. | A separate execution mechanism. |
| Arbitrary file read | Server-side file bytes can be retrieved. | Source inclusion, a usable credential or another executable path. |
| Local file inclusion (LFI) | A local file is processed by an inclusion mechanism. | Attacker-controlled executable content and an interpreter that evaluates it. |
| Remote file inclusion (RFI) | A remote resource is included. | Runtime configuration permits remote inclusion and evaluates its content. |
| Upload | A file is accepted or stored. | A handler, parser, inclusion path or privileged consumer processes it unsafely. |

An endpoint may use a raw file reader even if its parameter is named `page` or `include`. Confirm behavior instead of inferring it from the parameter name.

## Traversal and useful reads

**Kali / Bash:**

```bash
curl -i --path-as-is 'http://<HOSTNAME>/public/../../../../etc/passwd'
curl --get 'http://<HOSTNAME>/index.php' --data-urlencode 'page=../../../../etc/passwd'
```

`--path-as-is` prevents curl from normalizing path traversal. Vary depth and encoded forms only when the request's decoding/filtering behavior supports that hypothesis. Windows targets may accept backslashes, drive paths or mixed separators.

High-value read targets depend on the application and identity:

| Target | Why it matters |
|---|---|
| `/etc/passwd`, Windows hosts file | Confirm a file read and identify local context. |
| Application source and backups | Reveal credentials, routes, include logic and database queries. |
| Web/server configuration | Map document roots, virtual hosts, log locations and handlers. |
| User SSH private keys | Potential login using the matching account and passphrase. |
| Logs or session files | Potentially writable content for an inclusion chain. |

Extract an entire private key with correct line breaks before following the [SSH procedure](../01-Enumeration/File-Services.md#ssh--tcp-22). Do not assume the web process can read every user's home or system-protected file.

## PHP wrappers

Source disclosure through a filter:

```bash
curl --get 'http://<HOSTNAME>/index.php' --data-urlencode 'page=php://filter/convert.base64-encode/resource=config.php'
```

Decode only the Base64 portion of the response. The filter can expose PHP source when direct inclusion would execute it.

`data://text/plain;base64,<BASE64_CONTENT>` supplies inline content. When used in an include path, it needs compatible PHP configuration, including relevant URL-inclusion permissions. The source payload decodes to `<?php echo system($_GET["cmd"]); ?>`; its presence in a URL does not guarantee execution.

Other wrappers mentioned in the notes include `file://`, `http://`, `ftp://`, `php://`, `zlib://`, `glob://`, `phar://`, `ssh2://`, `rar://`, `ogg://` and `expect://`. They have different semantics and extension/configuration requirements. `expect://` is not installed by default; `phar://` is not a universal deserialization primitive on all PHP versions.

PHP filter-chain generation can sometimes turn a controllable include/filter path into executable content. Establish the wrapper behavior and available filters first; do not treat the generator as a generic file-read bypass.

## Log or session poisoning

**Prerequisites:** an include primitive evaluates the selected file, the application can read it, and an attacker controls bytes written into it.

1. Identify the actual log/session path from configuration or an error.
2. Send a recognizable marker in a field that is logged, such as User-Agent.
3. Include the file and confirm that the marker appears.
4. Replace the marker with a minimal interpreter proof and request the included file again.

Example PHP content for a controlled lab:

```php
<?php system($_GET['cmd']); ?>
```

Log escaping, truncation, permissions and interpreter configuration can prevent execution. Source paths such as `/var/log/apache2/access.log` and `C:\xampp\apache\logs\access.log` are candidates, not guaranteed locations. Request `cmd=id` or `cmd=whoami` to confirm the actual OS identity.

## File upload

Record accepted extension, multipart part MIME type, file signature, final stored name/path, retrieval URL and server processing. Change one property at a time.

| Check | Examples | Required condition |
|---|---|---|
| Extension filtering | `.php`, `.phtml`, `.php3`, `.php5`, mixed case | Server maps the accepted suffix to an executable handler. |
| Other runtime suffixes | `.asp`, `.aspx`, `.pl`, `.cgi`, `.jsp`, `.jspx`, `.cfm` | Corresponding runtime/handler is installed and enabled. |
| Double extension | `file.php.jpg` | Handler or unsafe name validation interprets the name as executable. |
| Multipart MIME change | `Content-Type: image/jpeg` | Validation trusts the claimed type rather than actual parsing. |
| Magic bytes/polyglot | `GIF89a;` followed by PHP | Validation accepts it and a later handler evaluates the PHP. |
| Metadata content | PHP in a JPEG comment | Uploaded bytes survive processing and later reach an interpreter. |
| Path traversal in filename | A path outside the upload directory | Server fails to constrain the destination. |
| Writable `.htaccess` | Add a handler for a chosen suffix | Apache allows per-directory overrides and honors that handler. |

Image metadata example on Kali:

```bash
exiftool -Comment='<?php system($_GET["cmd"]); ?>' image.jpg
```

Uploading the resulting image does not itself execute it. Image rewriting can remove the metadata.

Historical bypasses include trailing dots/spaces, IIS alternate data stream names, `file.asp;.jpg` and NUL-byte truncation. They depend on old or specific parsing stacks; `%00` is not a general modern PHP bypass. Keep them as version-specific hypotheses, not the first assumption.

## Verify and record

Fetch the final stored file and compare bytes if possible. A 200 response from the upload form may still mean rejection. If a script is downloaded as text, the path lacks a matching execution handler. Record uploaded names and restore any overwritten configuration after practice.

## Additional inclusion and IIS consumers

`php://input` reads the request body in a compatible inclusion context; PHP in a POST body is not executable on an ordinary endpoint. The source PDF+PHP example needs later PHP inclusion, not merely PDF parsing. Traversal through an `Accept` header requires a particular affected framework/parser; it is not a generic Django/Rails/Node behavior.

The source's “ASPX web shell” snippet is actually a **web.config handler mapping for classic ASP**. It requires IIS configuration override permission, installed/enabled ASP/ISAPI handling and compatible request filtering. A true `.aspx` page needs ASP.NET. Verify that IIS accepted the configuration and evaluated a marker, rather than counting a successful upload as execution.
