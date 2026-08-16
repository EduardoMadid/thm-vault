---
tags:
  - room
  - web
  - rce
  - broken-access-control
  - lfi
  - idor
  - command-injection
platform: TryHackMe
difficulty: medium
status: complete
title: support
---

# Support Operations Platform

> Pentest the internal Support Operations Platform, chain several trust-boundary weaknesses, and escalate to Remote Code Execution on the server.

**Flags**
- Admin flag: `THM{I_AM_ADMIN999}`
- `/home/ubuntu/user.txt`: `THM{GOT_THE_FLAG001}`

## In plain terms

Imagine a company's internal help desk website. Only employees should be able to log in, and only a manager (the "admin") should be able to do the powerful stuff. This writeup shows how a regular, low-level account was turned into full control of the server behind the website, one small trust mistake at a time.

The website made four big mistakes. First, it let anyone keep guessing passwords, so a common one was found by trying millions automatically. Second, it decided whether you were an admin by reading a value stored inside your own browser, which means you can simply edit it and claim to be an admin. Third, it let you read files off the server just by changing part of the web address, which leaked the real admin password stored in plain text. And fourth, one feature took whatever text you sent it and ran it as a command on the server itself, which is the worst of all: it means you can tell the server to do anything you want, including handing over the final secret file.

Each section below walks through one of those steps, with the exact commands and screenshots.

## 1. Reconnaissance

I started with an `nmap` scan using version detection and default scripts to gather as much information as possible.

![[assets/support1.png]]

Two ports are open: `22/tcp` (SSH) and `80/tcp` (HTTP). The web service is Apache 2.4.58 on Ubuntu. A few details already stand out:

- The response sets a `PHPSESSID` cookie, which tells us the backend is PHP with session handling.
- The `HttpOnly` flag is not set, so the session cookie is accessible from JavaScript.

SSH looks like the likely *exit* to a shell later; the entry point is clearly the web app on port 80.

Visiting the site returns a login form.

![[assets/support2.png]]

The form authenticates by **corporate email** and password over POST. I checked the page source but found nothing useful, so I moved on to directory enumeration. While that ran, I inspected the response headers with `curl`.

![[assets/support3.png]]

The `Expires: 1981` / `Cache-Control: no-store` combination is the signature of PHP's `session_start()`, which confirms the session backend. There is no `X-Powered-By` header (PHP version not disclosed here), and the session cookie has no `HttpOnly`, `Secure`, or `SameSite` attributes. That last point is a legitimate security finding on its own.

### Directory enumeration

![[assets/support4.png]]

`api.php` immediately caught my eye, along with `config.php`, `dashboard.php`, `includes/`, and `info.php`. The `api.php`, `dashboard.php`, and `logout.php` entries all return `302 → index.php`, meaning they sit behind authentication. `config.php` returns `200` with a zero-byte body, which is expected: it is pure PHP with no output, so nothing is echoed over HTTP.

`info.php` returns ~73 KB, which is the signature of an exposed `phpinfo()`.

![[assets/support5.png]]

From `phpinfo()` I confirmed the exact stack: **PHP 8.3.6**, Apache `mod_php`, host `tryhackme-2404` (Ubuntu 24.04), and the absolute filesystem paths. This is an information-disclosure finding on its own.


## 2. Authentication

With recon done, I moved to testing. The login page helpfully shows an email (`help@support.thm`) both as a placeholder and as the IT Operations contact, so I brute-forced its password with `ffuf` and `rockyou.txt`.

![[assets/support6.png]]

The password `snoopy` returned a `302` (redirect to the dashboard) while every failed attempt returned the login page, which the filter hid. I confirmed the credentials with `curl`.

![[assets/support7.png]]

The login succeeds and redirects to `dashboard.php`.

![[assets/support8.png]]

I am logged in as a **Helpdesk User**, a normal, non-admin account. The only visible feature is a "Select Theme" dropdown.


## 3. Privilege escalation: broken access control

I first tried tampering with the `?skin=default` parameter in the URL, but nothing obvious happened, so I opened the browser DevTools to look further.

![[assets/support9.png]]

In the Storage tab I found a cookie named `isITUser` with a 32-character hex value, which is the length of an MD5 hash. The name reads like a boolean question ("is IT user?"), and the value decodes to the hash of `false`.

![[assets/support10.png]]

Comparing hashes, I replaced the cookie with `md5("true")` to flip the boolean. After editing it in the Storage tab and reloading, I gained access to the admin view, which exposes the internal API.

![[assets/support11.png]]

![[assets/support12.png]]

