# networkwalks-week2-footprinting-scanning

Networkwalks Cybersecurity Internship, Week 2 Project: footprinting and reconnaissance with Kali tools, and network scanning with Zenmap.

🔍 Footprinting, Reconnaissance & Network Scanning
Project Modules: W2-PM1 (Footprinting with Multiple Kali Tools) and W2-PM5 (Network Scanning with Zenmap)
Target Domain: networkwalks.com (authorised lab target)
Second Target: My own home network
Operating Systems: Kali Linux (footprinting), Windows PC (Zenmap)
Assessment Type: Footprinting, Web Fingerprinting, DNS Enumeration and Network Discovery
Author: Eke Margret Uloma
Date: 11 November 2026

---
📌 1. Project Overview
This project was completed as part of my Cybersecurity & Ethical Hacking internship at Networkwalks.

It has two parts. 
In the first part, I carried out footprinting and reconnaissance on the authorised target domain, networkwalks.com, using six Kali Linux tools.
In the second part, I used Zenmap to scan my own home network and find the live hosts on it.
Together, the two parts show how an attacker moves from collecting public information about a target to mapping the live devices on a network.

🎯 Objectives
Identify the target domain's registration details.

Discover the technologies running the website.

Resolve the domain name to its IP address.

Examine the HTTP response headers.

Detect whether a Web Application Firewall (WAF) protects the site.

Enumerate the DNS records of the domain.

Discover the live hosts on my own local network and save the network topology.

Document the findings and what they mean for security.

> 🔒 **Privacy note:
>  The private IP addresses, MAC addresses and device names of my home network are masked in this repository on purpose. The public IP addresses and DNS records shown for networkwalks.com belong to the authorised lab target and are part of the findings.
> 
---
🛠️ 2. Tools and Technologies

Tool	Purpose	Activity type

WHOIS	Domain registration lookup	Passive

WhatWeb	Website technology fingerprinting	Active

nslookup	Domain-to-IP resolution	DNS lookup

curl -I	HTTP response header inspection	Active

wafw00f	Web Application Firewall detection	Active

dnsrecon	DNS record enumeration	DNS reconnaissance

Zenmap (Nmap GUI)	Live host discovery on a local network	Active

Windows CMD (ipconfig)	Local IP and MAC address identification	Local


---
🔎 3. Part A: Footprinting and Reconnaissance (W2-PM1)

Task 1: 
WHOIS, Domain Registration Information

Objective: Find the publicly available registration information of the target domain.

Command executed:
```bash
whois networkwalks.com
```
Results:

Information	Findings

Domain name	networkwalks.com

Registrar	GoDaddy.com, LLC

Registration date	6 November 2019

Last updated	12 November 2025

Expiry date	6 November 2027

Name servers	NS6135.HOSTGATOR.COM, NS6136.HOSTGATOR.COM

DNSSEC	Unsigned

Registrant	Registration Private (Domains By Proxy, LLC)

Analysis: The domain is registered through GoDaddy and uses HostGator name servers, which shows who hosts the site. The registrant's personal details are hidden behind a domain privacy service. DNSSEC is unsigned, so the DNS answers are not digitally signed.

Screenshots:

![WHOIS output part 1](images/whois-1.png)

![WHOIS output part 2](images/whois-2.png)

![WHOIS output part 3](images/whois-3.png)

![WHOIS output part 4](images/whois-4.png)

![WHOIS output part 5](images/whois-5.png)

![WHOIS output part 6](images/whois-6.png)

Key learning: WHOIS helps analysts identify the registrar, name servers and registration timeline of a domain.

---
Task 2: WhatWeb, Website Technology Fingerprinting
Objective: Identify the technologies used by the website.
Command executed:
```bash
whatweb networkwalks.com
```
Results:

Technology	Findings

Web server	Apache

CMS	WordPress 7.1.4

Plugin	WordPress Download Manager 3.3.76

JavaScript library	jQuery 3.7.1

IP address	192.232.216.135

HTTP redirect	301 Moved Permanently (http to https)

HTTPS response	200 OK

Page title	Networkwalks Academy

Analytics	Google Tag Manager

Contact email	info@networkwalks.com

Analysis: WhatWeb identified Apache as the web server and WordPress as the CMS, together with a download manager plugin and jQuery.

The site redirects http traffic to https, so it supports encrypted connections. Knowing the exact software and versions helps an analyst decide which components need a closer security review.

