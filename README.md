# 🔍 CEH Lab Assignment 1: Footprinting & Reconnaissance

![CEH](https://img.shields.io/badge/CEH-Certified%20Ethical%20Hacker-red?style=for-the-badge)
![Kali Linux](https://img.shields.io/badge/Kali%20Linux-557C94?style=for-the-badge&logo=kali-linux&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

---

## 📌 Objective

The goal of this assignment is to understand how attackers silently collect
intelligence about a target organization before launching any attack.
Using only publicly available tools and information, I performed complete
reconnaissance on a legal target — **hackthissite.org** — to map its
entire digital footprint without triggering any security alerts.

---

## 📚 Concepts Covered

| Concept | Description |
|--------|-------------|
| **Footprinting** | Systematically collecting information about a target's network, systems, and employees |
| **Reconnaissance** | The technique used to perform footprinting — gathering data without being detected |
| **Passive Recon** | Collecting info from public sources without touching the target (WHOIS, Google, Shodan) |
| **Active Recon** | Directly interacting with the target system — can trigger IDS/IPS alerts |
| **OSINT** | Open Source Intelligence — using public data for information gathering |
| **DNS Enumeration** | Discovering domain records, subdomains, and mail servers via DNS queries |

---

## 🛠️ Tools Used

| Tool | Description |
|------|-------------|
| `whois` | Retrieves domain registration info — registrar, dates, name servers |
| `nslookup` | Queries DNS to find IP addresses and mail server records |
| `theHarvester` | Gathers emails, subdomains, and hosts from search engines |
| `Amass` | Deep subdomain enumeration using DNS brute force and certificate logs |
| `DNSRecon` | Comprehensive DNS record discovery — A, MX, TXT, SPF, DMARC |
| `Recon-ng` | Web reconnaissance framework using modules to harvest OSINT data |
| `Shodan` | Search engine for internet-connected devices — reveals open ports and technologies |
| `Maltego` | Visual link analysis tool that maps relationships between domains, IPs, and emails |

---

## 🖥️ Lab Environment
```
Attacker Machine  : Kali Linux 2025.4
Target Domain     : hackthissite.org (legal ethical hacking practice site)
Network           : Standard internet connection
Type              : Passive & Active Reconnaissance Lab
```

---

## ⚡ Practical Execution

### 🔹 Tool 1: WHOIS
```bash
whois hackthissite.org
```

**What it does:** Queries the WHOIS database to retrieve domain registration details.

**What I Found:**
- Registrar: **Porkbun LLC**
- Creation Date: **2003-08-10** (20+ year old domain)
- Expiry Date: **2026-08-10**
- Name Servers: **BuddyNS** (c,f,g,h,j.ns.buddyns.com)
- DNSSEC: **Unsigned** ⚠️
- Domain Status: clientDeleteProhibited, clientTransferProhibited

**🔑 Key Observations:**
- DNSSEC is not enabled — domain is vulnerable to DNS cache poisoning
- Domain expires in 2026 — if not renewed, attacker could attempt registration
- BuddyNS name servers identified — can be probed for further DNS enumeration

---

### 🔹 Tool 2: nslookup
```bash
nslookup hackthissite.org
```

**What it does:** Queries DNS servers to resolve domain names to IP addresses.

**What I Found:**
```
137.74.187.100
137.74.187.101
137.74.187.102
137.74.187.103
137.74.187.104
```

**🔑 Key Observations:**
- 5 IP addresses returned — website uses **load balancing**
- All IPs in same subnet `137.74.187.x` — hosted in same data center (OVH, France)
- Non-authoritative answer — result came from cached DNS resolver

---

### 🔹 Tool 3: theHarvester
```bash
theHarvester -d hackthissite.org -b yahoo
```

**What it does:** Harvests emails, subdomains, and hosts from public search engines.

**What I Found:**
- 📧 Email: `sam@hackthissite.org`
- 🌐 Subdomains: `ctf`, `forum`, `irc`, `legal`, `mirror`.hackthissite.org
- Email format revealed: `firstname@domain.org`

**🔑 Key Observations:**
- Email can be used for **spear phishing** attacks
- IRC subdomain identified — old protocol, often has weaker security
- 5 subdomains found from a single Yahoo search — more sources would yield more results

---

### 🔹 Tool 4: Amass
```bash
amass enum -d hackthissite.org
amass enum -d hackthissite.org -v
```

**What it does:** Performs deep subdomain enumeration using DNS brute forcing,
certificate transparency logs, and 50+ public API integrations.

**What I Found:**
- Completed **2636/2636 enumeration tasks** at 100%
- Speed: 2 probes per second

> ⚠️ **Note:** Amass v5.0.0 completed all tasks successfully but due to
> a known output display bug, results were not printed to the terminal.
> The `amass db` command was removed in v5. DNSRecon was used as an
> alternative to display DNS results.

**🔑 Key Observations:**
- Amass v5 introduced breaking changes — `db` subcommand removed
- Tool still performed full enumeration in the background
- For visible output: use `amass enum -d target.com -o results.txt`

---

### 🔹 Tool 5: DNSRecon
```bash
dnsrecon -d hackthissite.org --lifetime 10
```

**What it does:** Enumerates all DNS records including A, MX, TXT, SPF, DMARC, and SRV.

**What I Found:**

| Record Type | Value | Significance |
|------------|-------|-------------|
| A Records | 137.74.187.100–104 | 5 load-balanced web servers |
| MX Records | aspmx.l.google.com | Uses Google Workspace email |
| SPF | mail.hackthissite.org | Dedicated internal mail server exists |
| DMARC | p=quarantine; pct=25 | Only 25% enforced — email spoofing possible |
| TXT | Harica verification token | SSL certificate from Harica CA |
| DNSSEC | Not enabled | Vulnerable to DNS spoofing |

**🔑 Key Observations:**
- DMARC is only 25% enforced — **75% of spoofed emails could bypass DMARC**
- `mail.hackthissite.org` is a separate mail server — can be targeted independently
- No DNSSEC + weak DMARC = high risk of email-based attacks

---

### 🔹 Tool 6: Recon-ng
```bash
recon-ng
workspaces create hackthissite
db insert domains
modules load recon/domains-hosts/bing_domain_web
options set SOURCE hackthissite.org
run
show hosts
```

**What it does:** Framework-based OSINT tool that uses modules to harvest data
from search engines, APIs, and public databases.

**What I Found — 21 Subdomains:**

| Subdomain | IP Address | Risk |
|-----------|-----------|------|
| `git.hackthissite.org` | 137.74.187.145 | 🔴 Critical — may expose source code |
| `api.hackthissite.org` | 137.74.187.115 | 🔴 Critical — weak API authentication |
| `email.hackthissite.org` | 172.239.57.117 | 🟠 High — dedicated mail server |
| `h5ai.hackthissite.org` | 185.199.111.153 | 🟠 High — file directory server |
| `irc.hackthissite.org` | 185.24.222.13 | 🟡 Medium — old IRC protocol |
| `stats.hackthissite.org` | 137.74.187.136 | 🟡 Medium — analytics server |
| `status.hackthissite.org` | 89.106.200.1 | 🟡 Medium — uptime monitor |

**🔑 Key Observations:**
- **git subdomain** is the most dangerous find — exposed Git repos can contain credentials
- **21 subdomains** discovered from just one Bing search module
- Each subdomain is an independent attack surface

---

### 🔹 Tool 7: Shodan

**Search Query:** `137.74.187.101`

**What it does:** Searches its database of internet-scanned devices to reveal
open ports, running services, and technologies — without touching the target.

**What I Found:**
```
IP          : 137.74.187.101
Country     : France
City        : Dunkerque
ISP         : OVH SAS (AS16276)
Last Seen   : 2026-03-21
Tags        : onion (Tor hidden service)

Open Ports  : 80/TCP, 443/TCP
Server      : HackThisSite (custom header — hides actual web server)
HTTP/2      : Supported
HSTS        : Enabled
jQuery      : v1.81 (outdated ⚠️)
CORS        : Access-Control-Allow-Origin: * (dangerous ⚠️)
```

**🔑 Key Observations:**
- Site has a **.onion Tor address** — dark web presence confirmed
- **jQuery 1.81** is outdated — known XSS vulnerabilities exist
- **CORS wildcard** — any website can make cross-origin requests to this server
- Custom server header hides web server identity — intentional fingerprinting protection

---

### 🔹 Tool 8: Maltego

**What it does:** Visual intelligence tool that automatically maps relationships
between domains, IPs, emails, people, and organizations using transforms.

**What I Found:**
- Complete visual graph of all discovered entities
- Emails linked to employee names
- Subdomains linked to IP addresses and hosting infrastructure
- 20+ nodes mapped showing full attack surface visually

**🔑 Key Observations:**
- Maltego revealed the **interconnections** between all data collected by other tools
- Single transform run discovered emails, subdomains, IPs, and person entities
- Visual mapping helps prioritize targets — most connected nodes = most critical assets

---

## 🎯 Key Findings
```
Domain          : hackthissite.org
Registrar       : Porkbun LLC
IP Addresses    : 137.74.187.100 - 137.74.187.104 (5 servers, OVH France)
Email Found     : sam@hackthissite.org
Email Format    : firstname@domain.org
Subdomains      : 21 discovered (git, api, email, irc, forum, mirror...)
Mail Provider   : Google Workspace (Gmail)
Technology      : jQuery 1.81, HTTP/2, HSTS
Dark Web        : .onion address confirmed
DNSSEC          : Not enabled
DMARC           : Only 25% enforced
```

---

## 🔐 Security Analysis

### What Vulnerabilities Were Identified:

| Vulnerability | Location | Description |
|--------------|---------|-------------|
| Outdated jQuery 1.81 | Main website | Known XSS vulnerabilities (CVE history) |
| CORS Wildcard | Server headers | Any origin can make cross-site requests |
| DNSSEC Disabled | DNS config | DNS cache poisoning attack possible |
| DMARC 25% only | Email config | Email spoofing using org's domain possible |
| Git subdomain exposed | git.hackthissite.org | Source code, API keys, credentials at risk |
| Anonymous mail server | mail.hackthissite.org | Dedicated mail server publicly accessible |

### 🕵️ Attacker Perspective:

> With only this reconnaissance data, an attacker can:
> - Send spear phishing emails to `sam@hackthissite.org` using the leaked email format
> - Scan `git.hackthissite.org` for exposed repositories containing hardcoded credentials
> - Exploit jQuery 1.81 XSS vulnerabilities to hijack user sessions
> - Abuse CORS misconfiguration to steal data from authenticated users
> - Spoof emails from `@hackthissite.org` due to weak DMARC enforcement
> - Enumerate the IP range `137.74.187.0/24` on Shodan to find more exposed services

---

## ⚠️ Risk Assessment

| Finding | Risk Level | Reason |
|---------|-----------|--------|
| `git.hackthissite.org` exposed | 🔴 **Critical** | May expose source code, API keys, credentials |
| `api.hackthissite.org` exposed | 🔴 **Critical** | APIs often have weaker authentication |
| jQuery 1.81 outdated | 🔴 **Critical** | Known XSS vulnerabilities |
| CORS Allow-Origin: * | 🔴 **Critical** | Any site can make cross-origin requests |
| Email address exposed | 🟠 **High** | Enables targeted spear phishing |
| DMARC only 25% enforced | 🟠 **High** | Email spoofing partially possible |
| DNSSEC not enabled | 🟠 **High** | DNS cache poisoning possible |
| Multiple IRC servers | 🟡 **Medium** | Old protocol, weaker security configs |
| h5ai file server exposed | 🟡 **Medium** | May expose files publicly |
| OVH hosting details known | 🟡 **Medium** | Hosting provider can be targeted |
| WHOIS privacy not enabled | 🟢 **Low** | Registrant details partially visible |

---

## 🛡️ Mitigation / Prevention
```
1. Enable WHOIS Privacy      → Hide registrant contact details from public WHOIS
2. Enable DNSSEC             → Prevent DNS cache poisoning and spoofing
3. Fix DMARC Policy          → Change pct=25 to pct=100 and policy to reject
4. Secure Git Subdomain      → Place behind VPN or remove public access
5. Update jQuery             → Upgrade from 1.81 to latest stable version
6. Fix CORS                  → Replace * with specific trusted domains only
7. Deploy WAF                → Block reconnaissance tools and scanners
8. Monitor Shodan            → Regularly check what is publicly exposed
9. Retire IRC servers        → Decommission or firewall old legacy services
10. Security Awareness       → Train employees on phishing and social engineering
```

---

## 💡 What I Learned

- 🔍 **Passive reconnaissance is extremely powerful** — I collected 21 subdomains, 5 IPs,
  employee emails, server technologies, and physical location without sending a single
  packet to the target server directly

- 🛠️ **Different tools reveal different information** — WHOIS gave registrar data,
  nslookup gave IPs, theHarvester gave emails, Recon-ng gave subdomains, Shodan gave
  technologies — combining results gives a complete picture

- ⚠️ **Small misconfigurations create big risks** — DMARC at 25%, CORS wildcard, and
  an exposed git subdomain seem minor but can lead to full system compromise

- 🔐 **DNSSEC and DMARC are critical but often overlooked** — most organizations
  configure email but forget to enforce these DNS security features properly

- 🌐 **Shodan sees everything** — services exposed to the internet are indexed
  automatically; organizations must treat Shodan as an attacker's first tool

- 📊 **Maltego connects the dots** — visual mapping showed me relationships between
  data that would have taken hours to discover manually

---

## ✅ Conclusion

This reconnaissance assignment demonstrated that a complete intelligence profile
of an organization can be built using only publicly available tools and information.
Starting from a single domain name, I discovered 5 server IPs, 21 subdomains,
employee email addresses, web technologies, server location, and a Tor hidden service
— all without triggering a single security alert.

The findings highlight that **what organizations expose publicly is often more than
they realize**, and proactive security monitoring, proper DNS configuration, and
regular attack surface assessments are essential to minimize reconnaissance risk.

---

## 📁 Repository Structure
```
CEH-Assignment-1/
├── README.md               ← This file
├── screenshots/
│   ├── whois_output.png
│   ├── nslookup_output.png
│   ├── theharvester_output.png
│   ├── amass_output.png
│   ├── dnsrecon_output.png
│   ├── recon-ng_output.png
│   ├── shodan_output.png
│   └── maltego_graph.png
└── notes/
    └── findings_summary.txt
```

---

## ⚠️ Disclaimer

> This assignment was performed in a **legal, controlled lab environment** on
> **hackthissite.org** — a website explicitly designed for ethical hacking practice.
> All tools and techniques demonstrated here are for **educational purposes only**.
> Never perform reconnaissance or any form of hacking on systems you do not have
> explicit written permission to test. Unauthorized scanning is illegal under the
> Computer Fraud and Abuse Act (CFAA) and equivalent laws worldwide.

---

<div align="center">
# 🔍 CEH Lab Assignment 1: Footprinting & Reconnaissance

![CEH](https://img.shields.io/badge/CEH-Certified%20Ethical%20Hacker-red?style=for-the-badge)
![Kali Linux](https://img.shields.io/badge/Kali%20Linux-557C94?style=for-the-badge&logo=kali-linux&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

---

## 📌 Objective

The goal of this assignment is to understand how attackers silently collect
intelligence about a target organization before launching any attack.
Using only publicly available tools and information, I performed complete
reconnaissance on a legal target — **hackthissite.org** — to map its
entire digital footprint without triggering any security alerts.

---

## 📚 Concepts Covered

| Concept | Description |
|--------|-------------|
| **Footprinting** | Systematically collecting information about a target's network, systems, and employees |
| **Reconnaissance** | The technique used to perform footprinting — gathering data without being detected |
| **Passive Recon** | Collecting info from public sources without touching the target (WHOIS, Google, Shodan) |
| **Active Recon** | Directly interacting with the target system — can trigger IDS/IPS alerts |
| **OSINT** | Open Source Intelligence — using public data for information gathering |
| **DNS Enumeration** | Discovering domain records, subdomains, and mail servers via DNS queries |

---

## 🛠️ Tools Used

| Tool | Description |
|------|-------------|
| `whois` | Retrieves domain registration info — registrar, dates, name servers |
| `nslookup` | Queries DNS to find IP addresses and mail server records |
| `theHarvester` | Gathers emails, subdomains, and hosts from search engines |
| `Amass` | Deep subdomain enumeration using DNS brute force and certificate logs |
| `DNSRecon` | Comprehensive DNS record discovery — A, MX, TXT, SPF, DMARC |
| `Recon-ng` | Web reconnaissance framework using modules to harvest OSINT data |
| `Shodan` | Search engine for internet-connected devices — reveals open ports and technologies |
| `Maltego` | Visual link analysis tool that maps relationships between domains, IPs, and emails |

---

## 🖥️ Lab Environment
```
Attacker Machine  : Kali Linux 2025.4
Target Domain     : hackthissite.org (legal ethical hacking practice site)
Network           : Standard internet connection
Type              : Passive & Active Reconnaissance Lab
```

---

## ⚡ Practical Execution

### 🔹 Tool 1: WHOIS
```bash
whois hackthissite.org
```

<img width="1911" height="1035" alt="whois" src="https://github.com/user-attachments/assets/41ae1aff-d837-468a-9dfc-f7ef6eb2db6d" />

**What it does:** Queries the WHOIS database to retrieve domain registration details.

**What I Found:**
- Registrar: **Porkbun LLC**
- Creation Date: **2003-08-10** (20+ year old domain)
- Expiry Date: **2026-08-10**
- Name Servers: **BuddyNS** (c,f,g,h,j.ns.buddyns.com)
- DNSSEC: **Unsigned** ⚠️
- Domain Status: clientDeleteProhibited, clientTransferProhibited

**🔑 Key Observations:**
- DNSSEC is not enabled — domain is vulnerable to DNS cache poisoning
- Domain expires in 2026 — if not renewed, attacker could attempt registration
- BuddyNS name servers identified — can be probed for further DNS enumeration

---

### 🔹 Tool 2: nslookup
```bash
nslookup hackthissite.org
```
<img width="1904" height="997" alt="nslookup" src="https://github.com/user-attachments/assets/96614522-35b6-4824-adb6-2a1245e892ff" />

**What it does:** Queries DNS servers to resolve domain names to IP addresses.

**What I Found:**
```
137.74.187.100
137.74.187.101
137.74.187.102
137.74.187.103
137.74.187.104
```

**🔑 Key Observations:**
- 5 IP addresses returned — website uses **load balancing**
- All IPs in same subnet `137.74.187.x` — hosted in same data center (OVH, France)
- Non-authoritative answer — result came from cached DNS resolver

---

### 🔹 Tool 3: theHarvester
```bash
theHarvester -d hackthissite.org -b yahoo
```
<img width="1691" height="872" alt="theHarvester" src="https://github.com/user-attachments/assets/dfe091d6-e9bf-4a2c-9a59-dee9d64f45cf" />

**What it does:** Harvests emails, subdomains, and hosts from public search engines.

**What I Found:**
- 📧 Email: `sam@hackthissite.org`
- 🌐 Subdomains: `ctf`, `forum`, `irc`, `legal`, `mirror`.hackthissite.org
- Email format revealed: `firstname@domain.org`

**🔑 Key Observations:**
- Email can be used for **spear phishing** attacks
- IRC subdomain identified — old protocol, often has weaker security
- 5 subdomains found from a single Yahoo search — more sources would yield more results

---

### 🔹 Tool 4: Amass
```bash
amass enum -d hackthissite.org
amass enum -d hackthissite.org -v
```
<img width="1710" height="460" alt="Amass" src="https://github.com/user-attachments/assets/f814c892-1a89-46c2-8f43-253d330f9c45" />

**What it does:** Performs deep subdomain enumeration using DNS brute forcing,
certificate transparency logs, and 50+ public API integrations.

**What I Found:**
- Completed **2636/2636 enumeration tasks** at 100%
- Speed: 2 probes per second

> ⚠️ **Note:** Amass v5.0.0 completed all tasks successfully but due to
> a known output display bug, results were not printed to the terminal.
> The `amass db` command was removed in v5. DNSRecon was used as an
> alternative to display DNS results.

**🔑 Key Observations:**
- Amass v5 introduced breaking changes — `db` subcommand removed
- Tool still performed full enumeration in the background
- For visible output: use `amass enum -d target.com -o results.txt`

---

### 🔹 Tool 5: DNSRecon
```bash
dnsrecon -d hackthissite.org --lifetime 10
```
<img width="1708" height="750" alt="dnsrecon" src="https://github.com/user-attachments/assets/84d7b9f1-1033-4809-868f-0fd4c8361135" />

**What it does:** Enumerates all DNS records including A, MX, TXT, SPF, DMARC, and SRV.

**What I Found:**

| Record Type | Value | Significance |
|------------|-------|-------------|
| A Records | 137.74.187.100–104 | 5 load-balanced web servers |
| MX Records | aspmx.l.google.com | Uses Google Workspace email |
| SPF | mail.hackthissite.org | Dedicated internal mail server exists |
| DMARC | p=quarantine; pct=25 | Only 25% enforced — email spoofing possible |
| TXT | Harica verification token | SSL certificate from Harica CA |
| DNSSEC | Not enabled | Vulnerable to DNS spoofing |

**🔑 Key Observations:**
- DMARC is only 25% enforced — **75% of spoofed emails could bypass DMARC**
- `mail.hackthissite.org` is a separate mail server — can be targeted independently
- No DNSSEC + weak DMARC = high risk of email-based attacks

---

### 🔹 Tool 6: Recon-ng
```bash
recon-ng
workspaces create hackthissite
db insert domains
modules load recon/domains-hosts/bing_domain_web
options set SOURCE hackthissite.org
run
show hosts
```

**What it does:** Framework-based OSINT tool that uses modules to harvest data
from search engines, APIs, and public databases.

**What I Found — 21 Subdomains:**

<img width="1708" height="939" alt="Recon-ng" src="https://github.com/user-attachments/assets/4ff487f6-1fd1-497c-9ff8-f9b70eb34a09" />

| Subdomain | IP Address | Risk |
|-----------|-----------|------|
| `git.hackthissite.org` | 137.74.187.145 | 🔴 Critical — may expose source code |
| `api.hackthissite.org` | 137.74.187.115 | 🔴 Critical — weak API authentication |
| `email.hackthissite.org` | 172.239.57.117 | 🟠 High — dedicated mail server |
| `h5ai.hackthissite.org` | 185.199.111.153 | 🟠 High — file directory server |
| `irc.hackthissite.org` | 185.24.222.13 | 🟡 Medium — old IRC protocol |
| `stats.hackthissite.org` | 137.74.187.136 | 🟡 Medium — analytics server |
| `status.hackthissite.org` | 89.106.200.1 | 🟡 Medium — uptime monitor |

**🔑 Key Observations:**
- **git subdomain** is the most dangerous find — exposed Git repos can contain credentials
- **21 subdomains** discovered from just one Bing search module
- Each subdomain is an independent attack surface

---

### 🔹 Tool 7: Shodan

**Search Query:** `137.74.187.101`

**What it does:** Searches its database of internet-scanned devices to reveal
open ports, running services, and technologies — without touching the target.

<img width="1917" height="970" alt="shodan io" src="https://github.com/user-attachments/assets/5d315376-9151-4bdd-9147-77cd97d9f336" />


**What I Found:**
```
IP          : 137.74.187.101
Country     : France
City        : Dunkerque
ISP         : OVH SAS (AS16276)
Last Seen   : 2026-03-21
Tags        : onion (Tor hidden service)

Open Ports  : 80/TCP, 443/TCP
Server      : HackThisSite (custom header — hides actual web server)
HTTP/2      : Supported
HSTS        : Enabled
jQuery      : v1.81 (outdated ⚠️)
CORS        : Access-Control-Allow-Origin: * (dangerous ⚠️)
```

**🔑 Key Observations:**
- Site has a **.onion Tor address** — dark web presence confirmed
- **jQuery 1.81** is outdated — known XSS vulnerabilities exist
- **CORS wildcard** — any website can make cross-origin requests to this server
- Custom server header hides web server identity — intentional fingerprinting protection

---

### 🔹 Tool 8: Maltego

**What it does:** Visual intelligence tool that automatically maps relationships
between domains, IPs, emails, people, and organizations using transforms.

**What I Found:**
- Complete visual graph of all discovered entities
- Emails linked to employee names
- Subdomains linked to IP addresses and hosting infrastructure
- 20+ nodes mapped showing full attack surface visually

**🔑 Key Observations:**
- Maltego revealed the **interconnections** between all data collected by other tools
- Single transform run discovered emails, subdomains, IPs, and person entities
- Visual mapping helps prioritize targets — most connected nodes = most critical assets

---

## 🎯 Key Findings
```
Domain          : hackthissite.org
Registrar       : Porkbun LLC
IP Addresses    : 137.74.187.100 - 137.74.187.104 (5 servers, OVH France)
Email Found     : sam@hackthissite.org
Email Format    : firstname@domain.org
Subdomains      : 21 discovered (git, api, email, irc, forum, mirror...)
Mail Provider   : Google Workspace (Gmail)
Technology      : jQuery 1.81, HTTP/2, HSTS
Dark Web        : .onion address confirmed
DNSSEC          : Not enabled
DMARC           : Only 25% enforced
```

---

## 🔐 Security Analysis

### What Vulnerabilities Were Identified:

| Vulnerability | Location | Description |
|--------------|---------|-------------|
| Outdated jQuery 1.81 | Main website | Known XSS vulnerabilities (CVE history) |
| CORS Wildcard | Server headers | Any origin can make cross-site requests |
| DNSSEC Disabled | DNS config | DNS cache poisoning attack possible |
| DMARC 25% only | Email config | Email spoofing using org's domain possible |
| Git subdomain exposed | git.hackthissite.org | Source code, API keys, credentials at risk |
| Anonymous mail server | mail.hackthissite.org | Dedicated mail server publicly accessible |

### 🕵️ Attacker Perspective:

> With only this reconnaissance data, an attacker can:
> - Send spear phishing emails to `sam@hackthissite.org` using the leaked email format
> - Scan `git.hackthissite.org` for exposed repositories containing hardcoded credentials
> - Exploit jQuery 1.81 XSS vulnerabilities to hijack user sessions
> - Abuse CORS misconfiguration to steal data from authenticated users
> - Spoof emails from `@hackthissite.org` due to weak DMARC enforcement
> - Enumerate the IP range `137.74.187.0/24` on Shodan to find more exposed services

---

## ⚠️ Risk Assessment

| Finding | Risk Level | Reason |
|---------|-----------|--------|
| `git.hackthissite.org` exposed | 🔴 **Critical** | May expose source code, API keys, credentials |
| `api.hackthissite.org` exposed | 🔴 **Critical** | APIs often have weaker authentication |
| jQuery 1.81 outdated | 🔴 **Critical** | Known XSS vulnerabilities |
| CORS Allow-Origin: * | 🔴 **Critical** | Any site can make cross-origin requests |
| Email address exposed | 🟠 **High** | Enables targeted spear phishing |
| DMARC only 25% enforced | 🟠 **High** | Email spoofing partially possible |
| DNSSEC not enabled | 🟠 **High** | DNS cache poisoning possible |
| Multiple IRC servers | 🟡 **Medium** | Old protocol, weaker security configs |
| h5ai file server exposed | 🟡 **Medium** | May expose files publicly |
| OVH hosting details known | 🟡 **Medium** | Hosting provider can be targeted |
| WHOIS privacy not enabled | 🟢 **Low** | Registrant details partially visible |

---

## 🛡️ Mitigation / Prevention
```
1. Enable WHOIS Privacy      → Hide registrant contact details from public WHOIS
2. Enable DNSSEC             → Prevent DNS cache poisoning and spoofing
3. Fix DMARC Policy          → Change pct=25 to pct=100 and policy to reject
4. Secure Git Subdomain      → Place behind VPN or remove public access
5. Update jQuery             → Upgrade from 1.81 to latest stable version
6. Fix CORS                  → Replace * with specific trusted domains only
7. Deploy WAF                → Block reconnaissance tools and scanners
8. Monitor Shodan            → Regularly check what is publicly exposed
9. Retire IRC servers        → Decommission or firewall old legacy services
10. Security Awareness       → Train employees on phishing and social engineering
```

---

## 💡 What I Learned

- 🔍 **Passive reconnaissance is extremely powerful** — I collected 21 subdomains, 5 IPs,
  employee emails, server technologies, and physical location without sending a single
  packet to the target server directly

- 🛠️ **Different tools reveal different information** — WHOIS gave registrar data,
  nslookup gave IPs, theHarvester gave emails, Recon-ng gave subdomains, Shodan gave
  technologies — combining results gives a complete picture

- ⚠️ **Small misconfigurations create big risks** — DMARC at 25%, CORS wildcard, and
  an exposed git subdomain seem minor but can lead to full system compromise

- 🔐 **DNSSEC and DMARC are critical but often overlooked** — most organizations
  configure email but forget to enforce these DNS security features properly

- 🌐 **Shodan sees everything** — services exposed to the internet are indexed
  automatically; organizations must treat Shodan as an attacker's first tool

- 📊 **Maltego connects the dots** — visual mapping showed me relationships between
  data that would have taken hours to discover manually

---

## ✅ Conclusion

This reconnaissance assignment demonstrated that a complete intelligence profile
of an organization can be built using only publicly available tools and information.
Starting from a single domain name, I discovered 5 server IPs, 21 subdomains,
employee email addresses, web technologies, server location, and a Tor hidden service
— all without triggering a single security alert.

The findings highlight that **what organizations expose publicly is often more than
they realize**, and proactive security monitoring, proper DNS configuration, and
regular attack surface assessments are essential to minimize reconnaissance risk.

---

## 📁 Repository Structure
```
CEH-Assignment-1/
├── README.md               ← This file
├── screenshots/
│   ├── whois_output.png
│   ├── nslookup_output.png
│   ├── theharvester_output.png
│   ├── amass_output.png
│   ├── dnsrecon_output.png
│   ├── recon-ng_output.png
│   ├── shodan_output.png
│   └── maltego_graph.png
└── notes/
    └── findings_summary.txt
```

---

## ⚠️ Disclaimer

> This assignment was performed in a **legal, controlled lab environment** on
> **hackthissite.org** — a website explicitly designed for ethical hacking practice.
> All tools and techniques demonstrated here are for **educational purposes only**.
> Never perform reconnaissance or any form of hacking on systems you do not have
> explicit written permission to test. Unauthorized scanning is illegal under the
> Computer Fraud and Abuse Act (CFAA) and equivalent laws worldwide.

---

<div align="center">

**Made with ❤️ for CEH Lab Practice**

![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)
![Cybersecurity](https://img.shields.io/badge/Cybersecurity-Lab-blue?style=for-the-badge)

</div>
