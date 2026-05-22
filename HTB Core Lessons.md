# Penetration Testing Mindset

- **What is the system and what is the highest‑impact outcome?**
- **What components exist and what trusts what?
- **What is externally reachable right now?**
- **Where does untrusted input become trusted or executed?**
- **What shortest chain leads from current access to maximum control?**

# Operational Procedure

- **Define Scope & Objectives** – Identify the system, critical assets, and high‑value outcomes.
    
- **Map the Environment** – Enumerate hosts, services, ports, and network topology.
    
- **Enumerate & Analyze** – Inspect each service for misconfigurations, default credentials, or trust violations.
    
- **Identify Weaknesses** – Classify vulnerabilities: misconfigurations, injections, auth flaws, cloud misconfigurations, admin panels.
    
- **Exploit & Chain Access** – Test exploits or logic flaws; pivot between services using credentials or trust relationships.
    
- **Post‑Exploitation & Internal Pivot** – Enumerate internal services, local databases, or privileged interfaces; escalate access.
    
- **Impact Assessment** – Extract sensitive data, verify system control, and measure risk.
    
- **Document & Report** – Record methodology, findings, exploitation steps, and remediation recommendations.


# WINDOWS / ACTIVE DIRECTORY (when you have creds)

1. SMB → shares, files  
2. LDAP → domain structure, users, computers  
3. Kerberos → tickets, SPNs, delegation  
4. AD CS → certificate services, templates (ESC1–ESC8)  
5. WinRM / RDP → remote access  
6. Privileges → groups, ACLs, GPOs

**Example:** A company domain like `corp.local` where employees log in with usernames such as `john.doe`. You connect to SMB and see shares like `SYSVOL` and `NETLOGON`, query users via LDAP, and discover a certificate service (AD CS) that allows you to request a certificate as `Administrator`, leading to full domain compromise.

Windows / Active Directory is a centralized identity system where users, computers, and permissions are managed through a domain. You might list SMB shares with `smbclient`, enumerate users and computers with LDAP, spray passwords with Kerberos, and discover a certificate service (AD CS) that allows impersonation of `Administrator`, leading to full domain compromise.  

**Tools:** `smbclient`, `crackmapexec` / `nxc`, `ldapsearch`, `kerbrute`, `impacket (GetUserSPNs, secretsdump, changepasswd)`, `certipy-ad`, `evil-winrm`, `xfreerdp`

# WINDOWS / ACTIVE DIRECTORY

**IDENTITY**  
In a Windows domain, identity enumeration involves discovering users, groups, and computer accounts stored in Active Directory. This is typically done using tools like `ldapsearch`, `nxc ldap`, or `crackmapexec ldap` to query directory objects, along with `kerbrute userenum` to identify valid usernames and `rpcclient` or `enum4linux` to list domain users and groups.

**AUTH**  
Authentication focuses on validating credentials or obtaining valid login access through Kerberos or NTLM. This is performed using tools such as `kerbrute` for password spraying, `nxc smb` to test credentials across SMB, and `impacket-getTGT` or `kinit` to request Kerberos tickets once valid credentials are known.

**ACCESS**  
Access refers to gaining interaction with the system, either through file shares or remote shells. This is achieved using tools like `smbclient` to browse shares, `evil-winrm` for remote PowerShell access, `xfreerdp` for graphical login, and Impacket tools such as `wmiexec.py` or `psexec.py` for command execution.

**TRUST**  
Trust exploitation involves abusing systems that grant access without traditional password authentication. In Active Directory, this commonly includes certificate services and delegation. Tools like `nxc ldap -M adcs` and `certipy-ad` are used to identify and exploit vulnerable certificate templates, while `bloodhound-python` and `GetUserSPNs.py` help map delegation paths and Kerberos trust relationships.

**PRIVILEGE**  
Privilege escalation involves increasing permissions within the domain or local system. This can be done by analyzing group memberships with `whoami /groups`, identifying weak permissions using `nxc smb --shares`, running enumeration tools like `winpeas`, or modifying accounts with tools such as `impacket-changepasswd` and built-in utilities like `schtasks` or `icacls`.