Screenshot:

![WhatWeb results](images/whatweb.png)

Key learning: WhatWeb exposes the technology stack of a website, including software versions.

---
Task 3: nslookup, Domain-to-IP Resolution
Objective: Resolve the domain name to its IP address.
Command executed:
```bash
nslookup networkwalks.com
```
Results:
```text
Server:   [private DNS resolver on my home network, redacted]
Non-authoritative answer:
Name:     networkwalks.com
Address:  192.232.216.135
```
Analysis: nslookup resolved networkwalks.com to the IPv4 address 192.232.216.135. The answer was non-authoritative, which means it came from a resolver and not directly from the domain's own name server. This matches the IP address that WhatWeb reported.
Screenshot:
![nslookup result with the private resolver address redacted](images/nslookup-redacted.png)
Key learning: nslookup shows how the Domain Name System turns a domain name into the IP address used on the network.
---
Task 4: curl, HTTP Response Headers
Objective: Read the HTTP response headers returned by the website.
Command executed:
```bash
curl -I https://networkwalks.com
```
Results:
Header	Observed value
Status	HTTP/2 200
Server	Apache
Content-Type	text/html; charset=UTF-8
Referrer-Policy	no-referrer-when-downgrade
X-Nginx-Cache	WordPress
X-Endurance-Cache-Level	0
Cookie	__wpdm_client (Secure; HttpOnly)
Date	Wed, 11 Nov 2026 08:29:05 GMT
The response also included `permissions-policy` and `link` headers, and the `link` header pointed to the WordPress REST API at `/wp-json/`.
Analysis: Status 200 means the server processed the request successfully. The cookie is set with the Secure and HttpOnly attributes, which protect it from being sent over unencrypted connections and from being read by JavaScript. The headers also reveal the web server, the caching setup and the WordPress API endpoint.
Screenshot:
![curl response headers](images/curl.png)
Key learning: HTTP headers reveal information about the server and its security settings without loading the full page.
---
Task 5: wafw00f, Web Application Firewall Detection
Objective: Find out whether a Web Application Firewall protects the website.
Command executed:
```bash
wafw00f networkwalks.com
```
Results:
```text
[*] Checking https://networkwalks.com
[+] The site https://networkwalks.com is behind ModSecurity (SpiderLabs) WAF.
[~] Number of requests: 2
```
Analysis: wafw00f detected the ModSecurity (SpiderLabs) WAF after only two requests. A WAF filters and monitors HTTP requests and can help block attacks such as SQL injection and cross-site scripting. Detecting a WAF does not prove that the site is safe from every web attack.
Screenshot:
![wafw00f result](images/wafw00f.png)
Key learning: wafw00f shows whether a firewall sits in front of a website and which product it is.
---
Task 6: dnsrecon, DNS Enumeration
Objective: Enumerate the DNS records of the domain.
Command executed:
```bash
dnsrecon -d networkwalks.com
```
Results (notable records):
Record	Finding
NS	HostGator name server (ns6135.hostgator.com)
DNS software	BIND 9.16.23-RH
MX	mail.networkwalks.com (192.232.216.135)
A	networkwalks.com (192.232.216.135)
TXT (SPF)	`v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all`
TXT	Google site verification record
SRV	8 records for cPanel email autodiscovery
Analysis: dnsrecon mapped the mail and name server setup of the domain. The SPF record lists which servers may send email for the domain. The SRV records show that cPanel email autodiscovery is used. The BIND version is also visible, which is information that an analyst would note when checking the DNS infrastructure.
Screenshots:
![dnsrecon output, middle section](images/dnsrecon-1.png)
![dnsrecon output, end of enumeration](images/dnsrecon-2.png)
Key learning: dnsrecon helps analysts see the full DNS and mail configuration of a domain in one scan.
---
🌐 4. Part B: Network Scanning with Zenmap (W2-PM5)
Objective: Find the live hosts on my own local network and save the network topology.
Steps followed:
Installed Nmap and Zenmap on my Windows PC.
Ran `ipconfig` in Command Prompt to find my local IP address and my LAN subnet.
Entered the subnet as the target in Zenmap, selected the Ping scan profile and ran the scan.
Recorded the live hosts with their IP addresses and MAC addresses.
Ran `ipconfig /all` to read the MAC address (Physical Address) of my own PC, because Zenmap did not show it for the local machine.
Opened the Topology tab, turned on the legend and saved the topology as a PDF.
Results:
Item	Finding
Network	Private home LAN, 192.168.x.0/24 (masked)
Scan profile	Ping scan (`nmap -sn`)
Addresses scanned	256
Live hosts	4

