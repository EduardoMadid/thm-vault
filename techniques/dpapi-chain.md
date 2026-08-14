---
title: dpapi-chain
tags: [dfir, windows, credentials, dpapi]
---

# DPAPI credential chain

## In plain terms
Chrome saves your passwords, but it doesn't store them as plain text — it locks them in a safe. The catch: the key to that safe is itself locked in another safe, and that one is locked by your Windows login password. So to read a saved password from a powered-off machine, you open the safes in order: crack the Windows password first, use it to open the master key, use that to open Chrome's key, and finally use Chrome's key to read the saved password. Each safe you open hands you the key to the next. The recovered password then unlocked a hidden encrypted container where the flag was buried in an image inside a PDF.

## TL;DR
The Windows DPAPI (Data Protection API) credential chain links user passwords, masterkeys, and historical credentials (CREDHIST) to decrypt protected app data, browser cookies, and stored secrets. Recovering a Chrome-saved password offline means walking a chain of "locks", where each unlocked artifact becomes the key to the next.

## What DPAPI is
A built-in Windows service. Programs call `CryptProtectData` / `CryptUnprotectData` to encrypt/decrypt secrets without managing their own key. The key is derived from the user's login password — it's never written to disk in usable form.

Windows doesn't use the password directly on each secret: it creates one **master key** per user (stored encrypted in `AppData\Roaming\Microsoft\Protect\<SID>\<GUID>`) and uses it to protect everything. Password unlocks the master key → master key unlocks the secrets (blobs). The `Preferred` file in that folder points to which GUID is the current master key.

## The chain of locks
| Lock | Source artifact         | Tool                      | Yields                        |
|------|-------------------------|---------------------------|-------------------------------|
| 1    | `SAM` + `SYSTEM`        | `secretsdump.py` + `john` | user login password           |
| 2    | `Protect\<SID>\<GUID>`  | `dpapi.py masterkey`      | DPAPI master key (hex)        |
| 3    | `Local State` (Chrome)  | `dpapi.py unprotect`      | Chrome AES-256 key (32 B)     |
| 4    | `Login Data` (Chrome)   | Python + AES-256-GCM      | saved password                |

### Lock 1 — NT hash from the hives
`SAM` holds the hashes, encrypted inside itself; the key (bootKey/syskey) lives in `SYSTEM` — that's why both are needed together.
```bash
secretsdump.py -sam SAM -system SYSTEM LOCAL
john --format=NT --wordlist=rockyou.txt vera.hash
```
`LOCAL` = offline hives. John doesn't decrypt — it tests wordlist entries, hashing each until one matches.

### Lock 2 — unlock the master key
```bash
dpapi.py masterkey -file '.../Protect/<SID>/<GUID>' -sid '<SID>' -password '<password>'
```
The `-sid` goes into the derivation recipe alongside the password. Output: `Decrypted key: 0x<hex>`.

### Lock 3 — Chrome AES key (Local State)
`Local State` is JSON; the `os_crypt.encrypted_key` field is the AES key in base64, wrapped in a DPAPI blob with a `DPAPI` prefix (5 bytes that must be stripped).
```bash
dpapi.py unprotect -file /tmp/chrome_key.bin -key 0x<masterkey>
```
Output: 32 bytes = the Chrome AES key.

### Lock 4 — decrypt the saved password
`Login Data` is SQLite; the `password_value` is a blob shaped: `v10` prefix (3 bytes) + nonce (12 bytes) + ciphertext + GCM tag (16 bytes). Decrypt with AES-256-GCM (pycryptodome) using the key from lock 3.

## 🧠 Insight: why plaintext, not the NT hash?
For a **LOCAL** account, the DPAPI master key is locked with the **SHA1 of the plaintext password** (not the NT hash). That's why extracting the NT hash in lock 1 wasn't enough; it had to be cracked down to plaintext to feed lock 2. The NT hash and the SHA1 are *different* derivations of the same password, and DPAPI only accepts the one it uses.

Windows password -> master key -> Chrome key -> password

## Tools
- Impacket (`secretsdump.py`, `dpapi.py`) via pipx
- John the Ripper (`--format=NT`, rockyou/seclists)
- pycryptodome (AES-256-GCM)

## 🔗 Appears in
- [[rooms/management-wants-a-word|management-wants-a-word]]