**SECRETS**  
Secrets extraction focuses on retrieving credentials such as password hashes or plaintext secrets. This is typically performed using tools like `secretsdump.py` to extract NTLM hashes, `mimikatz` or `lsassy` to dump credentials from memory, and `samdump2` to extract local account hashes.

---

# LINUX (when you have shell or creds)

1. Users → /etc/passwd, /etc/shadow, id  
2. Sudo → sudo -l (privilege escalation paths)  
3. SUID / Capabilities → special binaries with elevated rights  
4. Cron jobs → scheduled tasks running as root  
5. Services → running processes (ps, netstat)  
6. Files → configs, credentials, keys

**Example:** A web server running Ubuntu where you gain a low-privilege shell as `www-data`. You check `sudo -l` and find the user can run a script as root without a password. You modify or abuse that script to execute commands as root and escalate privileges.

Linux is a local-user-based operating system where access and privilege escalation depend on file permissions, sudo rights, and system misconfigurations. You might read `/etc/passwd` to enumerate users, run `sudo -l` to find escalation paths, discover SUID binaries that execute as root, or exploit cron jobs or exposed credentials in configuration files to gain root access.  

**Tools:** `bash`, `sudo`, `find`, `getcap`, `linpeas`, `ps`, `netstat`, `grep`, `cron`

# LINUX

**IDENTITY**  
On Linux systems, identity enumeration involves identifying local users and accounts by reading files such as `/etc/passwd` and using commands like `id` and `whoami` to determine the current user context and group memberships.

**AUTH**  
Authentication consists of validating or obtaining login credentials, most commonly through SSH. Tools such as `ssh` are used for login attempts, while `hydra` can perform brute-force attacks, and `su` allows switching users when credentials are known.

**ACCESS**  
Access is achieved by obtaining a shell or interacting with files on the system. This typically involves using `ssh` to gain remote access, interacting with the system through `bash`, and transferring files using `scp` or `sftp`.

**TRUST**  
Trust mechanisms in Linux often involve implicit access configurations such as SSH key-based authentication, mounted network shares, or inherited environment variables. These can be explored using commands like `ls ~/.ssh/authorized_keys`, `mount` to identify shared filesystems, and `env` to inspect environment variables.

**PRIVILEGE**  
Privilege escalation focuses on gaining root access by exploiting misconfigurations. This includes checking sudo permissions with `sudo -l`, identifying SUID binaries using `find / -perm -4000`, discovering capabilities with `getcap -r /`, reviewing scheduled tasks with `crontab -l`, and running automated tools like `linpeas`.

**SECRETS**  
Secrets extraction involves locating sensitive data such as password hashes or credentials stored in files. This is done by reading `/etc/shadow`, searching files with `grep -r "password"`, extracting readable strings from binaries using `strings`, and reviewing environment variables with `env`.

---

# WEB APPLICATION

1. Authentication → login, session handling  
2. Authorization → role checks, IDOR  
3. Input handling → injection points (SQL, command)  
4. File handling → uploads, downloads  
5. API endpoints → hidden functionality  
6. Backend access → database, internal services


**Example:** A login portal where after signing in as a normal user, you change a URL parameter from `user_id=1` to `user_id=2` and gain access to another user’s data (IDOR). Alternatively, you upload a malicious `.php` file through an upload feature and execute it to gain a shell on the server.

A web application handles authentication, user input, and backend logic through HTTP requests. You might bypass authorization by modifying parameters (IDOR), exploit input fields with SQL injection, upload a malicious file for remote code execution, or discover hidden endpoints that expose sensitive functionality.  

**Tools:** `burpsuite`, `curl`, `ffuf`, `gobuster`, `sqlmap`, browser dev tools

# WEB APPLICATION

**IDENTITY**  
In web applications, identity refers to discovering valid users within the system. This can be done by intercepting traffic with `burpsuite`, fuzzing usernames using `ffuf`, and sending crafted requests with `curl`.

**AUTH**  
Authentication involves interacting with login mechanisms and testing their security. Tools like `burpsuite` are used to analyze and manipulate requests, `hydra` can brute-force login forms, `ffuf` can automate login attempts, and `sqlmap` can exploit injection vulnerabilities to bypass authentication.

