# Attack Thesis Construction Worksheet

A beginner-friendly worksheet for turning enumeration output into a practical attack thesis.

This document is **not box-specific**. It should work for any beginner/intermediate lab, including targets that expose common combinations like FTP + SSH + HTTP.

Goal:

```text
Turn scan output into:
1. what matters,
2. why it matters,
3. what to test next,
4. where to go after the result.
```

Core rule:

> **The data is the hint.**

But beginners need a translation layer. This worksheet gives you that layer.

---

## Quick Start

Fill this out first.

```text
Target:
Current access level: [no credentials / web session / low-priv shell / domain user / other]
Known credentials:
Main scan command:
Main scan output file:
```

---

# 1. Open-Port Inventory

Copy the meaningful open ports from your scan.

| Port | Service | Version / Banner | What This Usually Enables |
|---|---|---|---|
| 21 | FTP | [version if shown] | File listing, upload/download, credentialed file access |
| 22 | SSH | [version if shown] | Remote shell if credentials or keys are found |
| 80 | HTTP | [version if shown] | Web app, website, API, files, routes, auth logic |
| 443 | HTTPS | [version if shown] | Encrypted web app or API |
| 445 | SMB | [version if shown] | File shares, Windows/domain info, possible anonymous access |
| 389 | LDAP | [version if shown] | Directory/domain information |
| 88 | Kerberos | [version if shown] | Active Directory authentication |
| [port] | [service] | [version] | [what this enables] |

Do not stop at:

```text
Port is open.
```

Write what the port **enables**.

---

# 2. Version and Banner Check

For each important service, ask:

```text
Is there a product name?
Is there a version number?
Is there a framework name?
Is there a hostname or domain?
Is there an error page or header leak?
```

| Service | Version / Product Found? | Where Did I See It? | What It Changes |
|---|---|---|---|
| FTP | [yes/no] | [nmap/banner] | May guide version research, but does not prove exploitability |
| SSH | [yes/no] | [nmap/banner] | Usually context only unless very old or credentials exist |
| HTTP | [yes/no] | [headers/page/source] | May guide route/framework/app testing |
| SMB | [yes/no] | [nmap/smb tools] | May help classify Windows/domain behavior |
| [service] | [yes/no] | [source] | [meaning] |

Important:

```text
A version number is evidence, not a vulnerability.
A missing version number is not proof the service is safe.
```

---

# 3. Service Relationship Check

Ports are not isolated. Ask whether one service can produce access to another.

## Common Relationships

| Pattern | What It Often Means | What To Check |
|---|---|---|
| FTP + SSH | File-transfer credentials may also be system credentials | If credentials are found, test whether they work over SSH |
| HTTP + SSH | Web app may leak usernames, passwords, keys, configs, or backups | Inspect web app before attacking SSH |
| HTTP + FTP | Web app may expose files, logs, packet captures, or credentials related to FTP | Inspect downloads and files |
| SMB + LDAP + Kerberos | Likely Windows / Active Directory | Enumerate domain identity and shares |
| Web + database port | Web app may use backend database | Look for config files, injection points, exposed backups |
| Multiple web ports | Main app plus dev/admin/API service | Inspect each web service and compare behavior |
| File share + remote login | Share may contain credentials for remote login | Inspect readable files before login attempts |

Write your relationship notes:

```text
Services that may be related:
Why they may be related:
What evidence would prove the relationship:
```

---

# 4. Access Without Credentials

Before exploiting anything, check what you can access without credentials.

| Surface | Can I Access It? | What Did I See? | Immediate Next Check |
|---|---|---|---|
| Website / web app | [yes/no] | [pages/forms/downloads/errors] | Browse manually; inspect routes and parameters |
| FTP anonymous | [yes/no] | [files/denied] | If allowed, list/download files |
| SMB anonymous | [yes/no] | [shares/denied] | If allowed, inspect shares |
| LDAP anonymous | [yes/no] | [domain info/denied] | If allowed, map users/domain |
| API | [yes/no] | [endpoints/errors] | Inspect methods, parameters, auth |
| Downloads/files | [yes/no] | [file types] | Identify type and inspect content |

Simple rule:

```text
Information without credentials usually comes before exploitation.
```

---

# 5. Service Path Cards

Use these when a service becomes relevant.

Each card answers:

```text
What does this service mean?
What do I test first?
If it works, where do I go?
If it fails, where do I go?
What would make this service important later?
```

---

## 5.1 FTP Path

FTP stands for **File Transfer Protocol**.

FTP can matter because it may:

- allow anonymous file access,
- expose files,
- accept credentials found elsewhere,
- leak plaintext credentials if captured in network traffic.

### First Test

Try anonymous login:

```bash
ftp TARGET_IP
```

Common anonymous credentials:

```text
username: anonymous
password: anonymous
```

Optional script check:

