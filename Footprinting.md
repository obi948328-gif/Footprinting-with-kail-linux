# Footprinting-with-kail-linux
# Penetration Testing Report: Footprinting & Network Scanning Phases
**Document ID:** W2-PM-FINAL | **Program:** Cybersecurity Internship | **Organization:** Networkwalks

| Report Metadata | Details |
| :--- | :--- |
| **Pentester Name** | Emmanuel John (Obi Chukwuemerie Samuel) |
| **Program / Batch** | B082-Networkwalks |
| **Date** | 14 November 2026 |
| **Modules Completed** | W2-PM1 (Multiple Kali Tools), W2-PM5 (Zenmap Scanning) |
| **Target Scope** | 1. `networkwalks.com` (Written permission secured)<br>2. Local LAN Network (Self-owned) |
| **Phases Covered** | Phase 1: Reconnaissance & Footprinting<br>Phase 2: Scanning & Network Discovery |

---

## 1. Liability Disclaimer
> **Authorization Notice:** All activities documented in this report were performed strictly on systems where explicit written permission was secured or on self-owned local devices. These materials are for educational and research purposes only. Unauthorized access to computer systems is illegal; every action taken with this knowledge is the sole responsibility of the user.

---

## 2. Introduction & Objective
This report documents Week 2 of the Cybersecurity & Ethical Hacking internship at Networkwalks. It demonstrates how an attacker transitions from passive/active public information gathering (Footprinting) to mapping live hosts on an internal network (Scanning). 
* **Phase 1 (W2-PM1):** Conducted reconnaissance against `networkwalks.com` using six Kali Linux CLI utilities.
* **Phase 2 (W2-PM5):** Performed local network discovery and topology mapping using Zenmap on Windows.

---

## 3. Tools & Environment
| Tool / OS | Purpose |
| :--- | :--- |
| **Kali Linux & Windows** | Operating systems used to execute reconnaissance and scanning |
| `whois` | Retrieve domain registration details, registrar info, and name servers |
| `whatweb` | Fingerprint web technologies, CMS versions, plugins, and server headers |
| `nslookup` | Resolve domain names to IPv4 addresses via DNS |
| `curl -i` | Inspect HTTP response headers and redirect behaviors |
| `wafw00f` | Detect and identify Web Application Firewalls (WAF) |
| `dnsrecon` | Enumerate DNS records (`SOA`, `NS`, `MX`, `A`, `TXT`, `SRV`) and DNS versions |
| **Zenmap (Nmap GUI)** | Discover live hosts, IP/MAC addresses, and generate network topology |

---

## 4. Phase 1: Footprinting & Reconnaissance (`networkwalks.com`)

### 4.1 Domain Registration (`whois`)
* **Command:** `whois networkwalks.com`[cite: 1]
* **Key Findings:** 
  * **Registrar:** GoDaddy.com, LLC (Created: `2019-11-06`, Expires: `2027-11-06`)[cite: 1, 2]
  * **Registrant Privacy:** Protected via *Domains By Proxy, LLC*[cite: 2]
  * **Name Servers:** `NS6135.HOSTGATOR.COM`, `NS6136.HOSTGATOR.COM`, `NS29.DOMAINCONTROL.COM`, `NS30.DOMAINCONTROL.COM`[cite: 2]
  * **DNSSEC:** Unsigned[cite: 1, 2]
* **Attacker Relevance:** Reveals the hosting provider (HostGator) and confirms domain privacy is active, preventing direct social engineering via registrant contact info.
* **Evidence:**  
  ![WHOIS Output](./screenshots/01-whois.png)

### 4.2 Web Technology Fingerprinting (`whatweb`)
* **Command:** `whatweb networkwalks.com`
* **Key Findings:**
  * **Web Server & Caching:** Apache with `x-nginx-cache` and `x-endurance-cache-level`[cite: 3, 4]
  * **CMS & Plugins:** WordPress `7.1.4`, WordPress Download Manager `3.3.76`, Bootstrap `7.1.4`[cite: 3, 4]
  * **Contact Exposed:** `info@networkwalks.com`[cite: 3, 4]
* **Attacker Relevance:** Exposing exact CMS and plugin versions allows attackers to search exploit databases (e.g., CVE, Exploit-DB) for known vulnerabilities in those specific releases.
* **Evidence:**  
  ![WhatWeb Output](./screenshots/02-whatweb.png)

### 4.3 DNS Resolution (`nslookup`)
* **Command:** `nslookup networkwalks.com`[cite: 5, 6]
* **Key Findings:** Resolved `networkwalks.com` to IPv4 address `192.232.216.135` (queried via Google DNS `8.8.8.8`)[cite: 5, 6].
* **Attacker Relevance:** Maps the domain name to its hosting IP address, establishing a target IP for port scanning and infrastructure mapping.
* **Evidence:**  
  ![Nslookup Output](./screenshots/03-nslookup.png)

### 4.4 HTTP Response Headers (`curl`)
* **Command:** `curl -i networkwalks.com`[cite: 7]
* **Key Findings:**
  * Returns `HTTP/1.1 301 Moved Permanently` redirecting to `https://networkwalks.com/`[cite: 7].
  * Exposes `Server: Apache`, `X-Redirect-By: WordPress - Really Simple Security`, `X-nginx-cache: WordPress`, and sets an `HttpOnly` session cookie (`__wpdm_client`)[cite: 7].
* **Attacker Relevance:** Confirms HTTPS enforcement while leaking backend caching architecture and active WordPress security plugins.
* **Evidence:**  
  ![Curl Output](./screenshots/04-curl.png)

