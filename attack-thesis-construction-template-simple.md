# Attack Thesis Construction Worksheet

A simple worksheet for beginners who have scan output and need to decide what to check next.

Use this after your first enumeration scan.

Goal:

```text
Turn raw output into a simple attack thesis.
```

Core rule:

> **The data is the hint.**

Do not start by asking, “What exploit should I run?”

Start by asking:

```text
What do I see?
What does it probably mean?
What should I test next?
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

# 5. What Type of Target Is This?

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

# 6. What Looks Most Promising First?

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

# 7. What Is Blocked or Not Worth Doing Yet?

List paths that currently look weak.

| Path | Why It Is Weak Right Now |
|---|---|
| SSH login | No credentials yet |
| SMB access | Anonymous access denied |
| Version exploit | No vulnerable version confirmed |
| SQL injection | No input point tested yet |
| Password attack | No valid usernames or password policy known |
| [path] | [reason] |

This prevents wasted time.

---

# 8. What Is the Next Test?

Pick one next action.

```text
Next test:
Why this test:
What I expect to learn:
What result would confirm my idea:
What result would refute my idea:
```

Example:

```text
Next test: Manually browse the web app and inspect the URL patterns.
Why this test: HTTP is the richest exposed surface and does not require credentials.
What I expect to learn: pages, parameters, downloads, login behavior, object IDs.
Confirming result: I find user-controlled IDs, files, or endpoints.
Refuting result: the app is static and exposes no useful functionality.
```

---

# 9. Simple Attack Thesis

Fill this in last.

```text
The target appears to be a [type of system] because I observed [ports/services/behaviors].

The most important open ports are [ports] because [reason].

These ports may be related because [relationship].

The strongest current attack surface is [surface] because [evidence].

The next logical test is [test] because it will confirm or refute [specific idea].
```

## Example

```text
The target appears to be a Linux web application server because I observed SSH on port 22 and a Python/Gunicorn web app on port 80.

The most important open ports are 80, 21, and 22 because the web app may expose data, FTP may involve file transfer or credentials, and SSH may become useful if credentials are found.

These ports may be related because credentials discovered through the web app or FTP may work for SSH.

The strongest current attack surface is the web application because it is reachable without credentials and exposes custom functionality.

The next logical test is manual web enumeration because it will show whether the app exposes files, parameters, object IDs, or authentication behavior.
```

---

# 10. Optional AI Check

Only use AI after you write your own thesis.

Ask AI to check:

```text
What assumptions am I making?
What evidence is missing?
Are these ports related?
Is my next test logical?
Am I ignoring a more obvious surface?
What should I avoid wasting time on?
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
Ports → Services → Relationships → Access → Best Surface → Next Test → Thesis
```

If you are stuck, return to the evidence.
