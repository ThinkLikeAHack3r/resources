# Attack Thesis Construction Worksheet

A simple worksheet for beginners who have scan output and need to decide what to check next.

Use this after your first enumeration scan.

Goal:

```text
Turn raw output into a simple attack thesis and a next action.
```

Core rule:

> **The data is the hint.**

Do not start by asking:

```text
What exploit should I run?
```

Start by asking:

```text
What do I see?
What does it probably mean?
What should I test next?
Where do I go if that test works or fails?
```

---

## Quick Start

Fill this out first.

```text
Target:
Current access level: [no credentials / web access / user shell / domain user / other]
Known credentials:
Main scan command:
Main scan output file:
```

---

# 1. What Ports Are Open?

Copy only the important open ports from your scan.

| Port | Service | Version / Banner | What It Usually Means |
|---|---|---|---|
| 21 | FTP | [version if shown] | File transfer; may allow anonymous login or expose files |
| 22 | SSH | [version if shown] | Remote Linux login; usually useful after credentials are found |
| 80 | HTTP | [version if shown] | Web application or website |
| 443 | HTTPS | [version if shown] | Encrypted web application |
| 445 | SMB | [version if shown] | Windows file sharing; may expose shares or domain info |
| 389 | LDAP | [version if shown] | Directory service; often Active Directory |
| 88 | Kerberos | [version if shown] | Active Directory authentication |
| [port] | [service] | [version] | [what it may mean] |

---

# 2. Is There a Version Number?

For each important service, ask:

```text
Did the scan show a version?
Did the banner show a product?
Did the website reveal a framework?
Did HTTP headers reveal a server?
Did an error page reveal software?
```

| Service | Version Found? | Where Did I See It? | Why It Matters |
|---|---|---|---|
| FTP | [yes/no] | [nmap/banner] | Old versions may have known issues |
| SSH | [yes/no] | [nmap/banner] | Usually not first attack unless very old or credentials exist |
| HTTP | [yes/no] | [headers/page/source] | Frameworks and servers guide enumeration |
| SMB | [yes/no] | [nmap/smb tools] | Helps classify Windows/domain behavior |
| [service] | [yes/no] | [source] | [meaning] |

Important:

```text
A version number does not automatically mean exploitable.
A missing version number does not mean safe.
```

---

# 3. Are the Open Ports Related?

Look for patterns. Ports are not isolated facts.

## Common Relationships

| Pattern | What It Suggests |
|---|---|
| 80/443 + 22 | Web app may leak credentials usable over SSH |
| 21 + 22 | FTP credentials may also work over SSH |
| 445 + 389 + 88 | Likely Windows / Active Directory environment |
| 80/443 + database port | Web app may connect to backend database |
| 8080/5000/8000 | Possible dev server, API, or custom app |
| SMB + readable files | Files may contain credentials or config data |
| Web download + PCAP/logs | Files may contain credentials or sensitive traffic |

Write your relationship notes:

```text
Ports that may be related:
Why they may be related:
What this relationship could reveal:
```

Example:

```text
Ports that may be related: 21 FTP, 22 SSH, 80 HTTP
Why they may be related: the web app may expose files or packet captures, FTP may expose credentials, SSH may accept reused system credentials.
What this relationship could reveal: credentials that become interactive shell access.
```

---

# 4. What Can I Access Without Credentials?

Check what is reachable before attacking anything.

| Surface | Can I Access It? | What Did I See? | Next Simple Check |
|---|---|---|---|
| Website | [yes/no] | [pages/forms/downloads] | Click through manually |
| FTP anonymous | [yes/no] | [files/no files/denied] | Try anonymous login |
| SMB anonymous | [yes/no] | [shares/denied] | List shares |
| LDAP anonymous | [yes/no] | [domain info/denied] | Query basic naming context |
| API | [yes/no] | [endpoints/errors] | Inspect routes and responses |
| Downloads/files | [yes/no] | [file types] | Inspect metadata/content |

Simple rule:

```text
Information without credentials usually comes before exploitation.
```

---

# 5. Path Decision Blocks

Use this section when a service becomes interesting.

