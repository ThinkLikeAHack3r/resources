# Attack Thesis Construction Worksheet

A beginner-friendly worksheet for turning enumeration output into a clear attack thesis.

Use this after an initial scan or after any major new evidence appears. The goal is not to guess the answer. The goal is to understand what the evidence means and choose the next test deliberately.

> **Rule:** The data is the hint.

---

## What This Is For

Use this worksheet when you have output from tools like:

- `nmap`
- `rustscan`
- `ffuf`
- `gobuster`
- `smbclient`
- `ldapsearch`
- `curl`
- Burp Suite
- Wireshark
- basic Linux enumeration commands

This worksheet helps you answer:

1. What did I actually observe?
2. What does that evidence probably mean?
3. What attack surfaces are most promising?
4. What is blocked or unlikely?
5. What should I test next?
6. Why is that the next logical test?

---

## What This Is Not

This is not a box walkthrough.

This is not a shortcut.

This is not a place to paste a target name and get the answer.

Do not use online writeups to complete this. The value comes from reasoning from your own evidence.

---

## Required Input

Before filling this out, collect at least one piece of real evidence.

Minimum useful input:

```text
Target:
Initial command:
Raw output:
Current access level:
Known credentials, if any:
What I am trying to understand:
```

Example:

```text
Target: 10.10.10.X
Initial command: nmap -sC -sV -oN nmap_initial.txt 10.10.10.X
Raw output: ports 21, 22, 80 open
Current access level: no credentials
Known credentials: none
Question: which service should I investigate first?
```

---

# 0. Command Syntax Review

Before interpreting output, make sure you understand the command you ran.

## Command Used

```bash
[command here]
```

## Syntax Breakdown

| Syntax Element | Meaning | Why It Was Used |
|---|---|---|
| `[binary]` | [what the program does] | [why this tool fits the task] |
| `[flag]` | [what the flag means] | [why it matters here] |
| `[argument]` | [what the argument represents] | [why this target/input was chosen] |

## Example

Command:

```bash
nmap -sC -sV -oN nmap_initial.txt 10.10.10.5
```

| Syntax Element | Meaning | Why It Was Used |
|---|---|---|
| `nmap` | Network Mapper; scans hosts and services | Converts an unknown IP into exposed services |
| `-sC` | Runs Nmap default scripts | Extracts extra service information safely |
| `-sV` | Attempts service and version detection | Helps identify what software is running |
| `-oN` | Saves normal output to a file | Preserves evidence for later review |
| `10.10.10.5` | Target IP address | The host being tested |

## Learning Notes

Write:

```text
This command confirmed:
This command did not confirm:
The next question this raises:
```

---

# 1. Raw Observations

Write only what the output directly shows. Do not interpret yet.

```text
Observed services:
Observed versions:
Observed hostnames:
Observed redirects:
Observed files:
Observed errors:
Observed credentials:
Observed unusual behavior:
```

## Example

```text
Observed services:
- 21/tcp FTP, vsftpd
- 22/tcp SSH, OpenSSH
- 80/tcp HTTP, Gunicorn/Python

Observed behavior:
- Web app is reachable without login.
- App exposes packet capture downloads through /data/{id}.
```

---

# 2. Plain English Thesis

Complete this in simple language.

```text
This machine appears to be a [system role] because it exposes [key services or behaviors].

The strongest current evidence is [specific ports, banners, pages, files, or behaviors].

There are [no obvious / possible / specific] service-level vulnerabilities visible yet.

The likely attack path is [web / identity / file access / misconfiguration / credential abuse / network service exploitation].

The next move should be [specific action] because [reason tied to evidence].
```

## Example

```text
This machine appears to be a Linux web application server because it exposes SSH and a Python/Gunicorn HTTP service.

The strongest current evidence is port 80 running a custom web app and port 21 exposing FTP.

There are no obvious version-based vulnerabilities visible yet, so the likely attack path is web application logic or credential exposure.

The next move should be to enumerate the web application because custom web apps usually expose more decision-making evidence than SSH without credentials.
```

