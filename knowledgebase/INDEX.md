# Knowledgebase Index

Compact map of the pentest knowledgebase under `/workspace/knowledgebase/`.
**Read this file first**, then jump to the specific file you need. Do not grep
blindly across 954 files.

Repos:
- `PayloadsAllTheThings/` (PAT) — web/API exploit payloads.
- `InternalAllTheThings/` (IATT) — internal/AD + post-exploitation.
- `HackerRecipes/` (HR) — theory + tool walkthroughs (AD, web, infra).

## Network / service recon

- Host discovery + port scanning: HR `infra/recon/hosts-discovery.md`, `infra/recon/port-scanning.md`.
- Port→protocol table: HR `infra/protocols/index.md`.
- L2/L3 discovery (nmap/masscan/ARP/DNS/MITM): IATT `cheatsheets/network-discovery.md`.
- AD-specific recon (DNS, DHCP, NetBIOS, MS-RPC, LDAP, enum4linux, responder, password policy, BloodHound): HR `ad/recon/`.

## Web recon

- Tech fingerprint, CMS, dir fuzz, vhost, WAF, crawling: HR `web/recon/` (`web-technologies.md`, `cms.md`, `directory-fuzzing.md`, `virtual-host-fuzzing.md`, `domains-enumeration.md`, `waf-fingerprinting.md`, `known-vulnerabilities.md`, `site-crawling.md`).
- Bug-hunting recon workflow (passive/active, hidden params): IATT `methodology/bug-hunting-methodology.md`.
- Vhost + hidden params + exposed `.git`/`.svn`: PAT `Virtual Hosts`, `Hidden Parameters`, `Insecure Source Code Management`.
- Default creds: HR `web/config/default-credentials.md`.

## Web exploitation (payloads) — PAT

- Injection: `SQL Injection` (split into per-DBMS files), `Command Injection`, `LDAP Injection`, `NoSQL Injection`, `GraphQL Injection`, `XPATH Injection`, `XSLT Injection`, `Server Side Include Injection`, `Server Side Template Injection` (per-engine), `LaTeX Injection`.
- File access: `File Inclusion`, `Directory Traversal`, `Upload Insecure Files`, `Zip Slip`, `Insecure Source Code Management`.
- Server-side request: `Server Side Request Forgery`, `XXE Injection`.
- Auth/session: `JSON Web Token`, `OAuth Misconfiguration`, `SAML Injection`, `Type Juggling`, `Account Takeover`, `Open Redirect`.
- Serialization/objects: `Insecure Deserialization` (per-language), `Prototype Pollution`, `Mass Assignment`, `Insecure Direct Object References`, `External Variable Modification`, `DOM Clobbering`.
- Misc: `XSS Injection`, `CRLF Injection`, `HTTP Parameter Pollution`, `Request Smuggling`, `Web Cache Deception`, `Race Condition`, `Insecure Management Interface`, `Web Sockets`, `Encoding Transformations`.
- CVE PoCs (EternalBlue, Struts2, Log4Shell, Shellshock…): PAT `CVE Exploits`.
- Theory + tool commands for the same vulns: HR `web/inputs/` (`sqli.md`, `ssti.md`, `ssrf/index.md`, `xxe-injection/index.md`, `file-inclusion/index.md` + `lfi-to-rce/`, `jwt.md`, `insecure-deserialization.md`, `unrestricted-file-upload.md`), HR `web/config/` (`http-request-smuggling/index.md`, `http-methods.md`).

## Linux privesc

- Authoritative playbook: IATT `redteam/escalation/linux-privilege-escalation.md`.
- Container escape: IATT `containers/docker.md`, `containers/kubernetes.md`.
- Thinner recipes: HR `infra/privilege-escalation/unix/` (`sudo.md`, `suid-sgid-binaries.md`, `capabilities.md`, `living-off-the-land.md`, `network-secrets.md`).
- Restricted-shell breakout: IATT `cheatsheets/escape-breakout.md`.

## Windows local privesc

- Authoritative playbook: IATT `redteam/escalation/windows-privilege-escalation.md`.
- AMSI bypass / defenses: IATT `redteam/evasion/windows-amsi-bypass.md`, `windows-defenses.md`.
- Mimikatz ref: IATT `cheatsheets/mimikatz-cheatsheet.md`.
- Thinner recipes: HR `infra/privilege-escalation/windows/` (`weak-service-permissions.md` is the fleshed-out one).

