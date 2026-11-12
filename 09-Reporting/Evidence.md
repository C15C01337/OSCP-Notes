# Evidence, reporting and exam preparation

[Home](../README.md) · [Workflow](../00-Start/Workflow.md) · [Machine template](Machine-Template.md) · [Report template](Report-Template.md) · [Inventories](Inventories.md)

## Before the attempt

The supplied collections contain conflicting snapshots of scoring, AD partial credit, reverts, bonuses, duration and tool restrictions. This handbook does not establish current exam policy. Use the [official exam guide](https://help.offsec.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide), [official FAQ](https://help.offsec.com/hc/en-us/articles/4412170923924-OSCP-Exam-FAQ), exam email and control panel. These links were supplied locally and were not opened during consolidation.

- [ ] Verify permitted tools **and features**, including Metasploit/Meterpreter, pivoting, automated exploitation/scanning, spoofing, commercial services and AI during both practical and reporting phases.
- [ ] Record the applicable objectives, proof/shell requirements, time windows, submission deadlines, naming, archive format/size and verification procedure.
- [ ] Test VPN, keyboard, terminal recording, screenshots, VM recovery and local offline notes/tools.
- [ ] Rehearse a report export, inspect the PDF and test the required archive process.
- [ ] Plan meals, rest, breaks and power/network contingencies.

The supplied newer guide specifically excludes SQLMap, mass vulnerability scanners, spoofing and AI chatbots, and restricts Metasploit/Meterpreter use. Keep those technical references for permitted practice; do not infer exam authorization from inclusion in this handbook. Track any limited-use tool allocation before the first attempt, including failed runs/checks if your applicable guide counts them.

## Capture at each milestone

Record execution host/shell, timestamp/timezone, target address/hostname, account and effective privilege. Save the exact command or HTTP request, relevant output, explanation and next transition. A valid password, working shell, required privilege, proof capture and panel submission are separate milestones.

The supplied guide's proof examples show the original proof file contents in an interactive target shell together with target network information (`ipconfig`, `ip addr` or `ifconfig`). Follow the exact accepted capture method and privilege rules in your guide; a web response or copied value alone may be insufficient. Keep screenshots legible, uncropped where required and matched to their machine and milestone. Submit required values within the practical window and separately track screenshot/report inclusion.

## Private recording workflow

**Kali / Bash:**

```bash
umask 077
mkdir -p lab-session/{scans,evidence,files,notes}
script -q -f lab-session/terminal.log
```

Work in the recorded child shell, then `exit` to finish. `script` records terminal output/control sequences; it is not a substitute for command context, screenshots or a complete evidence narrative. Logs may contain secrets. The source's automatic `.zshrc` wrapper is optional and can create confusing nested sessions; an explicit recorder is easier to identify and stop.

Use stable filenames such as `host01-03-privilege.png` and relative image links in private machine notes. Keep credential data and evidence outside public repositories. A local notebook is not automatically encrypted.

## Reproducible findings

For each host, explain the observation that led to the path, the prerequisite, the minimal test and the verified result. Include network forwards, file locations, tool versions and resulting identities at every transition. Avoid pasting a long scan without explaining which result mattered.

For an unchanged public exploit, retain its source reference. For a modified exploit, retain original URL/revision/hash, final code, invocation, payload-generation command, changed lines and rationale. Buffer-overflow practice reports also need derived offset, bad-byte, return-address and debugger evidence; see [the adaptation chapter](../08-Reference/Buffer-Overflows.md).

Describe impact and a concrete remediation tied to the cause: excessive service/ACL rights, exposed credentials, unsafe input handling, missing patch or insecure deployment. Assign severity from the demonstrated impact and context; do not label every template finding Critical automatically. Report only achieved results supported by evidence.

## Export and submission audit

- [ ] Every required target/objective has a complete reproduction and evidence reference.
- [ ] Proof values, identities and addresses agree across notes, screenshots, report and panel record.
- [ ] Commands are selectable text, images legible and captions meaningful.
- [ ] Modified code and dependencies are included according to the guide.
- [ ] Empty placeholders, sample claims, irrelevant sections and copied personal narrative are removed.
- [ ] PDF page breaks, code wrapping, table layout and links are visually inspected.
- [ ] Correct file/archive name, format, contents, size and upload/checksum procedure verified.
- [ ] Upload buffer remains; confirmation is recorded before the deadline.

SysReptor and OffSec-Reporting are supplied renderer options. The bundled setup script installs packages/Docker, starts services, fetches remote installers and imports designs; it is not an offline Markdown converter and was not run here. Rehearse the installed renderer beforehand. If a design watermark obscures code, adjust the design and inspect the export; do not assume deleting one HTML element fixes every layout.

## Track changes and cleanup

Record created files/accounts/services, changed passwords/ACLs/configuration, routes/listeners, snapshots and temporary keys. Restore the **recorded original** and verify it. Do not claim cleanup merely because a generic remove command printed success. Preserve evidence before removing a deliberately created proof artifact; follow the environment's instructions rather than copying the old report's blanket “all accounts removed” statement.
