---
title: management-wants-a-word
tags:
  - dfir
  - windows
  - forensics
  - tryhackme
  - hacker-holidays
difficulty: hard
platform: TryHackMe
---

# Management Wants a Word

## In plain terms
Someone left behind a triage image of a Windows machine, and the flag is hidden behind a wall of encryption. The saved Chrome password on that machine is the key to a secret encrypted vault,  but that password is itself locked behind the user's Windows login. So the whole room is a chain: crack the Windows password, use it to peel back layer after layer until you reach the Chrome-saved password, then use that password to open a hidden vault where the flag is buried inside an image, inside a PDF.

## TL;DR
Starting from a KAPE triage bundle, recover the account password, walk the full DPAPI chain to decrypt a Chrome-saved password, then use that password to mount a VeraCrypt container and extract the flag from an image embedded in a PDF.

## The path
KAPE artifacts → NT hash → cracked password → DPAPI master key → Chrome AES key → saved password → VeraCrypt container → flag.

## 1. Recon — the artifacts
The evidence is a KAPE triage bundle (targeted Windows artifact collection). The pieces that matter live under the user's profile: the registry hives (`SAM`, `SYSTEM`), the DPAPI `Protect` folder, Chrome's `Local State` and `Login Data`, and a `backup` file in `Documents`.

## 2. Account password
See [[dpapi-chain]] Lock 1 — `secretsdump.py` on the offline hives yields the NT hash, cracked with John to plaintext: `minivera`.

## 3. The DPAPI chain
The core of the room. Full method in [[dpapi-chain]]:
- **Master key** — `dpapi.py masterkey` with the plaintext password + SID.
- **Chrome AES key** — `dpapi.py unprotect` on `Local State` with the master key.
- **Saved password** — AES-256-GCM decrypt of the `Login Data` blob → `Wh4t1sV3raD0inG0nTh1sH0st`.

That recovered password is the Chrome-saved one — and it's the key to the vault.

## 4. The VeraCrypt vault
The `backup` file in `Documents` is a VeraCrypt container. Mount it with the recovered password:
```bash
sudo veracrypt --text --mount backup /tmp/vera_vault --pim=0
```
> ⚠️ Gotcha: passing `--keyfiles=""` throws an assertion error (`Not enough data available`), omit it entirely, the default is already no keyfile.

Inside: a folder `secret_financial_documents` with two files:
- `important_invoice_byte_lotus.pdf`
- `transactions_q3.csv` (a hint: "image asset correction" — steganographic nudge)

## 5. The flag
The flag lives in a PNG embedded inside the PDF. Extract embedded images with:
```bash
pdfimages -all important_invoice_byte_lotus.pdf out
```
Flag recovered: `THM{re██████████████d}` ✓

## 🧠 Insight
The whole room hinges on one fact: the DPAPI master key for a LOCAL account is locked with the plaintext password, not the NT hash — so cracking the hash to plaintext isn't optional, it's the hinge the entire chain swings on. See [[dpapi-chain]] for the why.

## 🔗 Techniques used
- [[techniques/dpapi-chain|dpapi-chain]]