**ACCESS**  
Access is achieved by interacting with the application as an authenticated or unauthenticated user. This is done through a web browser, scripted requests using `curl`, or advanced manipulation using `burpsuite`.

**TRUST**  
Trust in web applications is often based on session tokens or cookies. These mechanisms can be analyzed and manipulated using `burpsuite` to modify session data, or specialized tools like `jwt_tool` to tamper with JSON Web Tokens.

**PRIVILEGE**  
Privilege escalation involves gaining higher access within the application, such as moving from a normal user to an administrator. This is typically done by exploiting IDOR vulnerabilities using `burpsuite` or discovering hidden endpoints with `ffuf`.

**SECRETS**  
Secrets extraction includes retrieving sensitive backend data such as database contents or API keys. Tools like `sqlmap` are used for database extraction, `burpsuite` for analyzing responses, and `grep` for identifying leaked credentials in source code.


---

# NETWORK / INITIAL ACCESS (no creds)

1. Ports → nmap scan  
2. Services → versions, banners  
3. Web → directories, endpoints  
4. SMB → anonymous access  
5. LDAP → anonymous bind  
6. Vulnerabilities → version-based exploits


**Example:** You scan a target with `nmap` and find ports 22 (SSH), 80 (HTTP), and 445 (SMB) open. The HTTP service reveals a website, SMB allows anonymous access to a share, and SSH suggests possible credential attacks. This information determines your next steps and attack path.

The network phase involves discovering what services are exposed on a target before gaining access. You scan for open ports, identify running services like SSH, SMB, or HTTP, enumerate directories on web servers, and search for known vulnerabilities in service versions to determine entry points.  

**Tools:** `nmap`, `rustscan`, `gobuster`, `ffuf`, `smbclient`, `searchsploit`

# NETWORK / INITIAL ACCESS

**IDENTITY**  
At the network stage, identity usually means figuring out what kind of system you are dealing with before you have credentials. You are not enumerating users yet. You are identifying the host itself by its exposed services, hostnames, and banners. This is done with tools like `nmap` to scan ports and services, `smbclient` to see whether SMB exposes a hostname or domain name, and sometimes `ldapsearch` or `kerbrute` if services suggest an Active Directory environment.

**AUTH**  
Authentication at the network stage means testing whether any service accepts anonymous access, default credentials, or weak credentials. This includes trying SMB null sessions with `smbclient`, attempting anonymous LDAP binds with `ldapsearch`, testing login services such as SSH or web forms with `hydra`, and using `kerbrute` for Kerberos user enumeration or password spraying when a domain is present.

**ACCESS**  
Access in the initial phase means obtaining the first usable foothold, whether that is a shell, file share, web session, or authenticated service access. This can happen through a web browser, `curl`, `smbclient`, `ssh`, `ftp`, or remote access tools once credentials are found. At this stage, the goal is not privilege escalation yet, but simply getting a valid way into the system or application.

**TRUST**  
Trust at the network stage means identifying systems that grant access or authority based on configuration rather than on a password alone. Examples include anonymous SMB shares, LDAP services that allow unauthenticated reads, web applications that trust hidden parameters, certificate services such as AD CS that trust certificate enrollment, or APIs that trust exposed tokens. Tools like `nxc ldap -M adcs`, `ldapsearch`, `burpsuite`, and `certipy-ad` become important once enumeration suggests these trust systems may exist.

**PRIVILEGE**  
Privilege in the initial access phase means identifying whether the first foothold already has more power than expected. For example, a valid low-privilege account might unexpectedly have read access to sensitive SMB shares, enrollment rights in a certificate template, or membership in a powerful group. You check this with tools like `nxc smb --shares`, `rpcclient`, `ldapsearch`, and `whoami` once some level of access has been obtained.

**SECRETS**  
Secrets at the network stage are the credentials or sensitive files that can be extracted before or immediately after gaining the first foothold. These might come from anonymous shares, downloaded files, exposed configuration files, web source code, or cloud metadata. Tools like `smbclient`, `curl`, `burpsuite`, `grep`, `strings`, and `aws-cli` help retrieve and inspect these secrets, which are often what enable the next phase of the attack.

