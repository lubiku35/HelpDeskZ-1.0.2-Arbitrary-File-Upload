# Unauthenticated PHP Shell Upload & Hash Guessing

This script exploits a vulnerability in HelpDeskZ ≤ v1.0.2, a PHP-based support ticket system, which allows unauthenticated users to upload arbitrary PHP files (e.g., shells) through the ticket form.

Although uploaded files are renamed using:

```php
$filename = md5($_FILES['attachment']['name'] . time()) . "." . $ext;
```

the timestamp used is not randomized. As a result, MD5 hash values are predictable if the attacker knows:

1. The original uploaded filename
2. An approximate timestamp (e.g. server time)

By brute-forcing the hash using a small time window, this script locates the URL of the uploaded malicious shell.

## Vulnerability Details

**Software**: HelpDeskZ

**Version**: ≤ v1.0.2

**CVE**: Not officially assigned (historical)

**Vulnerability Type**: Unauthenticated File Upload → Remote Code Execution

**Original Author**: Lars Morgenroth (@krankoPwnz)

**Updated by**: Drew Griess + @whoislubiku (Python 3, enhancements)

## Exploit Workflow

1. Upload a .php shell using:

```
http://<target>/helpdeskz/?v=submit_ticket&action=displayForm
```

2. Submit all required fields (name, email, message, captcha) and attach your PHP file (e.g., shell.php).

3. Run this script to find the renamed and uploaded shell:

```
python3 helpdeskz_exploit.py http://<target>/helpdeskz/uploads/tickets/ shell.php
```

## Usage

```
python3 exploit.py <base_url> <uploaded_filename> [--window N] [--verbose]
```

| Argument              | Description                                                                          |
| --------------------- | ------------------------------------------------------------------------------------ |
| `<base_url>`          | URL where HelpDeskZ stores uploads (e.g. `http://<target>/helpdeskz/uploads/tickets/`) |
| `<uploaded_filename>` | The original file name used in upload (e.g. `shell.php`)                             |
| `--window`            | How many seconds in the past to brute-force (default: 300)                           |
| `--verbose`           | Print each attempted URL                                                             |

## Impact

Successful exploitation provides a fully unauthenticated remote shell on the target server. Combined with basic enumeration, attackers can quickly gain code execution.

## Disclaimer

This tool is intended only for authorized security testing and educational purposes.
Do not use this script on systems you do not own or have explicit permission to test.