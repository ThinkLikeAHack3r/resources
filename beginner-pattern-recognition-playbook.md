# Beginner Pattern Recognition Playbook

This is not another tutorial.

This document is a **pattern-recognition layer** for beginners reading scan output.

The goal is simple:

```text
When you see a service, you should know the first safe thing to check.
```

You are not trying to memorize walkthroughs.

You are training this reflex:

```text
Port → Meaning → First Check → Branch
```

---

## Core Rule

> **The data is the hint.**

But raw data needs translation.

A beginner who sees this:

```text
21/tcp open ftp
```

should immediately think:

```text
FTP is open.
Try anonymous login.
If anonymous works, inspect files.
If anonymous fails, FTP is probably useful later only if I find credentials.
```

That is the point of this document.

---

# 1. Immediate Port Reflexes

Use this table while reading your first scan.

| If You See | Think | Do Immediately | If It Works | If It Fails |
|---|---|---|---|---|
| `21 FTP` | File transfer may be exposed | Try anonymous FTP login | List/download/read files | Treat FTP as a credential destination later |
| `22 SSH` | Remote shell exists | Do not brute force first | Use only after finding creds/key | Look for creds elsewhere |
| `80/443 HTTP/HTTPS` | Web app or website exists | Open in browser | Inspect pages, routes, files, IDs, login behavior | Run light directory/route discovery |
| `445 SMB` | File shares / Windows info | Try anonymous share listing | Inspect readable shares | Revisit after creds |
| `389 LDAP` + `88 Kerberos` | Likely Active Directory | Identify domain info | Enumerate users/groups if allowed | Revisit after creds |
| `3389 RDP` | Windows GUI login | Do not attack first | Use after valid creds | Look for creds elsewhere |
| `5985 WinRM` | Remote Windows shell | Do not attack first | Use after valid creds | Look for creds elsewhere |
| `2049 NFS` | Network file share | Check exports | Mount/read files if allowed | Revisit if access changes |
| `161 UDP SNMP` | System info may leak | Try common community string if allowed | Read users/processes/system info | Move on |
| `3306/5432/1433 DB` | Database exposed | Confirm service/version | Use creds if found | Look for creds in web/configs/files |
| `5000/8000/8080` | Dev/custom web service | Open in browser | Inspect app/API behavior | Fuzz routes lightly |

---

# 2. Port 21 FTP Reflex

FTP stands for **File Transfer Protocol**.

When you see:

```text
21/tcp open ftp
```

your first thought should be:

```text
Can I log in anonymously?
```

## First Check

```bash
ftp TARGET_IP
```

Try:

```text
username: anonymous
password: anonymous
```

If blank passwords are accepted in your FTP client, you can also try pressing Enter at the password prompt.

## Branch

| Result | Meaning | Next Move |
|---|---|---|
| Anonymous login works | FTP is an information source | Run `ls`, download readable files, inspect them |
| Anonymous login fails | FTP requires credentials | Stop focusing on FTP for now |
| Files are visible | Files may contain users, configs, backups, keys, logs, or packet captures | Download and inspect locally |
| Upload is allowed | FTP may be writable | In labs only, check whether uploads are reachable or executed |
| Credentials found later | FTP may become useful again | Test those credentials against FTP |
| FTP credentials found | They may be system credentials | Test whether they also work for SSH |

## Pattern Sentence

```text
FTP is open, so I will test anonymous login first.
If anonymous login works, I will inspect files.
If anonymous login fails, FTP becomes a later credential-use path, not my main path right now.
```

---

# 3. Port 22 SSH Reflex

SSH stands for **Secure Shell**.

When you see:

```text
22/tcp open ssh
```

your first thought should be:

```text
This is probably where credentials become useful later.
```

## Do Not Start With

```text
Brute force SSH.
```

## First Check

Confirm version/context:

```bash
nmap -p22 -sV TARGET_IP
```

## Branch

| Result | Meaning | Next Move |
|---|---|---|
| SSH open, no credentials | Login exists but access is blocked | Look for credentials elsewhere |
| Username found | Possible login identity | Search for password/key |
| Password found | Credential test possible | Test SSH if authorized |
| Private key found | Key-based login possible | Identify correct username and test |
| Login succeeds | You have shell access | Start Linux post-exploitation enumeration |

## Pattern Sentence

```text
SSH is open, but I do not have credentials yet.
SSH is probably not the first attack surface.
I will return to SSH after I find a password, username, or private key.
```

---

# 4. HTTP / HTTPS Reflex

HTTP means **Hypertext Transfer Protocol**.

When you see:

```text
80/tcp open http
443/tcp open https
```

your first thought should be:

```text
Open it in a browser and observe behavior.
```

## First Checks

Open:

```text
http://TARGET_IP
https://TARGET_IP
```

Check headers:

```bash
curl -I http://TARGET_IP
```

## Look For