---

# CLOUD (AWS / Azure basics)

1. Identity → IAM users, roles  
2. Permissions → policies, privilege escalation  
3. Storage → S3 / blobs (public or misconfigured)  
4. Compute → instances, metadata access  
5. Secrets → keys, tokens, env variables  
6. Trust → role assumption, federation

**Example:** You discover exposed AWS credentials on a server and use them to run `aws s3 ls`, revealing a public or misconfigured storage bucket containing backups. You then check IAM roles and find you can assume a higher-privileged role, leading to full control of the cloud environment.

Cloud environments use API-driven identity and permission systems instead of local users. You might find exposed credentials, list storage buckets containing sensitive data, identify overly permissive roles, or escalate privileges by assuming a more powerful role within the cloud account.  
**Tools:** `aws-cli`, `az-cli`, `gcloud`, `ScoutSuite`, `Pacu`

# CLOUD (AWS)

**IDENTITY**  
In cloud environments, identity revolves around users and roles managed by systems like AWS Identity and Access Management. Tools such as `aws iam` and `aws sts get-caller-identity` are used to identify the current identity and permissions.

**AUTH**  
Authentication is based on access keys and tokens rather than passwords. These credentials are used with the `aws-cli` or extracted from environment variables to authenticate to cloud services.

**ACCESS**  
Access involves interacting with cloud resources such as storage and compute services. This includes listing storage buckets with `aws s3 ls` and querying instances with `aws ec2 describe-instances`.

**TRUST**  
Trust relationships in cloud environments are defined through role assumption and policy delegation. Tools like `aws sts assume-role` and frameworks like `pacu` are used to exploit these relationships and gain elevated access.

**PRIVILEGE**  
Privilege escalation occurs when overly permissive policies allow users to gain higher privileges. Tools like `pacu` and `ScoutSuite` are used to analyze and exploit these misconfigurations.

**SECRETS**  
Secrets extraction involves retrieving sensitive data such as credentials stored in cloud resources. This can be done using commands like `aws s3 cp` to download files or inspecting environment variables with `env`.


---

# CONTAINER / DOCKER

1. Environment → env variables (secrets)  
2. Filesystem → mounted volumes  
3. Privileges → root inside container  
4. Capabilities → dangerous flags  
5. Escape → host access vectors  
6. Network → internal services

**Example:** You gain access to a Docker container running a web app. Inside, you run `env` and find database credentials. You also discover the Docker socket (`/var/run/docker.sock`) is mounted, allowing you to interact with the host system and escape the container to gain root access on the underlying machine.

Containers are isolated application environments that often expose sensitive data through environment variables or misconfigurations. You might extract credentials using `env`, find mounted volumes that expose host files, or abuse the Docker socket to escape the container and gain control of the host system.  
**Tools:** `docker`, `env`, `linpeas`, `ps`, `mount`, `nsenter`

# CONTAINERS

**IDENTITY**  
In container environments, identity refers to the user context within the container. This is determined using commands like `whoami` and `id`.

**AUTH**  
Authentication is often minimal in containers since access is typically already granted upon entry, and separate authentication mechanisms are rarely enforced internally.

**ACCESS**  
Access involves interacting with the container environment through a shell, using commands like `bash` and `ls` to explore the filesystem.

**TRUST**  
Trust relationships in containers often involve the boundary between the container and the host system. These can be examined by checking for mounted Docker sockets using `ls /var/run/docker.sock` or inspecting mounted filesystems with `mount`.

**PRIVILEGE**  
Privilege escalation involves escaping the container to gain access to the host. This can be achieved using tools like `docker` to control the host daemon, `nsenter` to access host namespaces, or enumeration tools like `linpeas`.

**SECRETS**  
Secrets extraction focuses on identifying sensitive data within the container, such as environment variables and configuration files. This is done using `env` and reading configuration files with `cat`.
---

# CRITICAL PATTERN TO LOCK IN

Every system reduces to:

IDENTITY → AUTH → ACCESS → TRUST → PRIVILEGE → SECRETS

Your mistake on this box:

Stayed in IDENTITY (passwords)  
Missed TRUST (AD CS)

---