The point is to avoid getting stuck after the first discovery.

Each path should answer:

```text
If this works, where do I go next?
If this fails, where do I go next?
What evidence would make this path important again?
```

---

## 5.1 FTP Path — Port 21

FTP stands for **File Transfer Protocol**.

FTP can matter in four ways:

1. It may allow anonymous access.
2. It may expose files.
3. It may accept credentials found somewhere else.
4. It may leak credentials through packet captures because FTP transmits credentials in plaintext.

### First FTP Test

```bash
ftp TARGET_IP
```

Try:

```text
username: anonymous
password: anonymous
```

Optional script check:

```bash
nmap -p21 --script ftp-anon,ftp-syst TARGET_IP
```

### FTP Decision Tree

| Result | What It Means | What To Do Next | Thesis Update |
|---|---|---|---|
| Anonymous login works | FTP is an information source | List files, download readable files, inspect content | FTP becomes a primary file-access path |
| Anonymous login fails | FTP needs credentials | Stop attacking FTP for now; look for credentials elsewhere | FTP becomes a credential destination |
| Directory listing visible | Files may contain useful data | Download non-destructively and inspect | Look for configs, users, keys, logs, backups |
| Upload allowed | FTP may be writable | Only test benign upload if allowed by lab/scope | Possible file-write path, especially if files are web-served |
| Version number found | Product identified | Record version; check relevance carefully | Version is context, not proof of exploitability |
| Credentials found elsewhere | FTP may accept them | Test login with known creds if authorized | If FTP works, inspect files; then test whether creds also work for SSH |
| PCAP contains FTP traffic | FTP credentials may be plaintext | Open in Wireshark and filter `ftp` | Credentials may become SSH or FTP access |

### What To Inspect If FTP Gives Files

| File Type | Why It Matters | Next Move |
|---|---|---|
| `.txt`, `.md`, notes | May contain usernames, hints, paths | Read for names, services, credentials |
| `.conf`, `.ini`, `.yml`, `.env` | Often contains credentials or service settings | Search for passwords, tokens, hostnames |
| `.pcap` | May contain plaintext credentials | Open in Wireshark; filter FTP, HTTP, auth strings |
| `.zip`, `.tar`, backups | May contain source or configs | Extract locally and inspect |
| `.php`, `.py`, `.js` | Source code may reveal routes or secrets | Search for credentials, endpoints, file paths |
| SSH keys | May allow remote login | Check permissions and username context |

### FTP Relationship Logic

| Related Service | How FTP May Connect |
|---|---|
| SSH / 22 | FTP credentials may also work as Linux user credentials |
| HTTP / 80 or 443 | Web app may expose FTP files, backups, logs, or PCAPs |
| SMB / 445 | FTP and SMB may expose overlapping files or credentials |
| PCAP downloads | Captured FTP traffic may reveal USER and PASS commands |

### If FTP Fails

Do not stop.

Write:

```text
FTP anonymous access failed.
This means FTP is not currently an unauthenticated information source.
FTP may become useful later if I find credentials from HTTP, SMB, files, or PCAPs.
Next I should check the richest unauthenticated surface.
```

For a target with FTP + SSH + HTTP, the richest unauthenticated surface is usually HTTP.

### FTP Mini-Thesis

```text
Observation:
FTP is open on port 21.

Interpretation:
FTP may expose files if anonymous login works, or it may become useful later if credentials are found.

Prediction:
If anonymous login works, FTP may reveal files, configs, or credentials. If it fails, FTP is likely not the first path.

Next action:
Test anonymous FTP access. If denied, move to the strongest unauthenticated surface, usually HTTP.
```

### Cap-Style FTP Reasoning

If the scan shows:

```text
21 FTP
22 SSH
80 HTTP
```

A beginner should reason:

```text
FTP is open, but I do not yet have credentials.
SSH is open, but I do not yet have credentials.
HTTP is open and reachable without credentials.

Therefore HTTP is the best first surface.

If HTTP exposes a packet capture, and the packet capture contains FTP credentials, then FTP becomes important again.
If those FTP credentials belong to a Linux user, SSH may become the access path.
```

