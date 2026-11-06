

# Investigating a Tomcat Web Server Attack with Linux CLI

## Case summary
An attacker broke into a company's Tomcat web server and stole data. They first scanned the server with Nikto, then brute forced the admin account on the Tomcat manager page. Once logged in as admin, they uploaded a web shell and used it to run commands on the server. Every step came from the same IP: 177.101.130.211.

## Scenario
Hands-on lab where I played a system administrator investigating a ransom email about stolen company files (Immersive Labs, Cyber Million programme). I only had the web server's access log, so I used the Linux command line to rebuild what the attacker did.

## Tools
Linux CLI (grep, pipes, grep -v, wc)

## Investigation

### 1. Reconnaissance (scan)
**Search:** `grep 'cmd' access.log`

**Found:** Hundreds of requests from 177.101.130.211 with Nikto in the user agent, almost all returning 404.

**Why it's malicious:** Nikto is a vulnerability scanner, so these were automated tests run by the attacker to find weak spots, not hands-on commands.

### 2. Brute force
**Search:** `grep '177.101.130.211' access.log | grep 'manager' | grep -v 'Nikto'`

**Found:** 401 kept repeating on the manager login, then a 200 as admin.

**Why it's malicious:** 401 means wrong password. Lots of them in a row followed by a success shows the attacker guessed the admin password. This was worrying because it means the attacker now had working login credentials.

### 3. Web shell upload
**Search:** `grep 'upload' access.log`

**Found:** At 20:28 the attacker, logged in as admin, uploaded a file through /manager/html/upload. This created the new path /webshell.

**Why it's malicious:** I skipped the Nikto lines because those were automated by the scanner. This upload was a real action by the attacker, and it's how the web shell got onto the server.

### 4. Commands through the web shell
**Search:** `grep 'jsp?cmd' access.log`

**Found:** At 21:29 the attacker sent three commands to /webshell/index.jsp: ls, whoami, then cat /etc/passwd. All returned 200.

**Why it's malicious:** A web shell is a backdoor that lets the attacker run commands on the server through the browser. They checked which account they were using, then read /etc/passwd to see which user accounts exist on the system, so they know who to target next. The URL was encoded: + means a space and %2F means /, so cat+%2Fetc%2Fpasswd decodes to cat /etc/passwd.

## Indicators of Compromise

| Type | Value | What it was |
|---|---|---|
| IP | 177.101.130.211 | Attacker (scan, brute force, upload, commands) |
| Tool | Nikto | Vulnerability scanner |
| Account | admin | Brute forced Tomcat manager account |
| Path | /webshell/index.jsp | Uploaded web shell |
| File read | /etc/passwd | List of user accounts |

## Lessons learnt
- I first thought /etc/passwd held the users' passwords. It actually lists the user accounts. The real password hashes are in /etc/shadow, which only root can read.
- I first thought the Nikto lines were noise because they had no commands in them. They did contain commands like cmd=ls, but they all failed with 404. What made the real attack stand out was the 200 responses, the normal browser user agent, and the time gap.
- When I searched for admin to find the attacker's logged-in activity, it also matched Nikto URLs that had "admin" in them. I fixed it by searching the exact username slot instead.
- I expected the .css file loaded after login to show admin in the log, but it showed - instead. .css files don't need a login, so I filtered by the attacker's IP and removed Nikto to find it.
- Most of the log was scanner noise. Filtering by the attacker's IP and removing Nikto made the real activity stand out.