```bash
nmap -p21 --script ftp-anon,ftp-syst TARGET_IP
```

### Decision Logic

| Result | Meaning | Next Move |
|---|---|---|
| Anonymous login works | FTP is an information source | List files, download readable files, inspect content |
| Anonymous login fails | FTP needs credentials | Stop focusing on FTP for now; look for credentials elsewhere |
| Files are readable | Files may contain configs, notes, backups, keys, or logs | Download and inspect safely |
| Upload is allowed | FTP may be a write surface | Check whether uploaded files are reachable/executed, only in authorized labs |
| Version is shown | Product context exists | Record it; research carefully; do not assume exploitable |
| Credentials are found elsewhere | FTP may accept them | Test FTP login if authorized |
| FTP credentials work | File access confirmed | Inspect files; then consider whether same creds work on SSH |
| FTP traffic appears in PCAP | Credentials may be plaintext | Filter for FTP in Wireshark and inspect USER/PASS commands |

### If FTP Fails

Write:

```text
FTP anonymous access failed.
FTP is not currently an unauthenticated information source.
FTP may become useful later if credentials are found from web, SMB, configs, source code, or packet captures.
Next I should check the richest unauthenticated surface.
```

### FTP Thesis

```text
Observation:
FTP is open on port 21.

Interpretation:
FTP may expose files if anonymous login works, or may become useful later if credentials are found.

Prediction:
If anonymous access works, FTP may reveal files or secrets. If it fails, FTP is likely not the first path.

Next action:
Test anonymous FTP. If denied, move to a richer unauthenticated surface such as HTTP, SMB, or another exposed service.
```

---

## 5.2 SSH Path

SSH stands for **Secure Shell**.

SSH usually matters **after** you have credentials or a private key.

### First Test

Confirm the service:

```bash
nmap -p22 -sV TARGET_IP
```

Optional banner grab:

```bash
nc -nv TARGET_IP 22
```

### Decision Logic

| Result | Meaning | Next Move |
|---|---|---|
| SSH open, no credentials | Remote shell exists but no access yet | Look for credentials elsewhere |
| Username found | Login candidate exists | Look for password/key before testing |
| Password found | Credential test possible | Test SSH if authorized |
| Private key found | Key-based login possible | Identify username and test safely |
| Login succeeds | Initial shell access | Begin post-exploitation enumeration |
| Login fails | Creds may be service-specific or invalid | Return to evidence; test other services only if justified |

### SSH Thesis

```text
Observation:
SSH is open on port 22.

Interpretation:
The system supports remote shell login, but SSH likely requires credentials.

Prediction:
SSH will become useful if I find valid credentials or a private key.

Next action:
Do not brute force first. Look for credentials in web content, files, shares, configs, packet captures, or source code.
```

---

## 5.3 HTTP / Web Path

HTTP stands for **Hypertext Transfer Protocol**.

Web is often a strong first surface because it exposes behavior, not just a login prompt.

### First Tests

Open in browser:

```text
http://TARGET_IP
```

Check headers:

```bash
curl -I http://TARGET_IP
```

Optional route discovery:

```bash
ffuf -u http://TARGET_IP/FUZZ -w WORDLIST
```

### Decision Logic

| Result | Meaning | Next Move |
|---|---|---|
| Static page only | Less obvious logic | Check source, headers, directories |
| Login page | Authentication exists | Inspect behavior; test default creds only if appropriate |
| Already logged in | Session/auth logic may be weak | Explore authorization and object access |
| Download feature | Files may contain secrets | Download and inspect file types |
| Upload feature | File handling surface | Check restrictions and storage behavior |
| Numeric IDs in URL | Object access pattern | Test authorization carefully by changing IDs |
| API endpoints | Backend logic exposed | Map endpoints and parameters |
| Error message | Framework/path leak | Record technology and paths |

### HTTP Thesis

```text
Observation:
HTTP is open and exposes a web application.

Interpretation:
The web app is a strong unauthenticated surface because it may expose routes, files, parameters, object IDs, auth logic, or errors.

Prediction:
Manual browsing and route discovery may reveal the next meaningful test.

Next action:
Browse manually first. Identify pages, parameters, downloads, object IDs, login behavior, and errors before running aggressive tools.
```

---

## 5.4 File / Download Path

Use this when **any** service lets you download files.

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

### Decision Logic

| File Found | Why It Matters | Next Move |
|---|---|---|
| `.pcap` | May contain plaintext credentials or traffic | Open in Wireshark; filter FTP, HTTP, DNS, auth strings |
| `.conf`, `.ini`, `.yml`, `.env` | May contain passwords, tokens, hostnames | Search for secrets and internal services |
| `.zip`, `.tar`, backup | May contain old source/configs | Extract locally and inspect |
| Source code | May reveal routes, secrets, logic flaws | Read routes, search for credentials |
| Logs | May reveal usernames, paths, errors, tokens | Search auth events and paths |
| SSH key | May allow remote login | Identify likely username and test if authorized |