### 4.5 WAF Detection (`wafw00f`)
* **Command:** `wafw00f networkwalks.com`[cite: 8]
* **Key Findings:** Confirmed the site is protected behind **ModSecurity (SpiderLabs) WAF**[cite: 8].
* **Attacker Relevance:** Alerts an attacker that automated injection/exploitation payloads will likely be filtered or blocked, requiring careful evasion or rate-limiting awareness.
* **Evidence:**  
  ![Wafw00f Output](./screenshots/05-wafw00f.png)

### 4.6 DNS Enumeration (`dnsrecon`)
* **Command:** `dnsrecon -d networkwalks.com`[cite: 9]
* **Key Findings:**
  * **DNS Software Version:** Exposed Bind Version `"9.16.23-RH"` on `50.87.144.87` and `192.232.216.131`[cite: 9].
  * **Mail & Verification Records:** `MX mail.networkwalks.com` (`192.232.216.135`), Google Site Verification TXT record, and an SPF policy (`v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all`)[cite: 9].
  * **Service Records:** 8 `_autodiscover._tcp` SRV records pointing to `cpanelemaildiscovery.cpanel.net`[cite: 9].
* **Attacker Relevance:** Exposes mail server infrastructure, cPanel usage, and DNS server software versions, aiding in targeted phishing or DNS-level reconnaissance.
* **Evidence:**  
  ![DNSRecon Output](./screenshots/06-dnsrecon.png)

---

## 5. Phase 2: Network Scanning with Zenmap

### 5.1 Execution Steps
1. Identified local IPv4 address and subnet mask on Windows via `ipconfig`.
2. Launched **Zenmap** and executed a **Ping Scan** (`nmap -sn <subnet>`) across the local LAN to discover active hosts without intrusive port probing.
3. Recorded active host IPs and MAC addresses (Vendor OUI identification).
4. Generated and exported the visual network topology map with the legend enabled.

### 5.2 Discovered Hosts *(Replace with your actual LAN scan results)*
| Host # | IP Address | MAC Address | Status |
| :--- | :--- | :--- | :--- |
| 1 | `10.0.0.1` | `XX:XX:XX:XX:XX:01` | Up (Gateway) |
| 2 | `10.0.0.4` | `XX:XX:XX:XX:XX:04` | Up |
| 3 | `10.0.0.5` | `XX:XX:XX:XX:XX:05` | Up |
| 4 | `10.0.0.19` | `XX:XX:XX:XX:XX:19` | Up |

* **Topology Evidence:**  
  ![Zenmap Topology](./screenshots/07-zenmap-topology.png)

---

## 6. Risk Analysis & Impact
*Note: The findings below represent reconnaissance observations rather than confirmed exploitable vulnerabilities.*

| # | Risk / Finding | Observation | Potential Impact | Risk Level |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **CMS & Plugin Versions Exposed** | WhatWeb identified WordPress `7.1.4` and WP Download Manager `3.3.76`[cite: 3, 4] | Enables attackers to map software versions against public CVE databases | **Medium** |
| 2 | **DNS Software Version Exposed** | DNSRecon revealed Bind `"9.16.23-RH"`[cite: 9] | Assists attackers in fingerprinting underlying OS (Red Hat) and DNS daemon | **Medium** |
| 3 | **DNSSEC Unsigned** | WHOIS confirmed DNSSEC is disabled[cite: 1, 2] | Increases theoretical susceptibility to DNS spoofing/cache poisoning | **Low** |
| 4 | **HTTP Header Disclosure** | `curl -i` revealed Apache, Nginx cache, and security plugin headers[cite: 7] | Aids infrastructure profiling for targeted enumeration | **Low** |
| 5 | **Server IP & Mail Records Visible** | Resolved origin IP (`192.232.216.135`) and cPanel mail records[cite: 5, 9] | Exposes hosting infrastructure and mail routing paths | **Low** |
| 6 | **Multiple Active LAN Hosts** | Zenmap discovered multiple live devices on the local subnet | Unmonitored or rogue devices could reside on the network if unverified | **Medium** |

---

## 7. Recommendations & Remediation
1. **Obfuscate Version Banners:** Configure Apache, WordPress, and DNS (`version "none";` in BIND `named.conf`) to suppress software version strings in headers and DNS queries.
2. **Patch Management:** Maintain regular update schedules for WordPress core (`7.1.4`), plugins (`WP Download Manager`), and server packages.
3. **Enable DNSSEC:** Configure DNSSEC with the domain registrar to cryptographically sign DNS records and prevent spoofing.
4. **Harden HTTP Headers:** Strip unnecessary `X-Redirect-By` and caching headers while implementing strict security headers (`Strict-Transport-Security`, `Content-Security-Policy`).
5. **Maintain WAF Tuning:** Keep ModSecurity active and regularly review logs for blocked reconnaissance or exploit patterns.
6. **Routine Internal Asset Audits:** Conduct scheduled internal network scans to verify authorized MAC/IP inventories and detect rogue devices.

---

## 8. Key Takeaways
* **Passive vs. Active Recon:** Combining OSINT tools (`whois`, `dnsrecon`) with active probing (`whatweb`, `wafw00f`) builds a comprehensive profile of a target's defensive posture before a single exploit is attempted.
* **Defense-in-Depth:** Even when a WAF (ModSecurity) is active[cite: 8], misconfigured DNS banners or verbose HTTP headers can still leak valuable architectural details[cite: 7, 9].

---
**Author:** OBi Samuel| [LinkedIn](https://www.linkedin.com/in/obi-chukwuemerie-samuel-357807406) 