Host	Role	IP address	MAC address

1	Wi-Fi router (default gateway)	masked	recorded, masked

2	Mobile device 1	masked	recorded, masked

3	Mobile device 2 (hotspot)	masked	recorded, masked

4	My own Windows PC	masked	recorded, masked (from `ipconfig /all`)

Analysis: The ping scan found four live hosts out of the 256 addresses in the subnet: the router, two mobile devices and my own PC.

I recognised every device, so no unknown device was present on the network at the time of the scan.

The topology shows my PC (localhost) in the centre connected to the four hosts.

Screenshot (names and IP addresses masked):

![Zenmap topology with names and IP addresses masked](images/zenmap-topology-redacted.png)

Key learning: A ping scan is a quick way to see which devices are active on a network, and it is a useful first step before more detailed scanning.

---
📊 5. Summary of Findings

Tool	Major finding	Result

WHOIS	GoDaddy registrar, HostGator name servers, privacy service, DNSSEC unsigned	Successful

WhatWeb	Apache, WordPress 7.1.4, WP Download Manager 3.3.76, jQuery 3.7.1	Successful

nslookup	Resolved IP: 192.232.216.135	Successful

curl	HTTP/2 200, server and cache headers, `/wp-json/` link	Successful

wafw00f	ModSecurity (SpiderLabs) WAF detected	Successful

dnsrecon	NS, MX, SPF, TXT and SRV records, BIND 9.16.23-RH	Successful

Zenmap	4 live hosts on my home network, topology saved	Successful

Security observations

Domain privacy: The domain uses a privacy service, so the owner's personal details are not public.

Technology disclosure: WhatWeb exposed the CMS, plugin and library versions. This helps defenders keep an inventory of what is visible from outside, and it tells attackers what to research.

HTTPS: The site redirects http to https and sets the Secure and HttpOnly flags on its cookie.

WAF: ModSecurity was detected, so an application-layer protection is in place.

DNS: DNSSEC is unsigned, and the DNS software version and mail records are publicly visible.

Local network: All four live hosts on my home network were devices I recognised.

These observations come from reconnaissance and scanning only. They are not confirmed vulnerabilities.

---
🧠 6. Challenges Encountered

Long WHOIS output: The output did not fit on one screen, so I captured it in several screenshots.

At one point I pasted copied output back into the terminal by mistake and got "command not found" errors. This did no harm, and I repeated the command to get a clean result.

Save dialog in Zenmap: The Save button was not visible in the Save Topology window.

I solved it by typing the file name with `.pdf` and pressing Enter.

MAC address of my own PC: Zenmap did not show a usable MAC address for the machine I scanned from. I used `ipconfig /all` and read the Physical Address of my network adapter instead.

---
🎓 7. Skills and Knowledge Gained
Domain footprinting and WHOIS investigation.

Website technology fingerprinting.

DNS resolution and enumeration.

HTTP response header analysis.

Web Application Firewall identification.

Live host discovery and topology mapping with Zenmap.

Using Kali Linux and Windows command-line tools.

Documenting findings clearly, and protecting private details when publishing results.

---
🏁 8. Conclusion

This project gave me practical experience of the first two phases of ethical hacking: reconnaissance and scanning. With six Kali Linux tools I collected the registration details, web technologies, IP address, HTTP headers, firewall and DNS records of the authorised target.

With Zenmap I found the live hosts on my own network and saved its topology.

The exercise showed me how much information is public before any attack starts, and why defenders should check what they expose.

It also taught me to report observations honestly, to keep them separate from confirmed vulnerabilities, and to protect private network details when sharing results.

---
⚖️ Ethical Disclaimer
This work was done for educational purposes as part of the Networkwalks cybersecurity internship.

The footprinting was performed on the authorised target domain networkwalks.com, and the network scanning was performed on my own network. The activities were limited to reconnaissance, information gathering and host discovery. No exploitation, credential attacks or unauthorised access attempts were made.

---
Author: Eke Margret Uloma
Programme: Networkwalks Cybersecurity Internship
Project: Week 2 | Footprinting, Reconnaissance and Network Scanning
Platforms: Kali Linux, Windows
