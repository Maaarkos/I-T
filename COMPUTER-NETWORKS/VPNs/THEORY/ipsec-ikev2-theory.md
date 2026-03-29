# 🚀 IPsec & IKEv2: The Modern Standard (Parent & Child SAs)

Engineers looked at IKEv1 and decided to combine the **speed** of Aggressive Mode with the **security** of Main Mode (specifically, the "cookie" concept to prevent hackers from executing DoS attacks by forcing the router to calculate heavy DH math).

Thus, **IKEv2** was born. It is vastly superior, leaner, and establishes the initial tunnel using exactly **4 messages**.

---

### 👨‍👦 The Paradigm Shift: Goodbye Phases, Hello Parents & Children

In IKEv2, we completely abandon the concepts of "Phase 1" and "Phase 2". Instead, the architecture introduces the concept of a **Parent (`IKE_SA`)** and a **Child (`CHILD_SA`)**.

*   **`IKE_SA` (The Parent):** This is the main management tunnel (equivalent to the old Phase 1).
*   **`CHILD_SA` (The Child):** This is the tunnel for actual user data (equivalent to the old Phase 2). Just like in IKEv1, these are unidirectional (one for TX, one for RX) with their own SPI numbers.

**Why "Child"?** Because it logically derives from the Parent. If you reset or kill the Parent (`IKE_SA`), all of its Children (`CHILD_SA`) automatically die with it.

---

### ⚡ The Golden Rule of 4 Packets

Setting up the first tunnel takes only 4 packets. Here is how to easily remember it:

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
[ ROUTER A ]                                            [ ROUTER B ]
      |                                                      |
      | --- 1. IKE_SA_INIT (Ciphers, DH) ------------------> |
      | <--- 2. IKE_SA_INIT (Ciphers, DH) ------------------ |
      |                                                      |
      | --- 3. IKE_AUTH (ID, Cert/PSK + FIRST CHILD_SA) ---> |
      | <--- 4. IKE_AUTH (ID, Cert/PSK + FIRST CHILD_SA) --- |
</pre>

**Pair 1: `IKE_SA_INIT` (Packets 1 & 2)**
*   **What it does:** Starts building the Parent (`IKE_SA`). You exchange ciphers and Diffie-Hellman math.
*   **Status:** The Parent is half-ready, but you don't trust each other yet (no authentication).

**Pair 2: `IKE_AUTH` (Packets 3 & 4)**
*   **What it does:** IT DOES TWO THINGS AT ONCE!
    1.  *First:* You show your ID cards (Certificates/PSK), which finishes building the Parent (`IKE_SA`).
    2.  *Second:* Inside these exact same packets, you "smuggle" the parameters for the data tunnel (ESP/ACLs), which instantly creates the **First Child (`CHILD_SA`)**.

#### What if we have more subnets? (`CREATE_CHILD_SA`)
*   **Subnet 1 (Accounting):** Uses `IKE_SA_INIT` (2 pkts) + `IKE_AUTH` (2 pkts). TOTAL: 4 PACKETS! IKEv2 saves time by smuggling the first network inside the auth packets.
*   **Subnet 2 (HR):** Since `IKE_AUTH` was already "used up" to build the Parent and the first Child, how do we build a second Child? Engineers created a new message type: **`CREATE_CHILD_SA`** (2 packets). It is used *exclusively* for adding subsequent networks or for renewing keys (Rekeying).

---

### 🏆 Game Changers: Why IKEv2 is a Masterpiece

#### 1. The End of the Lifetime Lottery!
Remember how in IKEv1, mismatched lifetimes (e.g., you have 24h, the other side has 8h) were a lottery? Sometimes they agreed on the shorter one, sometimes an old firewall rejected it, and the network went down with a "No Proposal Chosen" error.

IKEv2 fixed this architectural flaw: **In IKEv2, the Lifetime value is NOT NEGOTIATED!**
It isn't even sent in the `IKE_SA_INIT` or `IKE_AUTH` packets!

*   **How it works:** Each router keeps its watch to itself. 
*   **Example:** Your Firepower is set to 24h. The partner's firewall is set to 8h. The tunnel comes up instantly without errors. After 8 hours, the partner's firewall (having the shorter timer) simply initiates the Rekey process first. Your Firepower accepts it, they renew the keys, and everyone is happy.

#### 2. Native EAP Support (Active Directory Integration)
**EAP** is not a single protocol; it is an "envelope for authentication methods".
Because IKEv2 natively supports EAP, your firewall can take the username and password you typed into the VPN prompt, put it in an EAP envelope, and send it straight to a RADIUS server (like Cisco ISE). The RADIUS server then asks Active Directory: *"Hey, is John Doe's password correct?"*.

> **💻 Windows Native Client Trivia:**
> You can connect to a Firepower VPN using the built-in Windows IKEv2 client! When you do this, Windows ALWAYS forces **Tunnel Mode**. If you configure IKEv2 Remote Access with EAP (e.g., EAP-MSCHAPv2) on the FPR, Windows will simply ask for your AD credentials, send them securely to the FPR, and the tunnel comes right up!

---

### 🏗️ Encapsulation Modes Reminder

Just like IKEv1, IKEv2 uses **Tunnel Mode** (default for 99% of setups) and **Transport Mode**. 

When do we use Transport Mode in IKEv2? In the exact same, rare cases as IKEv1:
1.  **DMVPN (Dynamic Multipoint VPN):** Cisco's tech for connecting hundreds of branches. It uses GRE, which already creates an IP "matryoshka". To avoid a double matryoshka, IKEv2 (which encrypts the GRE) is set to Transport Mode.
2.  **Host-to-Host:** When two servers with public IPs talk directly to each other and just want to encrypt the traffic without hiding their addresses.

---

### 🔐 Bonus Security Note: RADIUS vs. TACACS+

While discussing EAP and Active Directory, it's crucial to remember the difference between the two main AAA protocols:

| Feature | RADIUS | TACACS+ (Cisco Proprietary) |
| :--- | :--- | :--- |
| **Primary Use** | **Network Access** (Lets users into the LAN/VPN/Internet). | **Device Access** (Lets Admins into the router's console). |
| **Architecture** | Combines Authentication and Authorization into one process. | Separates Authentication and Authorization. |
| **Granularity** | You are either "in" or "out". | Allows strict control over *individual CLI commands* (e.g., Admin A can `show run`, but cannot `configure terminal`). |