---

# 3. Technical Thesis

## 3.1 System Classification

```text
The target is likely a [exact role] operating in a [standalone / Active Directory / containerized / web-hosted / hybrid] environment.

Confidence level: [low / medium / high]

Reason for confidence:
[specific evidence]

Reason to stay flexible:
[missing evidence or ambiguity]
```

---

## 3.2 Evidence Table: Port → Service → Function → Meaning

Only include high-signal ports.

| Port | Protocol | Service | What It Does | What The Output Proves | Why It Matters |
|---|---|---|---|---|---|
| `[port]` | `[TCP/UDP]` | `[service]` | `[function]` | `[confirmed fact]` | `[attack relevance]` |

Do not write “port is open” and stop. Explain what capability that port exposes.

## Example

| Port | Protocol | Service | What It Does | What The Output Proves | Why It Matters |
|---|---|---|---|---|---|
| 22 | TCP | SSH | Encrypted remote shell login | Remote login service exists | Useful if credentials or keys are found |
| 80 | TCP | HTTP/Gunicorn | Serves a Python web app | Custom web application is exposed | Likely source of logic flaws or data exposure |
| 21 | TCP | FTP | File transfer service | FTP is reachable | May expose files or plaintext credentials |

---

## 3.3 Acronym and Protocol Review

Expand every acronym the first time it appears.

| Term | Expanded Form | Plain English Meaning | Pentesting Relevance |
|---|---|---|---|
| SSH | Secure Shell | Encrypted remote command-line access | May allow login if credentials or keys are found |
| SMB | Server Message Block | Windows-style file and printer sharing | May expose shares, users, or writable locations |
| LDAP | Lightweight Directory Access Protocol | Directory lookup protocol | May expose users, groups, domains, and policy data |
| HTTP | Hypertext Transfer Protocol | Web traffic protocol | May expose web applications, APIs, files, or auth portals |
| FTP | File Transfer Protocol | Transfers files between systems | May expose files or plaintext credentials |

Add every acronym encountered during the box.

---

# 4. Service Grouping and Pattern Recognition

Group services by function. This prevents random tool-chaining.

## Authentication

```text
Services:
[Kerberos / SSH / login form / FTP auth / SMB auth]

Meaning:
[what authentication mechanism exists]

Attack implications:
[user enumeration / password spraying / Kerberos attacks / default credentials / session abuse]
```

## Directory / Identity / Metadata

```text
Services:
[LDAP / DNS / RPC / SNMP / API metadata]

Meaning:
[what information can be queried]

Attack implications:
[domain mapping / usernames / hostnames / group discovery]
```

## File Access

```text
Services:
[SMB / FTP / NFS / WebDAV / downloads]

Meaning:
[how files may be listed, read, uploaded, or executed]

Attack implications:
[anonymous access / writable shares / credential leaks / upload abuse]
```

## Application Layer

```text
Services:
[HTTP / HTTPS / custom web service / API]

Meaning:
[what application surface exists]

Attack implications:
[fuzzing / authentication testing / file discovery / API testing / injection testing / authorization testing]
```

## Remote Execution / Administration

```text
Services:
[SSH / RDP / WinRM / Telnet / admin panels]

Meaning:
[how interactive access may happen]

Attack implications:
[credential reuse / key abuse / remote shell access]
```

## Conclusion

```text
This system primarily functions as a [identity system / web application / file server / database server / hybrid].
```

---

# 5. Security Controls and Constraints

List anything that weakens or removes attack paths.

| Observation | What It Means | Attack Paths Reduced or Eliminated |
|---|---|---|
| `[example: SMB signing required]` | SMB relay is likely not viable | NTLM relay against SMB |
| `[example: no anonymous FTP]` | Cannot list files without credentials | Anonymous file discovery |
| `[example: filtered UDP]` | UDP services may exist but are not confirmed | Do not assume only TCP matters |

Conclusion:

```text
These constraints currently eliminate or reduce [attack types], but they do not eliminate [remaining viable paths].
```

---

# 6. Attack Surface Characterization

