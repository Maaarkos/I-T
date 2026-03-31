# 🗺️ The Ultimate Cisco NAT Guide (FMC & ASA)

In the Cisco world, NAT is often confusing because documentation mixes **WHAT** we are changing (Source vs. Destination) with **HOW** we configure it (Auto NAT vs. Manual NAT). 

Let's organize this chaos into a clean, visual hierarchy.

---

### 🌳 The NAT Decision Tree

**🌐 CISCO NAT**

*   **1️⃣ CHANGING THE SOURCE** *(Goal: Hide the sender)*
    *   **Static NAT (1-to-1)**
        *   ⚙️ *Config:* Auto NAT (Object)
        *   🎯 *Use Case:* Dedicated Mail Server IP
    *   **Dynamic PAT (Many-to-1)**
        *   ⚙️ *Config:* Auto NAT (Object)
        *   🎯 *Use Case:* Entire Office to Internet
    *   **Dynamic NAT (Pool-to-Pool)**
        *   ⚙️ *Config:* Legacy
        *   🎯 *Use Case:* Dead technology

*   **2️⃣ CHANGING THE DESTINATION** *(Goal: Change the recipient on the fly)*
    *   **Static PAT (Port Forwarding)**
        *   ⚙️ *Config:* Auto NAT (Object)
        *   🎯 *Use Case:* Inbound Web Server in DMZ
    *   **Static Dest NAT (IP Hijacking)**
        *   ⚙️ *Config:* Manual NAT (Sec 1)
        *   🎯 *Use Case:* Redirect Dead IP (1.1.1.1 -> 9.9.9.9)

*   **3️⃣ TARGET-DEPENDENT CHANGE** *(Goal: Strict conditions)*
    *   **Twice NAT / Policy NAT**
        *   ⚙️ *Config:* Manual NAT (Sec 1)
        *   🎯 *Use Case:* NAT only for a specific VPN partner

*   **4️⃣ NO CHANGE** *(Goal: Bypass NAT)*
    *   **Identity NAT (NAT Exempt)**
        *   ⚙️ *Config:* Manual NAT (Top of Sec 1)
        *   🎯 *Use Case:* Protect IPsec VPN traffic

---

### 📖 Detailed Explanations & Real-World Scenarios

#### 1️⃣ Source NAT (Legacy: "Inside NAT")
We use this when traffic goes from the Inside to the Outside. We want to hide the true sender from the internet.
*   **Static NAT:** Our internal Mail Server (`192.168.10.100`) is permanently mapped to its own, dedicated public IP address (`80.80.80.89`).
*   **Dynamic PAT:** The entire office (`/24` subnet) goes out to the internet hiding behind the single public IP of the firewall. The firewall distinguishes sessions using Port numbers.

#### 2️⃣ Destination NAT / UN-NAT (Legacy: "Outside NAT")
This happens at the very ingress of the firewall. We change the recipient the packet was originally flying towards.
*   **Port Forwarding:** A client from the internet types our public IP into their browser. The firewall catches it and changes the destination on-the-fly to the private IP of the Web Server in the DMZ.
*   **IP Hijacking (Redirection):** A legacy machine stubbornly sends logs to a dead, hardcoded IP (`1.1.1.1`). The firewall intercepts it and silently replaces the destination with the new server (`9.9.9.9`).

#### 3️⃣ Twice NAT / Policy NAT
We perform NAT, but *only* under very specific conditions. In a single rule, the firewall checks **BOTH** the Source and the Destination before making a decision.
*   **Example:** *"Change the source to a public IP, BUT ONLY IF the packet is going to the specific subnet of our business partner."*

#### 4️⃣ Identity NAT / NAT Exempt
We want to protect specific traffic from being caught by another, broader NAT rule (e.g., protecting VPN traffic from being PAT'ed to the internet).
*   **Example:** *"If you are going to LAN-B through the IPsec VPN tunnel, translate your address to... your own address (do not change anything)."* This must always be placed at the very top of your rules (Above Rule 1).