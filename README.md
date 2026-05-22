# ThinkLikeAHack3r Resources

Free methodology references for people learning hacking, HackTheBox, TryHackMe, Capture The Flag labs, and practical penetration testing.

These resources are designed to help beginners move beyond tutorial-following and develop independent black-box reasoning. The goal is not to provide shortcuts or walkthrough answers. The goal is to help you understand what your recon output means, what it implies about the target, and what to test next.

> **Core principle:** The data is the hint.

Every port, banner, endpoint, error message, redirect, header, permission, service response, file path, credential, and command output is evidence. These resources are meant to help you read that evidence instead of waiting for a walkthrough to tell you what to do.

---

## Resources

### Port Enumeration Glossary

- [Read online](./port-enumeration-glossary.md)

A quick-reference guide for common ports, exposed services, pentesting significance, first enumeration moves, and common false assumptions.

Use this when you run an initial scan and need to understand what a port or service may imply.

---

### Pentesting Decision Framework

- [Read online](./pentesting-decision-framework.md)
- [Quick cheatsheet](./pentesting-decision-framework-cheatsheet.md)
- [Download PDF](./downloads/pentesting-decision-framework.pdf)

A practical framework for reasoning through identity, authentication, access, trust, privilege, and secrets across networks, web apps, Linux, Active Directory, cloud, and containers.

Use this when you are stuck and need to classify what kind of system you are touching, what evidence you already have, and what question to answer next.

---

### Attack Thesis Construction Template

- [Read online](./attack-thesis-construction-template.md)

A structured template for turning enumeration output into an attack thesis. Use it after initial recon to explain command syntax, interpret service output, classify the target, identify viable attack surfaces, track unknowns, and choose the next logical test.

This template is designed to prevent random tool-chaining. It forces every action to connect back to observed evidence.

---

## How to Use These Resources

Use the resources in this order during a beginner or intermediate lab:

1. **Run initial enumeration**
   - Identify open ports, exposed services, banners, protocols, and reachable application surfaces.

2. **Use the Port Enumeration Glossary**
   - Translate open ports and service names into practical meaning.
   - Ask what each service enables.
   - Identify the most promising surfaces.

3. **Use the Pentesting Decision Framework**
   - Classify the system.
   - Think through identity, authentication, access, trust, privilege, and secrets.
   - Decide what question needs to be answered next.

4. **Use the Attack Thesis Construction Template**
   - Convert observations into a structured thesis.
   - Explain why each command or test is being used.
   - Track unknowns.
   - Choose the next logical action.

The goal is to build a repeatable thinking process, not a memorized sequence of tools.

---

## Methodology

The resources are built around this recurring pattern:

```text
IDENTITY → AUTH → ACCESS → TRUST → PRIVILEGE → SECRETS
```

Ask these questions repeatedly:

| Stage | Core Question |
|---|---|
| Identity | What users, hosts, services, roles, or accounts exist? |
| Auth | How do those identities prove who they are? |
| Access | What can I currently reach, read, write, download, upload, or execute? |
| Trust | What does this system trust that it probably should not? |
| Privilege | Where can access become higher-impact access? |
| Secrets | Where would credentials, tokens, keys, hashes, configs, or sensitive data live? |

If you are lost, return to the evidence.

---

## Episode 1 Companion

These resources support the ThinkLikeAHack3r Episode 1 methodology:

**How to Learn Hacking: Escape the Tutorial Trap**

The episode argues that tutorials often teach steps, not decision-making. These documents are intended to help close that gap by giving you a practical reasoning structure for unguided boxes and black-box-style labs.

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

---

## Contributing

Corrections and improvements are welcome.

Good contributions include:

- Clarifying confusing explanations
- Adding missing port/service context
- Improving beginner readability
- Fixing inaccurate tool descriptions
- Adding safer methodology notes
- Improving examples without turning them into walkthrough spoilers

Please keep the focus on reasoning, evidence, and methodology.

---

## License

This repository is intended to be freely accessible for learning.

See the repository license for reuse terms.

---

## Channel

These resources support the ThinkLikeAHack3r YouTube channel.

More resources will be added as the methodology develops.
