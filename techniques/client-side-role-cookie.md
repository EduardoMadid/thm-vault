---
title: client-side-role-cookie
tags:
  - tech
  - web
  - access-control
  - authorization
  - cookies
  - broken-access-control
---

# Client-side role cookie (forgeable authorization)

## In plain terms
The website decided whether you were an admin by reading a value your own browser sends back to it. It is like a club that decides you are a VIP by reading a wristband you printed at home. The value was scrambled with a hash, but it was only the scrambled form of the word "true" or "false", so you scramble "true" yourself, wear that, and the door opens.

## TL;DR
When authorization is decided by a **client-controlled cookie** whose value is the hash of a small, guessable input (a boolean, a role name, a low integer), the trust boundary is broken: the client can forge any value. In this room a cookie `isITUser` held `md5("false")` for a normal user; replacing it with `md5("true")` unlocked the IT/Admin panel and the internal API.

## The mechanism
Authorization should be decided by state the server controls (a server-side session, or a token the server signed with a secret). Here it was decided by a cookie the browser owns and can edit freely.

The server-side check looked like this:
```php
if (($_COOKIE['isITUser'] ?? md5('false')) !== md5('true')) {
    die('Access denied');
}
```

The value space is **two options**. Recovering the input is a one-line offline test, no cracking rig needed:
```bash
echo -n 'true'  | md5sum   # b326b5062b2f0e69046810717534cb09
echo -n 'false' | md5sum   # 68934a3e9455fa72420237eb05902327
```

| Step | Artifact | Action | Result |
|------|----------|--------|--------|
| 1 | Cookie `isITUser` | Read value in DevTools → Storage | `md5("false")` (normal user) |
| 2 | `md5("true")` | Compute locally | `b326b5062b2f0e69046810717534cb09` |
| 3 | Cookie `isITUser` | Overwrite value, reload | IT/Admin panel unlocked |

Notice `HttpOnly` was not set on the cookie, which is what makes it trivially editable from the client side.
## Detection
- Cookies or hidden fields named like a role/flag: `isAdmin`, `isITUser`, `role`, `usertype`, `level`.
- Values that are fixed-length hex (32 = MD5, 40 = SHA1, 64 = SHA256) with no per-request variation.
- Test by hashing obvious inputs (`true`/`false`/`1`/`0`/`admin`/`user`) and comparing.

## Mitigation
- Never store the authorization decision client-side.
- Keep the role in the **server-side session**, keyed to the authenticated user.
- If a token must travel to the client, sign it (HMAC/JWT with a server secret) and verify the signature.

## Tools
- Browser DevTools (Storage / Cookies tab)
- `md5sum` / `echo -n ... | md5sum`
- `curl -b "cookie=value"` for scripted testing

## 🔗 Appears in
- [[rooms/support|support]]