## Active Directory enumeration

- Enumeration + BloodHound: IATT `active-directory/ad-adds-enumerate.md`, `ad-adds-groups.md`.
- Roasting: IATT `active-directory/ad-roasting-asrep.md`, `ad-roasting-kerberoasting.md`, `ad-roasting-timeroasting.md`.
- Spraying / policy: IATT `active-directory/pwd-spraying.md`.
- Shares: IATT `active-directory/internal-shares.md`.
- LAPS/gMSA: IATT `active-directory/pwd-read-laps.md`, `pwd-read-gmsa.md`, `pwd-read-dmsa.md`.
- Recon theory: HR `ad/recon/` (`ldap.md`, `ms-rpc.md`, `bloodhound/index.md`).

## Active Directory exploitation / lateral movement

- NTLM relay + coercion: IATT `active-directory/internal-relay-ntlm.md`, `internal-relay-coerce.md`, `internal-relay-kerberos.md`.
- Pass-the-hash/key/overpass: IATT `active-directory/hash-pass-the-hash.md`, `hash-pass-the-key.md`, `hash-over-pass-the-hash.md`.
- Kerberos tickets (golden/silver/diamond/sapphire): IATT `active-directory/kerberos-tickets.md`.
- Delegation: IATT `active-directory/kerberos-delegation-constrained.md`, `kerberos-delegation-rbcd.md`, `kerberos-delegation-unconstrained.md`, `kerberos-bronze-bit.md`, `kerberos-s4u.md`.
- ACL/ACE abuse: IATT `active-directory/ad-adds-acl-ace.md`.
- ADCS: IATT `active-directory/ad-adcs-certificate-services.md`, `ad-adcs-esc01.md` … `ad-adcs-esc15.md`, `ad-adcs-golden-certificate.md`.
- CVE walkthroughs: IATT `active-directory/CVE/` (ZeroLogon, NoPAC, PrintNightmare, PrivExchange, MS14-068).
- Trusts: IATT `active-directory/trust-relationship.md`, `trust-ticket.md`, `trust-sid-hijacking.md`.
- Full theory + chains: HR `ad/movement/` (`index.md` is the master checklist; `ntlm/`, `kerberos/`, `dacl/`, `adcs/`, `mitm-and-coerced-authentications/`, `exchange-services/`, `print-spooler-service/`, `sccm-mecm/`, `trusts/`).

## Persistence / shells

- Reverse/bind shells: IATT `cheatsheets/shell-reverse-cheatsheet.md`, `shell-bind-cheatsheet.md`.
- Host persistence: IATT `redteam/persistence/linux-persistence.md`, `windows-persistence.md`, `rdp-persistence.md`.
- AD persistence: HR `ad/persistence/` (adminsdholder, dcshadow, skeleton-key, golden/silver tickets, sid-history).
- Download/execute LOLBin: IATT `redteam/access/windows-download-execute.md`.

## Pivoting

- Techniques: IATT `redteam/pivoting/network-pivoting-techniques.md`.
- Tools (chisel/ligolo/sshuttle/gost…): IATT `redteam/pivoting/network-pivoting-tools.md`.
- Recipes: HR `infra/pivoting/port-forwarding.md`, `socks-proxy.md`.

## Hash cracking

- Hashcat/john usage: IATT `cheatsheets/hash-cracking.md`.
- Hash capture + cracking: IATT `active-directory/hash-capture.md`.
- Cracking theory: HR `ad/movement/credentials/cracking.md`.

## MSSQL

- RCE (xp_cmdshell/CLR/OLE): IATT `databases/mssql-command-execution.md`.
- Linked servers: IATT `databases/mssql-linked-database.md`.
- Enum/creds/impersonation: IATT `databases/mssql-enumeration.md`, `mssql-credentials.md`, `mssql-audit-checks.md`.

## C2 / Metasploit

- Metasploit: IATT `command-control/metasploit.md`.
- Cobalt Strike: IATT `command-control/cobalt-strike.md`.

## Out of scope for most HTB machines

- Cloud (AWS/Azure), DevOps/CI-CD, mobile, wireless, binary exploitation:
  present but enterprise/niche-oriented; consult only if the box clearly needs it.