### File Thesis

```text
Observation:
A service exposed downloadable files.

Interpretation:
Files may reveal credentials, internal paths, source code, or sensitive traffic.

Prediction:
Inspecting the files may reveal the next access path.

Next action:
Download non-destructively, identify file types, and inspect content for credentials, paths, users, and service relationships.
```

---

## 5.5 SMB Path

SMB stands for **Server Message Block**.

SMB may expose shares, files, domain names, users, or Windows environment clues.

### First Tests

Anonymous share listing:

```bash
smbclient -L //TARGET_IP/ -N
```

If credentials exist:

```bash
smbclient -L //TARGET_IP/ -U USER
```

### Decision Logic

| Result | Meaning | Next Move |
|---|---|---|
| Anonymous shares visible | Files may be readable | Inspect shares |
| Access denied | Credentials needed | Look for creds elsewhere |
| `SYSVOL` / `NETLOGON` | Active Directory likely | Move toward AD enumeration |
| Writable share | Possible write surface | Assess carefully in authorized labs |
| Host/domain name shown | Identity context | Add to thesis |

### SMB Thesis

```text
Observation:
SMB is open.

Interpretation:
The target may expose file shares or Windows/domain information.

Prediction:
If anonymous access is allowed, SMB may reveal files, users, or configuration data.

Next action:
Test anonymous share listing. If denied, revisit SMB after credentials are found.
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

# 7. Rank the Current Attack Surfaces

Rank based on evidence, not habit.

| Rank | Surface | Why It Matters | What I Will Test |
|---|---|---|---|
| 1 | [surface] | [reason] | [next test] |
| 2 | [surface] | [reason] | [next test] |
| 3 | [surface] | [reason] | [next test] |

## Simple Ranking Rules

Usually prioritize:

1. Web apps with visible functionality
2. Anonymous file access
3. Readable shares or downloads
4. Directory/identity services that leak information
5. Login services only after credentials are found

Usually avoid starting with:

```text
Brute-forcing SSH
Running random exploits
Attacking every port equally
Ignoring what the application does
```

---

# 8. Blocked Paths and Return Conditions

List paths that are currently weak, and what would make them worth revisiting.

| Path | Why It Is Weak Right Now | What Would Make It Important Later |
|---|---|---|
| SSH login | No credentials yet | Valid password or SSH key found |
| FTP login | Anonymous denied | Valid FTP credentials found |
| SMB access | Anonymous denied | Valid local/domain credentials found |
| Version exploit | No vulnerable version confirmed | Reliable version-specific evidence |
| SQL injection | No input point tested yet | Parameter/form behavior found |
| Password attack | No usernames or policy known | Usernames and authorization to test |

This prevents wasted time while preserving paths that may matter later.

---

# 9. Pick the Next Test

Choose one next action.

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
Download readable files and inspect for configs, credentials, packet captures, source, or notes.

Where I go if it fails:
Move to the richest unauthenticated surface, such as HTTP or SMB, depending on what else is open.
```

---

# 10. Final Attack Thesis

Fill this in last.

```text
The target appears to be a [type of system] because I observed [ports/services/behaviors].

The most important open ports are [ports] because [reason].

These services may be related because [relationship].

The strongest current attack surface is [surface] because [evidence].

The next logical test is [test] because it will confirm or refute [specific idea].

If that works, I will [next branch].

If that fails, I will [fallback branch].
```

---

# 11. Generic Example: FTP + SSH + HTTP

This is a common beginner pattern.

```text
Open services:
21 FTP
22 SSH
80 HTTP
```

Reasoning:

```text
FTP may expose files if anonymous access works.
SSH probably requires credentials and is likely a destination.
HTTP exposes application behavior and is often the richest unauthenticated surface.
FTP credentials, if found, may also work over SSH.
HTTP may expose files, logs, packet captures, configs, or credentials that make FTP or SSH useful.
```

Thesis:

```text
The target appears to be a Linux server exposing a web application plus file-transfer and remote-login services.

The most important services are HTTP, FTP, and SSH because HTTP may expose application logic or files, FTP may expose or accept credentials, and SSH may become useful if valid system credentials are discovered.

The strongest current attack surface is HTTP if FTP anonymous access fails, because HTTP is reachable without credentials and may expose files, object IDs, routes, or authentication behavior.

The next logical test is to check FTP anonymous access first because it is quick and safe. If anonymous access fails, move to manual web enumeration. If web enumeration exposes files or packet captures, inspect them for credentials that may work against FTP or SSH.
```

This example is not tied to any specific box. It is a pattern.

---

# 12. Optional AI Check

Only use AI after you write your own thesis.

Ask AI to check:

```text
What assumptions am I making?
What evidence is missing?
Are these services related?
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
