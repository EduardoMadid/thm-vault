---
title: path-traversal-source-disclosure
tags:
  - web
  - lfi
  - path-traversal
  - info-disclosure
  - php
  - tech
---

# Path traversal source disclosure

## In plain terms
The site built a filename out of part of the web address to load a "theme". By putting `../` in that part, you climb out of the theme folder and point the filename at other files on the server. The site then prints those files back to you as text — including a config file that had the admin password sitting in plain sight.

## TL;DR
A theme parameter (`?skin=`) built a file path and returned the file's **contents as text** in the response. Path traversal (`../config`, `../api`) climbed out of `skins/` and disclosed the raw PHP source of `config.php` (a hardcoded master password) and `api.php` (the authorization logic). Because the file was emitted as text rather than executed, plain traversal was enough and no stream wrapper was needed.

## The mechanism
The parameter was used to build a path roughly like `skins/<skin>.php`, and the file's contents were written into the response. Traversal escapes the intended folder:
```
?skin=../config   →  skins/../config.php   →  config.php   (source shown as text)
?skin=../api      →  skins/../api.php       →  api.php      (source shown as text)
```
The disclosed `config.php` contained:
```php
<?php
$MASTER_PASSWORD = 'support@110';
$SITE_VER  = '1.0';
$SITE_NAME = 'support_portal';
```

## 🔑 The key distinction: read-and-emit vs include-and-execute
This is the part worth internalizing, because it decides your toolset:

| Sink behavior | What you see for a `.php` file | How to get the source |
|---------------|-------------------------------|-----------------------|
| `include` / `require` (executes) | The file **runs**; you see its *output*, not its code. A pure-variable `config.php` produces **0 bytes**. | `php://filter/convert.base64-encode/resource=...` to encode-then-read |
| `readfile` / `file_get_contents` / echo (emits text) | The file's **raw source** appears as text | Plain traversal already gives you the source |

In this room the source came out with **plain traversal**, and `php://filter` was *blocked* — both facts point to a read-and-emit sink, not an executing include. That also explains why `config.php` returned 0 bytes when requested directly over HTTP (Apache executed it, no output) but showed full source through the `skin` parameter (the sink printed it as text).
## Detection
- Parameters that name a file/page/template/theme/language: `skin`, `page`, `file`, `template`, `lang`, `view`.
- Try `../` traversal toward known files (`../config`, `../index`, `/etc/passwd`).
- Watch the response: raw `<?php` in the body = source disclosure; a warning about a missing stream = executing include with an appended suffix.

## Mitigation
- Allowlist valid values (`default`, `red`, `green`, `blue`) and reject anything else.
- Strip path separators with `basename()`, and confine with `realpath()` inside the intended directory.
- Never build an include/read path directly from user input.

## Tools
- `curl "http://target/page.php?skin=../config"`
- Browser `view-source:` for raw response inspection
- `php://filter/convert.base64-encode/resource=<file>` *(only when the sink executes and the wrapper is allowed)*

## 🔗 Appears in
- [[rooms/support|support]]