| Thing You See | What It Means | Next Move |
|---|---|---|
| Login page | Authentication exists | Inspect login behavior |
| Already logged in | Session/auth may be weak | Check authorization and object access |
| Download button | Files may contain secrets | Download and inspect file type |
| Upload button | File handling exists | Check restrictions and storage behavior |
| URL like `/data/5` | Numbered object access | Test nearby IDs carefully |
| API endpoint | Backend logic exposed | Map routes and parameters |
| Error message | Technology/path leak | Record framework/path details |
| Static page | Less obvious logic | View source, headers, and directories |

## Pattern Sentence

```text
HTTP is open, so I will inspect the application manually first.
I am looking for pages, routes, files, parameters, object IDs, login behavior, and errors.
```

---

# 5. Port Combination Reflexes

Most useful beginner insight comes from **relationships**, not isolated ports.

## FTP + SSH

```text
21 FTP
22 SSH
```

Think:

```text
FTP may expose or accept credentials.
SSH may provide shell access if those credentials are valid system credentials.
```

First move:

```text
Try FTP anonymous.
If denied, look for credentials elsewhere.
If credentials are found, test FTP and then SSH.
```

---

## FTP + HTTP + SSH

```text
21 FTP
22 SSH
80 HTTP
```

Think:

```text
FTP may expose files.
HTTP may expose app behavior, files, logs, packet captures, or credentials.
SSH may become the shell path after credentials are found.
```

First moves:

```text
1. Try FTP anonymous.
2. If FTP anonymous fails, inspect HTTP.
3. If HTTP exposes files or packet captures, inspect them.
4. If credentials appear, test FTP or SSH.
```

Pattern sentence:

```text
FTP and SSH are likely credential-dependent. HTTP is reachable now and may reveal the credentials or files that make FTP/SSH useful.
```

---

## SMB + LDAP + Kerberos

```text
445 SMB
389 LDAP
88 Kerberos
```

Think:

```text
This likely involves Windows / Active Directory.
```

First moves:

```text
1. Try SMB anonymous share listing.
2. Try basic LDAP domain query if allowed.
3. Identify domain name.
4. If credentials appear later, enumerate users, groups, shares, and Kerberos paths.
```

---

## HTTP + Database Port

```text
80 HTTP
3306 MySQL
5432 PostgreSQL
1433 MSSQL
```

Think:

```text
The web app may connect to the database.
Database credentials may exist in configs, source code, backups, or environment files.
```

First moves:

```text
1. Inspect web app.
2. Look for exposed config/source/backup files.
3. Do not assume the database is accessible without credentials.
```

---

## Web Port + High Web Port

```text
80 HTTP
8080 HTTP
5000 HTTP
8000 HTTP
```

Think:

```text
There may be a main app plus dev app, API, admin service, or secondary framework.
```

First moves:

```text
1. Open each web service.
2. Compare page titles, technologies, and behavior.
3. Check whether one service leaks info about the other.
```

---

# 6. Service Result → Next Action Matrix

Use this after your first check.

| Result | Immediate Meaning | Next Action |
|---|---|---|
| Anonymous FTP works | Files are reachable | Download and inspect files |
| Anonymous FTP denied | FTP needs creds | Move to web/SMB/other info source |
| SSH open but no creds | Login path exists | Search for creds elsewhere |
| Web app has downloads | Files may contain secrets | Inspect downloaded files |
| Web app has numeric IDs | Possible object access issue | Test adjacent IDs carefully |
| SMB shares readable | Files may contain secrets | Inspect shares |
| LDAP reveals domain | AD likely | Map domain/users if allowed |
| PCAP found | Traffic may contain credentials | Open in Wireshark and filter protocols |
| Config file found | May contain secrets | Search for passwords/tokens/hosts |
| Credentials found | New access path exists | Test against relevant services |
| Shell obtained | Initial access achieved | Move to privilege escalation enumeration |

---

# 7. Beginner Attack Thesis Builder

After matching patterns, fill this out.

```text
I see these important services:
[ports/services]

The first service I should check is:
[service]

Because:
[reason]

If it works, I will:
[next action]

If it fails, I will:
[fallback action]

The services may relate this way:
[relationship]

My current thesis:
[target appears to be X, and the likely first path is Y because Z]
```

## Example Pattern

```text
I see these important services:
FTP, SSH, HTTP.

The first service I should check is:
FTP anonymous login, because it is quick and may expose files.

If it works, I will:
list/download/inspect files.

If it fails, I will:
move to HTTP because it is reachable without credentials and may expose application behavior.

The services may relate this way:
HTTP or FTP may reveal credentials that later work over SSH.

My current thesis:
The target appears to be a Linux server with web, file-transfer, and remote-login services. The likely first path is unauthenticated information gathering through FTP or HTTP, with SSH becoming useful only if credentials are discovered.
```

---

# 8. What This Is Training

This document is not telling you the answer.

It is training automatic pattern recognition:

```text
FTP open → try anonymous
SSH open → save for credentials
HTTP open → inspect behavior
SMB open → check shares
LDAP/Kerberos → think Active Directory
Files found → inspect for secrets
Credentials found → test related services
Shell found → enumerate privilege paths
```

That is the point.

Not walkthrough dependency.

Pattern recognition.