---

## 5.2 SSH Path — Port 22

SSH stands for **Secure Shell**.

SSH usually matters after you have credentials.

### SSH Decision Tree

| Result | What It Means | What To Do Next | Thesis Update |
|---|---|---|---|
| SSH open, no creds | Remote shell exists but no access yet | Look for credentials elsewhere | SSH is a destination, not first path |
| Username found | Login candidate exists | Do not brute force blindly; look for password/key | Username may help later |
| Password found | Credential test possible | Test SSH if authorized | Possible initial shell |
| Private key found | Key-based login possible | Identify username and test key safely | Possible initial shell |
| Login succeeds | Shell access | Begin Linux post-exploitation enumeration | Move from initial access to post-exploitation |
| Login fails | Creds not valid for SSH | Try FTP/web/other service if appropriate | Credentials may be service-specific |

### SSH Mini-Thesis

```text
Observation:
SSH is open on port 22.

Interpretation:
The target likely supports remote shell access, but SSH probably requires credentials.

Prediction:
SSH will become useful if I find a valid username/password or private key.

Next action:
Do not brute force first. Look for credentials in web content, FTP/SMB files, configs, PCAPs, or source code.
```

---

## 5.3 HTTP / Web Path — Ports 80, 443, 8080, 5000

HTTP stands for **Hypertext Transfer Protocol**.

Web is often the best beginner starting point because it exposes behavior.

### First Web Tests

Open:

```text
http://TARGET_IP
```

Check headers:

```bash
curl -I http://TARGET_IP
```

Basic directory discovery:

```bash
ffuf -u http://TARGET_IP/FUZZ -w WORDLIST
```

### Web Decision Tree

| Result | What It Means | What To Do Next | Thesis Update |
|---|---|---|---|
| Static page only | Less immediate logic | Check source, headers, directories | Web may still hide files/routes |
| Login page | Auth exists | Inspect behavior, default creds if allowed | Auth surface exists |
| Already logged in | Session handling may be weak | Explore authorization and object access | Access-control testing matters |
| Download feature | Files may contain secrets | Download and inspect | File/data exposure path |
| Upload feature | File handling surface | Check restrictions and storage | Potential upload abuse |
| Numeric object IDs | Possible object access issue | Change IDs carefully and compare | Possible IDOR path |
| API endpoints | Backend logic exposed | Map endpoints/parameters | API testing path |
| Error message | Framework/path leak | Record framework and paths | Technology context |

### Web Mini-Thesis

```text
Observation:
HTTP is open and exposes a web application.

Interpretation:
The web app is the richest unauthenticated surface because it has behavior, routes, files, parameters, or authentication logic.

Prediction:
Manual browsing and route discovery may reveal files, object IDs, credentials, or logic flaws.

Next action:
Browse manually first, then enumerate directories/endpoints based on what the app appears to do.
```

---

## 5.4 File Download Path

Use this when any service lets you download files.

Examples:

```text
FTP files
SMB shares
Web downloads
NFS exports
Backup archives
Packet captures
Logs
Source code
```

### File Decision Tree

| File Found | What It May Contain | What To Do Next |
|---|---|---|
| PCAP | Credentials, sessions, plaintext protocols | Open in Wireshark; filter FTP, HTTP, DNS |
| Config file | Passwords, database strings, internal hosts | Search for credentials and endpoints |
| Source code | Routes, secrets, upload paths, logic flaws | Read routes and search for secrets |
| Backup archive | Old source/configs/secrets | Extract locally and inspect |
| Logs | Usernames, paths, errors, tokens | Search for auth events and paths |
| SSH key | Remote login material | Identify username and test if authorized |

### File Mini-Thesis

```text
Observation:
I found downloadable files.

Interpretation:
Files may reveal credentials, internal paths, source code, or sensitive traffic.

Prediction:
Inspecting the files may reveal the next access path.

Next action:
Download non-destructively, identify file types, and inspect content for credentials, paths, users, and service relationships.
```

---

# 6. What Type of Target Is This?

Choose the best current classification.

