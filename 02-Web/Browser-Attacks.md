# XSS and browser-side workflows

[Home](../README.md) · [Web enumeration](Enumeration.md) · [Template injection](Template-Injection.md)

## Cross-site scripting

**Trigger:** untrusted input executes as script in another application's browser origin. Determine reflected/stored/DOM behavior and the output context before choosing a payload.

```html
<script>alert(1)</script>
```

This proof applies only to an HTML context where script tags execute. Attribute and JavaScript-string contexts need matching syntax. A stored value may be rendered only on an administrator's page; locate the consumer.

The source cookie-capture example uses `document.cookie`, not `document.cookies`. HttpOnly cookies are inaccessible to JavaScript. A cross-origin callback may also be blocked by policy or connectivity; lack of a callback does not independently prove absence of XSS.

## Actions in a logged-in origin

An XSS payload may perform same-origin requests with the viewer's session. The supplied WordPress example first reads an administrator form, extracts its CSRF nonce and then submits a user-creation request. This chain requires an administrator to execute the script, access to that form and the correct fields/nonce for that version.

Record the real request before constructing a script. A CSRF nonce does not prevent a script already executing in the same trusted origin from reading it. Creating a user is a state change; a successful HTTP response still needs confirmation of the resulting account and role.

Long character-code payloads in the source are encodings of the same script, not separate techniques. They also include copied debugger text that would break execution; retain readable JavaScript while developing and encode only when the actual output context requires it.

## HTML rendered by server-side tools

A file iframe such as `file:///etc/passwd` normally cannot read a server file through an ordinary user's browser. A server-side HTML-to-PDF renderer is a different consumer and may access files or URLs under its own identity and policy. Establish which program renders the HTML before labeling the behavior as file read or SSRF.

## Client-side delivery references

HTA files, Office/LibreOffice macros, Windows library files and shortcuts require a user or client application to open/process them under a compatible policy. See [client-side delivery](../03-Access/Client-Side-Delivery.md) for their consumers and prerequisites.