Complete the sentence:

```text
This is a [type] target, not primarily a [type] target.
```

Examples:

```text
This is an identity target, not primarily a web exploit target.
This is a web application target, not primarily an infrastructure target.
This is a file-access target, not primarily a remote code execution target.
```

## Primary Surfaces

| Surface | What Can Be Done There | Current Evidence | Next Test |
|---|---|---|---|
| `[Surface 1]` | `[action]` | `[evidence]` | `[next command or check]` |
| `[Surface 2]` | `[action]` | `[evidence]` | `[next command or check]` |
| `[Surface 3]` | `[action]` | `[evidence]` | `[next command or check]` |

---

# 7. Unknowns and Required Tests

Do not hide uncertainty. Track it.

| Unknown | Why It Matters | How To Test It |
|---|---|---|
| Is UDP exposing SNMP? | Could leak system information | Run targeted UDP scan |
| Is SMB anonymous enabled? | Could expose shares | Run SMB enumeration |
| Is the web app custom? | Custom apps often contain logic flaws | Inspect pages, routes, headers, and behavior |
| Does object access enforce authorization? | Determines whether IDOR is possible | Change object IDs and compare access behavior |

---

# 8. Logical Attack Strategy

Write the strategy as a chain of reasoning, not a random tool list.

## Step 1: Information Without Credentials

```text
First, attempt [enumeration action] because [reason].

Expected useful output:
[users / shares / directories / versions / endpoints]

If successful, this changes the thesis by:
[how]
```

## Step 2: Application or Service Interaction

```text
Next, test [specific surface] for [behavior] because [evidence].

Expected useful output:
[error messages / login behavior / file listings / API responses]
```

## Step 3: Credential or Data Acquisition

```text
If enumeration reveals [users/files/configs], attempt [credential discovery / password reuse / hash capture / config review].
```

## Step 4: Initial Access

```text
Use the most plausible access vector:
[SSH / web shell / SMB write / FTP upload / WinRM / exploit / API abuse]
```

## Step 5: Privilege Escalation Direction

```text
After access, check:
[local users / sudo rights / scheduled tasks / writable paths / services / credentials / kernel / capabilities]
```

---

# 9. Thinking Framework

Before writing the final thesis, answer these.

## Role Identification

```text
What role do the ports suggest?
What evidence supports that role?
What evidence contradicts it?
```

## Redundancy Confirmation

```text
Do multiple services support the same classification?
If yes, confidence increases.
If no, keep the hypothesis flexible.
```

## Attack Type Selection

Choose one primary attack category:

- Web exploitation
- Identity or authentication abuse
- File or share abuse
- Network service exploitation
- Misconfiguration
- Credential discovery or reuse

## Constraint Elimination

```text
What is blocked?
What is probably not worth attacking first?
What would be a waste of time right now?
```

## Path of Least Resistance

```text
Where can I get information or access without credentials first?
What single command should I run next?
What do I expect it to prove or disprove?
```

---

# 10. One-Line Final Thesis

Use this format:

```text
The target is a [system role], and due to [key observations and constraints], the attack should focus on [specific attack class] through [primary surface], starting with [next logical enumeration action].
```

## Example

```text
The target is a Linux web application server, and due to the exposed Python/Gunicorn web app plus no current credentials for SSH, the attack should focus on web application logic through the HTTP surface, starting with manual web enumeration and route analysis.
```

---

# 11. Optional AI Validation

Only use AI after you write your own thesis first.

Do not ask AI to solve the box.

Ask it to:

- Identify unstated assumptions
- Point out missing evidence
- Check whether your next action follows from the data
- Explain command syntax
- Explain output fields
- Suggest safe additional enumeration

Use the companion file:

```text
ai-attack-thesis-construction-system-prompt.md
```

The AI prompt is for validation and gap-filling, not replacing your own reasoning.

---

# Final Rule

If you are stuck, return to the evidence.

```text
Observation → Interpretation → Prediction → Next Action
```

Do not memorize tools as the method. Tools answer questions. The question comes first.
