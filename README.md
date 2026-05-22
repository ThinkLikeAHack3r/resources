# ThinkLikeAHack3r Resources

Free methodology references for people learning hacking, HackTheBox, TryHackMe, Capture The Flag labs, and practical penetration testing.

These resources are designed to help beginners move beyond tutorial-following and develop independent black-box reasoning. The goal is not to provide walkthrough answers. The goal is to help you follow a repeatable testing process, recognize what each stage is asking, and verify your reasoning.

> **Core principle:** The data is the hint.

---

## Resource Stack

Use these in order.

### 1. Port & Service Glossary

- [Read online](./port-service-glossary.md)

Reference guide for common ports, services, protocols, and their pentesting significance.

Use this when you need to understand what a port or service is.

**Question it answers:**

```text
What is this service, and why does it matter?
```

---

### 2. Pentest Reasoning Framework

- [Read online](./pentest-reasoning-framework.md)
- [Quick reference](./pentest-reasoning-framework-quick-reference.md)

The core stepwise methodology for working through an unguided box or authorized test.

It teaches the repeating sequence:

```text
Starting Data
→ Discovery
→ Service Interpretation
→ Access Hypothesis
→ Initial Foothold
→ Local Enumeration
→ Privilege Escalation
→ Objective Verification
→ Debrief
```

The quick reference is the one-page memory anchor for the framework:

```text
IDENTITY → AUTH → ACCESS → TRUST → PRIVILEGE → SECRETS
```

Use the full framework as the operating process. Use the quick reference while actively working through a target.

**Question it answers:**

```text
Where am I in the engagement, and what question should I answer next?
```

---

### 3. AI Process Auditor Prompt

- [Read online](./ai-process-auditor-prompt.md)

A prompt for using an AI assistant to audit your process after you have already done the reasoning.

Use this to check assumptions, identify missing evidence, validate whether your next action follows from your data, and catch skipped stages.

Do not use it to outsource the thinking. Use it to audit the thinking.

**Question it answers:**

```text
What am I assuming that the evidence does not prove?
```

---

## Recommended Workflow

```text
Scan
→ Port & Service Glossary
→ Pentest Reasoning Framework
→ Pentest Reasoning Framework Quick Reference
→ Optional AI Process Auditor
```

In practice:

1. **Start with limited data**
   - Usually an IP address, hostname, URL, or credentials.
   - Ask what discovery would reveal the most useful next information.

2. **Perform discovery**
   - Identify open ports, services, banners, versions, and reachable surfaces.

3. **Interpret the output**
   - Ask what the services imply.
   - Ask whether services are related.
   - Ask whether version information exists.
   - Ask whether known vulnerabilities are relevant.
   - Ask what can be checked safely before exploitation.

4. **Use the quick reference**
   - Route your evidence into the right reasoning category:
     - Identity
     - Authentication
     - Access
     - Trust
     - Privilege
     - Secrets

5. **Build an access hypothesis**
   - Choose the most evidence-supported path.
   - Do not attack every port equally.

6. **Obtain and verify initial foothold**
   - Identify who you are, where you are, what operating system you are on, and what access you actually have.

7. **Enumerate locally**
   - Ask what privileges you have, what groups you are in, what files are accessible, and whether privilege escalation is needed.

8. **Escalate only when evidence supports it**
   - Validate the condition that enables privilege escalation.
   - Execute the minimum necessary action in authorized labs.

9. **Verify the objective**
   - Confirm whether the target files, flags, or proof threshold are accessible.

10. **Debrief**
   - Identify what pattern worked, what failed, what tools mattered, and what you should recognize faster next time.

---

## Methodology Anchor

The framework is built around two nested structures.

### Execution sequence

```text
Starting Data
→ Discovery
→ Interpretation
→ Hypothesis
→ Foothold
→ Enumeration
→ Escalation
→ Objective
→ Debrief
```

### System reasoning loop

```text
IDENTITY → AUTH → ACCESS → TRUST → PRIVILEGE → SECRETS
```

Ask these repeatedly:

| Stage | Core Question |
|---|---|
| Identity | What users, hosts, services, roles, or accounts exist? |
| Auth | How do those identities prove who they are? |
| Access | What can I currently reach, read, write, download, upload, or execute? |
| Trust | What does this system trust that it probably should not? |
| Privilege | Where can access become higher-impact access? |
| Secrets | Where would credentials, tokens, keys, hashes, configs, or sensitive data live? |

---

## Episode 1 Companion

These resources support the ThinkLikeAHack3r Episode 1 methodology:

**How to Learn Hacking: Escape the Tutorial Trap**

The episode argues that tutorials often teach steps, not decision-making. These documents are intended to help close that gap by giving you a repeatable reasoning structure for unguided boxes and black-box-style labs.

---

## Intended Audience

These resources are for:

- Beginners who have completed guided rooms but feel lost on unguided boxes
- HackTheBox and TryHackMe learners
- CTF players developing methodology
- Cybersecurity students learning practical enumeration
- Early-stage penetration testers building structured reasoning habits

They are not intended for unauthorized activity. Use them only in legal, authorized labs, training environments, bug bounty programs, or professional engagements where you have permission to test.

---

## Legal and Ethical Use

These materials are for education, authorized security testing, and defensive skill development.

Do not use these resources against systems you do not own or do not have explicit permission to test.

Do not paste real client secrets, private keys, tokens, passwords, or confidential data into public AI tools.

---

## Channel

These resources support the ThinkLikeAHack3r YouTube channel.

More resources will be added as the methodology develops.
