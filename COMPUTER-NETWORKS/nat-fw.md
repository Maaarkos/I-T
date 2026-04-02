# 🗺️ The Ultimate Cisco NAT Guide (FMC & ASA)

In the Cisco world, NAT is often confusing because documentation mixes **WHAT** we are changing (Source vs. Destination) with **HOW** we configure it (Auto NAT vs. Manual NAT) and legacy terminology from the old ASA days (Inside vs. Outside NAT).

Let's organize this chaos once and for all.

### 📊 The NAT Cheat Sheet (Memorize This)

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto;">
+--------------------------------------------------------------------------------+
|                       THE CISCO NAT MASTER MATRIX                              |
+-----------------------+--------------------+-----------------------------------+
| WHAT ARE WE CHANGING? | FMC CONFIG METHOD  | REAL-WORLD EXAMPLE                |
+-----------------------+--------------------+-----------------------------------+
| SOURCE (Who sent it?) | Auto NAT (Object)  | Dynamic PAT (Office to Internet)  |
|                       | Auto NAT (Object)  | Static NAT (1-to-1 Mail Server)   |
+-----------------------+--------------------+-----------------------------------+
| DESTINATION (To who?) | Auto NAT (Object)  | Port Forwarding (Inbound Web)     |
|                       | Manual NAT (Sec 1) | IP Hijacking (Redirect Dead IP)   |
+-----------------------+--------------------+-----------------------------------+
| BOTH (Src & Dst)      | Manual NAT (Sec 1) | Twice NAT / Policy NAT            |
+-----------------------+--------------------+-----------------------------------+
| NOTHING (Bypass)      | Manual NAT (Top)   | Identity NAT (For IPsec VPNs)     |
+-----------------------+--------------------+-----------------------------------+
</pre>

---

### 1️⃣ CHANGING THE SOURCE (Source NAT / Legacy: "Inside NAT")
**Goal:** Hide the true sender from the outside world.

*   **A) Static NAT (1-to-1)**
    *   **Direction:** Inside -> Outside.
    *   **How to configure:** Usually as **Auto NAT** (configured directly inside the Host Object).
    *   **Example:** Our internal Mail Server (`192.168.10.100`) is permanently mapped to its own, dedicated public IP address (`80.80.80.89`).
*   **B) Dynamic NAT (Pool-to-Pool) - `[LEGACY]`**
    *   **Direction:** Inside -> Outside.
    *   **Example:** You have 50 computers and a pool of 10 public IPs. First come, first served. *Today, this technology is practically dead.*
*   **C) Dynamic PAT (Many-to-1)**
    *   **Direction:** Inside -> Outside.
    *   **How to configure:** Always as **Auto NAT** (Object NAT). This is the most popular rule in the world.
    *   **Example:** The entire office (`/24` subnet) goes out to the internet hiding behind the single public IP of the firewall. The firewall distinguishes sessions using Port numbers.

---

### 2️⃣ CHANGING THE DESTINATION (Destination NAT / UN-NAT / Legacy: "Outside NAT")
**Goal:** Change the recipient the packet was originally flying towards. *(This happens at the very ingress of the firewall!)*

*   **A) Static PAT (Port Forwarding)**
    *   **Direction:** Outside -> Inside.
    *   **How to configure:** Usually as **Auto NAT** (inside the server Object).
    *   **Example:** A client from the internet types our public IP into their browser. The firewall catches it and changes the destination on-the-fly to the private IP of the Web Server in the DMZ.
*   **B) Static Destination NAT (IP Hijacking / Redirection)**
    *   **Direction:** Inside -> Outside.
    *   **How to configure:** As **Manual NAT** (Section 1 in FMC).
    *   **Example 1 (The Dead IP):** A legacy machine stubbornly sends logs to a dead, hardcoded IP (`1.1.1.1`). The firewall intercepts it and silently replaces the destination with the new server (`9.9.9.9`).
    *   **Example 2 (The Clever Admin):** Employees in the office try to bypass corporate web filters by manually setting Google DNS (`8.8.8.8`) in Windows. You, as a clever admin, create a UN-NAT rule: *"Any traffic heading to 8.8.8.8 on port 53 (DNS), intercept it on the fly and change the destination to our internal corporate DNS server (192.168.10.200)."* The user thinks they are asking Google, but they are actually asking your server, which happily blocks TikTok. 😎

> **💡 ISP/Home Router Trivia: The "DMZ" Feature**
> The so-called "DMZ" feature found on typical ISP ONT devices (Optical Network Terminals - modem + media converter) is actually just a massive **Destination NAT (Outside NAT)**! 
> The ONT translates *everything* coming to the public IP directly to a specific private IP (which is usually the WAN port of your own router, or your PC if you don't have a router). 
> **PRO-TIP:** Instead of using the ISP's "DMZ", it is always best to call your ISP and ask them to switch their device into a transparent **Bridge Mode** (making it just a dumb pipe). This allows your own firewall to get the public IP directly on its interface, completely avoiding the nightmare of Double NAT!

---

### 🔀 3️⃣ TARGET-DEPENDENT CHANGE (Twice NAT / Policy NAT)
**Goal:** Perform NAT, but *only* under very specific, strict conditions.

*   **A) Manual NAT (Section 1 in FMC)**
    *   **Direction:** Any.
    *   **How it works:** In a single rule, the firewall checks **BOTH** the Source and the Destination before making a decision.
    *   **Example (The Bank Whitelist):** Imagine a partner Bank's server has a strict firewall whitelist. They require all our API connections to come from a specific IP, e.g., `192.168.2.99`. We configure a Manual NAT (Policy NAT) rule: *"If traffic is going specifically to the Bank's IP, translate our source to 192.168.2.99."* Meanwhile, a standard Auto NAT handles all other regular internet traffic for the office.

---

### 🛑 4️⃣ NO CHANGE (Identity NAT / NAT Exempt)
**Goal:** Protect specific traffic from being caught by another, broader NAT rule (e.g., protecting VPN traffic from being PAT'ed to the internet).

*   **A) Static Identity NAT**
    *   **Direction:** Inside -> VPN_Zone.
    *   **How to configure:** ALWAYS as **Manual NAT** (Placed at the very top of Section 1 - "Above Rule 1").
    *   **Example:** *"If you are going to LAN-B through the IPsec VPN tunnel, translate your address to... your own address (do not change anything)."*