Later, once I could read source files (see section 5), the logic behind this became clear. The `api.php` source checks the cookie directly and rejects anything that is not the hash of `true`:

![[assets/support18.png]]

```php
if (($_COOKIE['isITUser'] ?? md5('false')) !== md5('true')) { die('Access denied'); }
```

The same file also shows the `include('/var/www/db.php')` and the `$id = $_GET['id']` line that make the IDOR in the next section possible.

See [[techniques/client-side-role-cookie|client-side-role-cookie]] for the general technique.
## 4. Internal User API: IDOR

The API lets a user query their own profile at `/user/3`, returning JSON with `email`, `2FA`, and `admin` fields. My own profile shows `admin: false`, so the cookie only unlocked the API surface; real admin is decided by a separate per-user field.

Because the endpoint takes a numeric ID, I could query other users' profiles just by changing it. This is an **IDOR** (Insecure Direct Object Reference).

![[assets/support13.png]]

This let me enumerate other accounts, including the admin's email.


## 5. Local File Inclusion: config disclosure

After testing several things without luck, I went back to the `?skin=` parameter and tried a path traversal to `../config`. Viewing the page source revealed the contents of `config.php`.

![[assets/support14.png]]

The `php://filter` wrapper was blocked, but plain traversal worked and disclosed a hardcoded `$MASTER_PASSWORD = 'support@110'`.

I logged out and tried to sign in as admin with that password. It failed as written, but succeeded once I dropped the `@` and used `support110`. At this point I did not know why; the reason only became clear later, after I could read the real user table over RCE (section 6).

> **Why `support110` and not `support@110`:** the value in `config.php` is `support@110`, but the actual admin account (`specialadmin@support.thm`, seen later in `db.php`) has the password `support110`. So `config.php`'s `$MASTER_PASSWORD` was not the string the login actually validates against. Dropping the `@` happened to match the real admin password. The character was never "stripped" by the app; the two files simply stored different values, and login checks against the database.

Signing in with the admin credentials gave me administrator access and the first flag on the homepage.

![[assets/support15.png]]

**Admin flag:** `THM{I_AM_ADMIN999}`

Full breakdown in [[techniques/path-traversal-source-disclosure|path-traversal-source-disclosure]].
## 6. Remote Code Execution: command injection

The admin dashboard has a diagnostics feature that displays the date/time. Inspecting the request in DevTools showed it sends the **raw shell command** in a `sys` parameter (`sys=date +"%H:%M:%S"`), POSTed to `dashboard.php`. The backend executes whatever string I supply.

I appended a pipe operator followed by `ls` to test execution, and the server returned a full directory listing, confirming command injection:

```
sys=date|ls
```

![[assets/support16.png]]

Reusing the same vector, I dumped the database file that the app includes. This finally revealed the real user table and explained the admin password (`support110`) from section 5:

```
sys=date|cat /var/www/db.php
```

![[assets/support19.png]]

Finally, I read the target file directly:

```
sys=date|cat /home/ubuntu/user.txt
```

![[assets/support17.png]]

**`/home/ubuntu/user.txt`:** `THM{GOT_THE_FLAG001}`

The core objective of this room is to achieve Remote Code Execution on the server, which this command injection delivers.

See [[techniques/os-command-injection|os-command-injection]].
## Security findings summary

| # | Finding | Impact |
|---|---------|--------|
| 1 | Session cookie without `HttpOnly` / `Secure` / `SameSite` | Session theft via XSS |
| 2 | Exposed `phpinfo()` (`info.php`) | Full stack / path disclosure |
| 3 | Weak credentials (`help@support.thm:snoopy`) | Trivial account takeover |
| 4 | Role decided by client-side cookie `isITUser = md5(bool)` | Privilege escalation by forging the cookie |
| 5 | Path traversal via `?skin=` | Source/credential disclosure (`config.php`, `api.php`) |
| 6 | Hardcoded credentials in source | Plaintext admin password |
| 7 | IDOR in `/user/<id>` | Enumeration of all user profiles |
| 8 | Command injection via `sys` parameter | Remote Code Execution |

## Kill chain

Recon (nmap, gobuster, phpinfo) → weak login (`ffuf` + rockyou) → forge `isITUser = md5("true")` (broken access control) → LFI via `?skin=../config` / `../api` → admin login → **flag 1** → command injection via `sys` (dump `db.php`, read `user.txt`) → **flag 2**.

## 🔗 Techniques used
- [[techniques/client-side-role-cookie|client-side-role-cookie]]
- [[techniques/path-traversal-source-disclosure|path-traversal-source-disclosure]]
- [[techniques/os-command-injection|os-command-injection]]