```text
[ ] Web application target
[ ] Linux server
[ ] Windows / Active Directory target
[ ] File-share target
[ ] Cloud target
[ ] Container target
[ ] Hybrid / unclear
```

Why?

```text
I think this because:
Evidence that supports it:
Evidence that contradicts it:
Confidence: [low / medium / high]
```

---

# 7. What Looks Most Promising First?

Rank the top three surfaces.

| Rank | Surface | Why It Matters | What I Will Test |
|---|---|---|---|
| 1 | [surface] | [reason] | [next test] |
| 2 | [surface] | [reason] | [next test] |
| 3 | [surface] | [reason] | [next test] |

## Simple Ranking Rules

Usually prioritize:

1. Web apps with visible functionality
2. File shares or downloads
3. Anonymous access
4. Exposed identity services
5. Login services only after you have credentials

Usually do not start with:

```text
Brute-forcing SSH
Running random exploits
Attacking every port equally
Ignoring what the web app actually does
```

---

# 8. What Is Blocked or Not Worth Doing Yet?

List paths that currently look weak.

| Path | Why It Is Weak Right Now | What Would Make It Important Later |
|---|---|---|
| SSH login | No credentials yet | Valid password or SSH key found |
| FTP login | Anonymous denied | Valid FTP creds found |
| SMB access | Anonymous access denied | Valid domain/local creds found |
| Version exploit | No vulnerable version confirmed | Reliable version-specific evidence |
| SQL injection | No input point tested yet | Parameter or form behavior found |
| Password attack | No valid usernames or policy known | Usernames and authorization to test |

This prevents wasted time.

---

# 9. What Is the Next Test?

Pick one next action.

```text
Next test:
Why this test:
What I expect to learn:
What result would confirm my idea:
What result would refute my idea:
Where I go if it works:
Where I go if it fails:
```

Example:

```text
Next test:
Test anonymous FTP login.

Why this test:
FTP is open and may expose files without credentials.

What I expect to learn:
Whether FTP is an information source now or only a credential destination later.

Confirming result:
Anonymous login works and files are listed.

Refuting result:
Anonymous login is denied.

Where I go if it works:
Download readable files and inspect for configs, credentials, PCAPs, source, or notes.

Where I go if it fails:
Move to HTTP because it is reachable without credentials and exposes application behavior.
```

---

# 10. Simple Attack Thesis

Fill this in last.

```text
The target appears to be a [type of system] because I observed [ports/services/behaviors].

The most important open ports are [ports] because [reason].

These ports may be related because [relationship].

The strongest current attack surface is [surface] because [evidence].

The next logical test is [test] because it will confirm or refute [specific idea].

If that works, I will [next branch].

If that fails, I will [fallback branch].
```

## Example

```text
The target appears to be a Linux web application server because I observed SSH on port 22, FTP on port 21, and a Python/Gunicorn web app on port 80.

The most important open ports are 80, 21, and 22 because the web app may expose data, FTP may expose or accept credentials, and SSH may become useful if credentials are found.

These ports may be related because credentials discovered through the web app or FTP may work for SSH.

The strongest current attack surface is the web application because it is reachable without credentials and exposes custom functionality.

The next logical test is manual web enumeration because it will show whether the app exposes files, parameters, object IDs, or authentication behavior.

If that works, I will inspect any exposed files or object IDs.

If that fails, I will return to FTP and verify whether anonymous access or version-specific evidence changes the path.
```

---

# 11. Optional AI Check

Only use AI after you write your own thesis.

Ask AI to check:

```text
What assumptions am I making?
What evidence is missing?
Are these ports related?
Is my next test logical?
Am I ignoring a more obvious surface?
What should I avoid wasting time on?
Where should I go if this path works?
Where should I go if this path fails?
```

Do not ask:

```text
How do I solve this box?
What is the exploit?
Give me the answer.
```

Use the companion prompt:

```text
ai-attack-thesis-construction-system-prompt.md
```

---

# Final Reminder

Keep it simple.

```text
Ports → Services → Relationships → Access → Best Surface → Next Test → Branch
```

If you are stuck, return to the evidence.
