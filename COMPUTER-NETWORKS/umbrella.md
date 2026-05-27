# ☂️ Cisco Umbrella: DNS Security & Secure Internet Gateway (SIG)

Cisco Umbrella evolved from the OpenDNS acquisition. Its primary DNS resolvers are accessible globally at **`208.67.222.222`** and **`208.67.220.220`**. 

These are not just two servers; they represent a massive global infrastructure utilizing **Anycast** routing. Depending on where a host is physically located, the internet automatically routes their query to the closest and fastest Umbrella data center.

But Umbrella is much more than just blocking bad URLs. It is a predictive security engine.

---

### 🛠️ Deployment Methods (How to force traffic to Umbrella)

We can enforce the use of Umbrella in our network through several methods:

1.  **Network-Level (NAT/DHCP):** We can configure our local DHCP to hand out Umbrella IPs, or use Destination NAT on the firewall to intercept port 53 traffic and force it to Umbrella.
2.  **Roaming Clients:** Installing the Umbrella Roaming Client (or the Umbrella module within Cisco Secure Client) on Windows/macOS laptops.
3.  **Mobile Devices:** Using MDM (Mobile Device Management) security connectors for iOS/Android.
4.  **Virtual Appliances (The Golden Standard for Enterprises):** 
    If you just point your router to Umbrella, the cloud only sees your single Public IP. You lose all user visibility. 
    *The Solution:* You deploy lightweight Umbrella Virtual Appliances (VAs) inside your LAN. You point your DHCP to these VAs. 
    *   If a PC asks for an internal domain (`printer.corp.local`), the VA forwards it to your internal Active Directory DNS.
    *   If a PC asks for `bad-website.com`, the VA forwards it to the public Umbrella cloud. 
    *   *The Benefit:* The VA attaches internal metadata to the query! Umbrella now sees the internal private IP, and if integrated with AD, it even sees the exact First and Last Name of the user making the query.

---

### 🧠 Advanced Algorithms & Machine Learning

Umbrella doesn't just wait for a domain to be reported as malicious. It actively queries Authoritative DNS servers and analyzes global logs using Machine Learning to predict attacks before they happen.

#### 1. Analyzing Authoritative DNS Behavior
*   **Newly Staged Infrastructures:** Umbrella notices that 50 brand-new domains were just registered on a single authoritative server. Why? Because hackers often stage massive infrastructure in advance for an upcoming phishing campaign. Umbrella flags them proactively.
*   **Fast Flux Domains:** Umbrella sees that the `A record` for a domain changes its IP address every few minutes. Hackers do this to hide Command & Control (C2) botnet servers behind a constantly shifting wall of compromised home PCs.
*   **DNS Hijacking:** A legitimate, trusted domain suddenly changes its assigned `A record` to a strange IP hosted in a high-risk country. Umbrella detects the anomaly and flags the domain takeover.

#### 2. Machine Learning Models
*   **Co-occurrence Model:** A user visits a legitimate site (e.g., `yahoo.com`), but in the background, a hidden script makes a query to a risky domain. Umbrella learns this "co-occurrence" pattern and flags the hidden domain as a malicious payload delivery system.
*   **Traffic Spike Model:** Umbrella compares the current query volume for a domain against its historical baseline. A sudden, massive spike in queries for an unknown domain usually indicates a malware outbreak or a botnet waking up.
*   **Predictive IP Space Monitoring:** Let's say the Traffic Spike model catches a malicious domain (`virus-update.com`). The Predictive model looks deeper. It notices this domain belongs to a specific authoritative DNS server (`ns1.cheap-shady-hosting.net`). It discovers 500 other freshly registered domains on that exact same server. The model finds the common denominator and predicts that all 500 domains are part of a massive, upcoming phishing campaign, blocking them all instantly.

---

### 🕵️‍♂️ The "Killer Feature": From DNS to Proxy (SIG)

**THIS IS THE MOST IMPORTANT CONCEPT OF UMBRELLA!**

Umbrella's primary function is **Domain Reputation (Layer 3/4)**. If a domain is perfectly safe, Umbrella returns the IP, and the user connects directly. If it's pure evil, Umbrella returns a block page.

But what if the domain is **"Risky"** or **"Unknown"**? 
Umbrella silently activates its **SIG (Secure Internet Gateway)** feature. In other words, Umbrella transforms from a simple DNS server into a full-fledged **Cloud-Based Proxy**.

Instead of returning the real IP of the risky website, Umbrella returns the IP of its own Proxy servers. The host connects to the Proxy at **Layer 7 (Application Layer)**.

Since 99% of web traffic today is encrypted (HTTPS), we must deploy the Umbrella Root Certificate to our corporate machines (e.g., via GPO). This allows the Umbrella Proxy to decrypt the traffic, inspect it, and re-encrypt it. 
Once inside Layer 7, Umbrella can perform deep inspection:

*   **Web Reputation (Score -10 to +10):** It evaluates not just the domain name, but the HTTP headers, the specific server behavior, and the exact IP hosting the content.
*   **URL Reputation:** It checks the specific, deep link (e.g., `site.com/hidden/malware.exe`) rather than just the main domain.
*   **File Inspection (The Facebook/Pastebin Problem):** This is where SIG shines. Platforms like Facebook, Dropbox, or Pastebin host both safe content and malicious payloads. We cannot block Facebook entirely! Because SIG operates at Layer 7, it allows the user to browse Facebook, but if they click a malicious file, SIG extracts the file, calculates its hash, checks it against signatures (and Cisco Secure Malware Analytics), and blocks *only* the malicious download while keeping the rest of the site functional.

---

### 🔎 Cisco Umbrella Investigate

Finally, it is worth mentioning **Cisco Investigate**. Think of it as a "Google Search for Threat Intelligence". It provides access to the massive, global intelligence database gathered by Cisco Talos. Security analysts can type in an IP, domain, or file hash, and receive a rich, detailed history of its reputation, who owns it, and what other malicious infrastructure it is tied to.