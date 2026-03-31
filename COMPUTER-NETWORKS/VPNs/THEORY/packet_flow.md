# 🚦 Cisco FTD Packet Flow & Order of Operations

Burn this mental map into your memory before taking the CCNP SCOR exam. Understanding the exact order in which a firewall processes a packet is the key to solving 99% of routing and NAT issues.

### 🗺️ The Golden Path (Mental Map)

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto;">
[1. INGRESS] -> [2. UN-NAT] -> [3. ROUTING] -> [4. ACP] -> [5. NAT] -> [6. VPN] -> [7. EGRESS]
</pre>

1.  **Ingress (The Entrance):** Which cable/zone did you come from?
2.  **UN-NAT (Destination NAT):** Is your destination a real address, or just a public mask? *(We take off the destination mask).*
3.  **Routing (The Map):** Where does the *real* destination live, and which door (Zone) must you exit through?
4.  **ACP (The Bouncer):** Is the *Real Sender* allowed to talk to the *Real Destination* through this specific door?
5.  **NAT (Source NAT):** Before you leave, do I need to put a mask on your face (PAT)?
6.  **VPN (Encryption):** Is this exit door an IPsec pipe? If so, I pack you into a safe (ESP).
7.  **Egress (The Exit):** Have a safe trip!

---

### 🏢 Scenario 1: Inbound Web Traffic (Static NAT)

**The Situation:** A client on the internet types `http://80.80.80.80` into their browser. The packet flies through the internet and hits your firewall. What happens next?

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto;">
[ CLIENT ]                                                             [ WEB SERVER ]
203.0.113.5  ====================> [ FIREWALL ] ====================>  192.168.50.100
                                         |
 1. INGRESS: Zone = Outside              |
 2. UN-NAT:  Dst 80.80.80.80 changed to 192.168.50.100 (Static NAT)
 3. ROUTING: Where is 192.168.50.100? -> Exit Zone = DMZ
 4. ACP:     Allow 203.0.113.5 to access 192.168.50.100 on Port 80? -> ALLOW!
 5. EGRESS:  Send out via DMZ interface
</pre>

**Step-by-Step Breakdown:**
1.  **Ingress:** The packet enters through the ISP cable. The firewall assigns it to the `Outside` zone.
2.  **UN-NAT (The Key Step!):** The firewall looks at the destination (`80.80.80.80`). It checks the NAT table and says: *"Aha! I have a Static NAT rule. This public IP is just a cover. The real target is my internal server `192.168.50.100`."* **Action:** The firewall physically overwrites the destination IP in the packet header.
3.  **Routing:** The firewall asks: *"Where does this REAL target live?"*. The routing table replies: *"You must exit through the `DMZ` interface."*
4.  **ACP (The Bouncer):** Now the security policy steps in. The bouncer asks: *"Can the Client (`203.0.113.5`) from the `Outside` zone enter the `DMZ` zone and talk to the REAL Server (`192.168.50.100`)?"* 
    *(Crucial Exam Note: The bouncer checks the REAL address, NOT the public one!)*. **Result: ALLOW.**
5.  **NAT (Source Check):** Does the sender need a mask? No, the server needs to know who to reply to. Leave it original.
6.  **Egress:** The packet safely leaves the firewall and hits the Web Server.

---

### 🧟‍♂️ Scenario 2: The "Dead IP" Redirection (Destination NAT)

**The Situation:** An old machine on your LAN stubbornly sends logs to a dead IP address (`1.1.1.1`). Why? Because the IP is hardcoded into its legacy software, and the company that hosted it went bankrupt. We must force the firewall to intercept these packets and redirect them to a new server (`9.9.9.9`) on the fly, without the legacy machine ever noticing.

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto;">
[ LEGACY MACHINE ]                                                     [ NEW SERVER ]
  192.168.10.50  ================> [ FIREWALL ] ====================>     9.9.9.9
  (Thinks it's                           |                             (Receives from
   talking to 1.1.1.1)                   |                              80.80.80.80)
                                         |
 1. INGRESS: Zone = Inside               |
 2. UN-NAT:  Dst 1.1.1.1 intercepted & changed to 9.9.9.9
 3. ROUTING: Where is 9.9.9.9? -> Exit Zone = Outside
 4. ACP:     Allow 192.168.10.50 to access 9.9.9.9? -> ALLOW!
 5. NAT:     Src 192.168.10.50 changed to 80.80.80.80 (PAT)
 6. EGRESS:  Send out via Outside interface
</pre>

**Step-by-Step Breakdown:**
1.  **Ingress:** The packet from the machine enters the LAN cable. Zone: `Inside`.
2.  **UN-NAT (Changing the Target):** The firewall looks at the destination (`1.1.1.1`). It checks the NAT table and says: *"Aha! I have a special rule. If anyone from the Inside tries to go to 1.1.1.1, I must quietly change their destination to `9.9.9.9`."* **Action:** The firewall physically erases `1.1.1.1` and writes `9.9.9.9`.
3.  **Routing:** The firewall asks: *"Where does this NEW target (`9.9.9.9`) live?"*. The routing table replies: *"Exit through the `Outside` zone."* *(Notice: Routing is already looking for the NEW address!)*
4.  **ACP (The Bouncer):** The bouncer asks: *"Can the Machine (`192.168.10.50`) from the `Inside` zone talk to the New Server (`9.9.9.9`) in the `Outside` zone?"* 
    *(The bouncer has absolutely no idea the machine originally wanted to go to 1.1.1.1!)*. **Result: ALLOW.**
5.  **NAT (Putting a Mask on the Source):** The firewall looks at the sender (`192.168.10.50`). It says: *"You are going out to the public internet, so I must put a mask on you (PAT)."* It changes the source to the firewall's public IP (e.g., `80.80.80.80`).
6.  **Egress:** The packet flies into the internet. It now has a Source of `80.80.80.80` and a Destination of `9.9.9.9`. The legacy machine is none the wiser!