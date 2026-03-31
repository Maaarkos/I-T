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
    *   **Example:** A legacy machine stubbornly sends logs to a dead, hardcoded IP (`1.1.1.1`). The firewall intercepts it and silently replaces the destination with the new server (`9.9.9.9`).

---

### 🔀 3️⃣ TARGET-DEPENDENT CHANGE (Twice NAT / Policy NAT)
**Goal:** Perform NAT, but *only* under very specific, strict conditions.

*   **A) Manual NAT (Section 1 in FMC)**
    *   **Direction:** Any.
    *   **How it works:** In a single rule, the firewall checks **BOTH** the Source and the Destination before making a decision.
    *   **Example:** *"Change the source to a public IP, BUT ONLY IF the packet is going to the specific subnet of our business partner."*

---

### 🛑 4️⃣ NO CHANGE (Identity NAT / NAT Exempt)
**Goal:** Protect specific traffic from being caught by another, broader NAT rule (e.g., protecting VPN traffic from being PAT'ed to the internet).

*   **A) Static Identity NAT**
    *   **Direction:** Inside -> VPN_Zone.
    *   **How to configure:** ALWAYS as **Manual NAT** (Placed at the very top of Section 1 - "Above Rule 1").
    *   **Example:** *"If you are going to LAN-B through the IPsec VPN tunnel, translate your address to... your own address (do not change anything)."*