---
name: cracking
description: How to crack password hashes (NTLM, NetNTLMv1/v2, Kerberos, bcrypt, md5crypt, etc.) with hashcat/john. Use when a sub-agent has captured hashes (ASREProast, Kerberoast, /etc/shadow, SAM) and needs plaintext.
---

# Hash cracking

Reference: `kb-internal` `cheatsheets/hash-cracking.md` and
`active-directory/hash-capture.md`, `kb-recipes`
`ad/movement/credentials/cracking.md`.

## Identify the hash type first

- `$6$...` = sha512crypt (Linux `/etc/shadow`), `$1$` = md5crypt,
  `$y$` = yescrypt, `$2b$`/`$2a$`/`$2y$` = bcrypt.
- `$krb5tgs$23$...` = Kerberoast TGS (hashcat 13100, john `krb5tgs`),
  `$krb5asrep$23$...` = ASREProast (hashcat 18200, john `krb5asrep`).
- `aad3b435b51404eeaad3b435b51404ee:<ntlm>` = NTLM (hashcat 1000).
- NetNTLMv2 challenge/response (hashcat 5600), NetNTLMv1 (5500), DCC2 (2100).
- Use `hashid` or `hashcat --example-hashes` to confirm.

## Commands

```bash
# identify
hashid <file>   # or: hashcat --example-hashes | grep -i <name>

# hashcat (GPU)
hashcat -m <mode> -a 0 <hashes> /usr/share/wordlists/rockyou.txt
hashcat -m <mode> -a 0 <hashes> rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# john
john --format=<fmt> --wordlist=/usr/share/wordlists/rockyou.txt <hashes>
john --show <hashes>
```

- Common modes: 0=MD5, 1000=NTLM, 5500/5600=NetNTLMv1/v2, 13100=TGS,
  18200=ASREP, 1800=sha512crypt, 500=md5crypt, 3200=bcrypt, 2100=DCC2.
- Wordlists in Exegol: `/usr/share/wordlists/rockyou.txt` (and seclists).
- **No GPU in Exegol docker** → hashcat may refuse or be slow. Fall back to
  `john` (CPU) on error, or use `hashcat --force` for quick CPU runs only.

## Workflow

1. Save each hash set to a file under
   `/workspace/engagements/<box>/loot/`.
2. Run cracking in a **sibling pane** of your own tab (Herdr `pane split` +
   `pane run`) so it proceeds in parallel with other work.
3. **Harvest the result** — do not abandon the pane:
   - `herdr pane wait-output <pane> --match "Recovered" --timeout 600000`
     (hashcat prints `Recovered: N/1 hashes`) or `--match "Session completed"`
     / `--match "Exhausted"`.
   - Then read the result: `herdr pane read <pane> --source recent-unwrapped`
     and confirm with `hashcat -m <mode> --show <hashes>` (or `john --show`).
   - If the wait times out, re-check the pane; if still running, wait again.
4. Try fast attacks first (rockyou + best64), then rules, then masks targeted
   at the box theme/username (`--user`, `kw1`, etc.) if needed.
5. When cracked, write `user:password` to `notes.md` immediately.

Report the mode, wordlist, and result (cracked/not-cracked).
