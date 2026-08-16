---
title: os-command-injection
tags: [web, rce, command-injection, php]
---

# OS command injection (command-carrying parameter)

## In plain terms
One feature on the site ran a small system command and showed you the result — a clock that prints the server's time. The problem: the site sent the *actual command* to run inside a request, and the server ran whatever it was handed. So instead of letting it run the clock command, you hand it your own command, and the server obediently runs that too.

## TL;DR
A diagnostics feature passed the shell command in a client-controlled parameter (`sys`) and executed it server-side. Because the attacker controls the command string, appending a shell operator (`|`, `;`, `&&`, `$(...)`) runs arbitrary commands. This gave **RCE**, which read the database file and the target flag directly.

## The mechanism
The request literally shipped the command to the server:
```
sys=date +"%H:%M:%S"
```
The backend executed that string through a shell sink (in PHP: `system`, `exec`, `shell_exec`, `passthru`, `popen`, or backticks). When the whole string is attacker-controlled, you do not even need to "escape" anything — you chain your own command onto it:

```
sys=date|ls                          # confirm execution (directory listing)
sys=date|cat /var/www/db.php         # dump the app's user table
sys=date|cat /home/ubuntu/user.txt   # read the target file
```

| Operator | Effect |
|----------|--------|
| `\|` | pipe output of first command into second (runs both) |
| `;` | run second command after first, unconditionally |
| `&&` | run second only if first succeeds |
| `$(...)` / `` `...` `` | command substitution (inline execution) |

The value looked like `date +"%H:%M:%S"`, which is a design smell in itself: the app was shipping a raw shell command to the client and trusting it back.
## Detection
- Any feature that runs or reflects a system command: ping, traceroute, nslookup, "diagnostics", "system info", uptime, date.
- Parameters named `cmd`, `exec`, `sys`, `run`, `host`, `ping`, `ip`.
- Confirm with a benign chain first (`; id` or `| id`), never with something destructive.

## Mitigation
- Never build a shell command from client input.
- Map client actions to a fixed **allowlist** handled server-side (e.g. `action=uptime` → call a specific function, not a string).
- If exec is unavoidable, use array-form APIs that skip the shell and escape arguments (`escapeshellarg`).

## Escalation
Single-command RCE → typically upgrade to a **reverse shell** for interactive access. Here a direct `cat` of the target file was enough, so no shell was needed — always try the cheapest path that meets the objective first.

## Tools
- Browser DevTools (inspect the request/parameter)
- `curl --data-urlencode "sys=..."` for scripted payloads
- `nc` / reverse-shell one-liners if interactive access is required

## 🔗 Appears in
- [[rooms/support|support]]
