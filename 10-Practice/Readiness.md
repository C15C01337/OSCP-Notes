# Practice and readiness

[Home](../README.md) · [Workflow](../00-Start/Workflow.md) · [Evidence guide](../09-Reporting/Evidence.md) · [Progress inventory](../09-Reporting/Inventories.md)

## Use the remaining preparation time

Practice the entire loop: enumerate an unfamiliar host, justify the chosen hypothesis, establish access, verify an escalation prerequisite, capture evidence and reconstruct the report. Use timed blocks and breaks; a fixed study calendar or number of solved machines is not proof of readiness.

- Explain a failed route, DNS query, authentication and authorization check separately.
- Reproduce a manual web/SQL test from a saved request without a scanner's conclusion.
- Verify Linux sudo/SUID/capability and Windows service/token paths by their actual permissions and trigger.
- Explain AD keys/tickets, one object-permission path and each lateral identity transition.
- Reach an internal service and test a callback through the pivot.
- Adapt a local exploit while preserving its original, diff and invocation.
- Produce a complete report from saved evidence, including cleanup records.

After a hint, identify the missed observation, close the walkthrough and reattempt from a fresh state later. Track time to first useful finding, environment failures, missing evidence and hint dependence. Use spaced reattempts, active recall and a short explanation in your own words (the source's Feynman/Leitner ideas) instead of rereading the same command list.

Foundations to revisit when needed: Linux/Windows users, groups, UID/token/UAC and filesystems; TCP/UDP, DNS and routing; HTTP/SQL; Bash, Python, Ruby and PowerShell; basic assembly/calling conventions. Confidentiality, integrity and availability describe impact; a vulnerability is a weakness, a threat is a potential cause of harm and risk depends on context/likelihood and consequence.

Small scripting drills from the roadmap: enumerate an authorized address range with explicit timeout/error handling; wrap a selected Nmap scan while preserving output; extract hyperlinks from a local document. Know where the script runs and what it sends.

## Supplied practice catalogues

The following names/links were supplied locally. Availability, current difficulty and exam similarity were **not checked**. The sources contain questionable labels and duplicate/mismatched VulnHub URLs, so ratings and asserted exploit categories have been removed. These are optional catalogue leads, not a recommendation to spend money or a prediction of exam targets.

### HackTheBox

[Lame](https://www.hackthebox.com/machines/Lame) · [Legacy](https://www.hackthebox.com/machines/Legacy) · [Brainfuck](https://www.hackthebox.com/machines/Brainfuck) · [Blue](https://www.hackthebox.com/machines/Blue) · [Shocker](https://www.hackthebox.com/machines/Shocker) · [Devel](https://www.hackthebox.com/machines/Devel) · [Bashed](https://www.hackthebox.com/machines/Bashed) · [Optimum](https://www.hackthebox.com/machines/Optimum) · [Nibbles](https://www.hackthebox.com/machines/Nibbles) · [Bastard](https://www.hackthebox.com/machines/Bastard) · [Beep](https://www.hackthebox.com/machines/Beep) · [Granny](https://www.hackthebox.com/machines/Granny) · [Grandpa](https://www.hackthebox.com/machines/Grandpa) · [Cronos](https://www.hackthebox.com/machines/Cronos) · [Nineveh](https://www.hackthebox.com/machines/Nineveh) · [Arctic](https://www.hackthebox.com/machines/Arctic) · [Solidstate](https://www.hackthebox.com/machines/Solidstate) · [Node](https://www.hackthebox.com/machines/Node) · [Jerry](https://www.hackthebox.com/machines/Jerry) · [Valentine](https://www.hackthebox.com/machines/Valentine) · [Poison](https://www.hackthebox.com/machines/Poison) · [Conceal](https://www.hackthebox.com/machines/Conceal) · [Sunday](https://www.hackthebox.com/machines/Sunday) · [BankRobber](https://www.hackthebox.com/machines/BankRobber) · [Irked](https://www.hackthebox.com/machines/Irked) · [SecNotes](https://www.hackthebox.com/machines/SecNotes) · [Forest](https://www.hackthebox.com/machines/Forest) · [SwagShop](https://www.hackthebox.com/machines/SwagShop) · [Bastion](https://www.hackthebox.com/machines/Bastion) · [Active](https://www.hackthebox.com/machines/Active) · [Nest](https://www.hackthebox.com/machines/Nest) · [Remote](https://www.hackthebox.com/machines/Remote) · [Sauna](https://www.hackthebox.com/machines/Sauna) · [Mirai](https://www.hackthebox.com/machines/Mirai) · [Popcorn](https://www.hackthebox.com/machines/Popcorn) · [Fuse](https://www.hackthebox.com/machines/Fuse) · [Querier](https://www.hackthebox.com/machines/Querier) · [Quick](https://www.hackthebox.com/machines/Quick) · [Blocky](https://www.hackthebox.com/machines/Blocky) · [Magic](https://www.hackthebox.com/machines/Magic) · [Tabby](https://www.hackthebox.com/machines/Tabby) · [OpenAdmin](https://www.hackthebox.com/machines/OpenAdmin) · [Admirer](https://www.hackthebox.com/machines/Admirer) · [Blunder](https://www.hackthebox.com/machines/Blunder) · [Doctor](https://www.hackthebox.com/machines/Doctor) · [Ippsec Youtube](https://youtube.com/playlist?list=PLla1QtqibYnFvysbaj2SlqShgZTJjb7Er&si=iCOx914bEbym_UMN) · [Ippsec Active Directory](https://www.youtube.com/watch?v=jUc1J31DNdw&list=PLIU37Kps3dWApq77TnChQ52clmau8DfZA) · [0xdf Writeups](https://0xdf.gitlab.io/)

### TryHackMe

[Blue](https://tryhackme.com/room/blue) · [Ignite](https://tryhackme.com/room/ignite) · [Kenobi](https://tryhackme.com/room/kenobi) · [Agent Sudo](https://tryhackme.com/room/agentsudoctf) · [Vulnversity](https://tryhackme.com/room/vulnversity) · [Ice](https://tryhackme.com/room/ice) · [Brainstorm](https://tryhackme.com/room/brainstorm) · [HackPark](https://tryhackme.com/room/hackpark) · [Skynet](https://tryhackme.com/room/skynet) · [Steel Mountain](https://tryhackme.com/room/steelmountain) · [Relevant](https://tryhackme.com/room/relevant) · [Shrek](https://tryhackme.com/room/shrek) · [Overpass 2 - Hacked](https://tryhackme.com/room/overpass2hacked) · [Alfred](https://tryhackme.com/room/alfred) · [Corp](https://tryhackme.com/room/corp) · [Thompson](https://tryhackme.com/room/thompson) · [Wreath](https://tryhackme.com/room/wreath) · [Brooklyn Nine Nine](https://tryhackme.com/room/brooklynninenine) · [Vault](https://tryhackme.com/room/vault) · [HackerSploit YouTube](https://www.youtube.com/watch?v=a5iQtVBYec4&list=PLBf0hzazHTGMaLYlIX25wdXboA6XwiZSw) · [John Hammond YouTube](https://www.youtube.com/watch?v=xl2Xx5YOKcI&list=PL1H1sBF1VAKUOm3WyiZ-m2Oqwku4Xp6if)

### VulnHub

[Bob](https://www.vulnhub.com/entry/bob-1,231/) · [Brainpan](https://www.vulnhub.com/entry/brainpan-1,51/) · [GoldenEye](https://www.vulnhub.com/entry/goldeneye-1,240/) · [NullByte](https://www.vulnhub.com/entry/nullbyte-1,232/) · [Pluck](https://www.vulnhub.com/entry/pluck-1,229/) · [Sedna](https://www.vulnhub.com/entry/sedna-1,249/) · [Retired](https://www.vulnhub.com/entry/retired-1,150/) · [Colt](https://www.vulnhub.com/entry/colt-1,48/) · [Tibra](https://www.vulnhub.com/entry/tibra-1,205/) · [MicroVuln](https://www.vulnhub.com/entry/microvuln-1,157/) · [Razorback](https://www.vulnhub.com/entry/razorback-1,194/) · [Rocky](https://www.vulnhub.com/entry/rocky-1,168/) · [Vulnix](https://www.vulnhub.com/entry/vulnix-1,229/) · [Surveillance](https://www.vulnhub.com/entry/surveillance-1,102/) · [Maverick](https://www.vulnhub.com/entry/maverick-1,109/) · [HackerSploit YouTube](https://www.youtube.com/watch?v=i-dQwejj518&list=PLBf0hzazHTGMcijE6NgwyjfbR33D8GJVe) · [InfoSec](https://www.youtube.com/watch?v=lrTaaBWsQlQ&list=PLxTwjzMO9Zf6BLXcw8Hxjmt-sDHMacQxU)

### Additional names from the image-only lists

**Proving Grounds, source Windows column:** Nickel, Slort, Authby, Jacko, MeatHead, UT99, MedJed, Algernon, Billyboss, Kevin, Metallus, Hutch, Heist, Vault, Shenzi, DVR4, Craft, Hepet, Squid.

**Source Linux column:** ClamAV, Wombo, Payday, Fail, Nibbles, Banzai, Hunit, Dibble, Zino, Hetemit, Peppo, Postfish, Malbec, Sybaris, Hawat, ZenPhoto, Readys, Nukem, Sorcerer, Walla, Pelican, Snookums, Exfiltrated. Source labels Postfish as involving phishing and Vault as involving MITM; those are training contexts, not exam permission.

**Source harder column:** Bratarina, Internal, Clyde, Vector, Shifty, XposedAPI, HelpDesk, Twiggy, Pebbles, Panic, CookieCutter, Tico.

**Additional VulnHub names visible in the source image:** Kioptrix 1/1.1/1.2/1.3/2014, FristiLeaks, Stapler, VulnOS, SickOs, HackLAB Vulnix, dev/random Scream, SkyTower, Mr-Robot, pWnLab, Lin.Security, Temple of Doom, Pinky's Palace, Tr0ll, Wintermute, Web Developer, SolidState, Hackme, Escalate_Linux, DC 6/DC 9, Digitalworld.local (Bravery, Development, Mercy, JOY, FALL), Prime, Misdirection, Sar, Djinn, EVM, DerpNStink, RickdiculouslyEasy, Breach, NullByte, Bob, Toppo, W34kn3ss, GoldenEye, Infosec Prep OSCP, LemonSqueezy, Brainpan, Lord of the Root, Tiki, Healthcare, Photographer, Glasgow, DevGuru, IMF, Tommy Boy, Billy Madison, Wallaby's Nightmare, Moria, BSides Vancouver, DEFCON Toronto Galahad, SpyderSec, Vulnerable Docker, Node, Readme, HackNos, Metasploitable 3, Election, Hacker Kid. The small screenshot clips some labels; names above are recognition aids, not validated links.

### Buffer-overflow practice

Supplied targets/resources include Vulnserver TRUN, SLmail, FreeFloat FTP, MiniShare, Brainpan, Ability, PCMan FTP, Crossfire, Aviosoft Digital TV Player and HexChat. Use matching deliberately vulnerable builds in a disposable lab; see [buffer overflows](../08-Reference/Buffer-Overflows.md). These are historical exercises, not current exam claims.

### Other training leads

The collections name Metasploitable2, DVWA, OWASP WebGoat, Mutillidae, BadStore, PentesterLab, Root-Me, OverTheWire, Exploit Education, Cryptopals, Microcorruption, pwnable.kr, PicoCTF, PortSwigger Web Security Academy, Hacker101, CMD Challenge and other CTF catalogues. Their external challenge bodies were not downloaded. Use an exercise that fills an identified skill gap rather than collecting more resource lists.
