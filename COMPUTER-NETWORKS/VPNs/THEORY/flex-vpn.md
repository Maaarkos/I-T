# 🚀 FlexVPN Architecture & EAP Authentication

**FlexVPN** is not a protocol itself; it is a Cisco-designed, universal architecture (a framework) that relies entirely on the pure **IKEv2** engine under the hood. 

What makes this solution so great? Whether you are building Site-to-Site, Remote Access, or Hub-to-Spoke topologies, you use a unified configuration based on **VTI (Virtual Tunnel Interfaces)**, which natively support routing without the need for GRE. It is like a specially prepared cockpit for network engineers.

### 🌟 The Killer Features of FlexVPN

Cisco took the IKEv2 standard and built a massive ecosystem around it. Here is what FlexVPN brings to the table:

*   **Multi-Vendor Interoperability:** Because it is based on the open IKEv2 standard, you can easily build tunnels with non-Cisco devices.
*   **Dynamic Routing Support:** We can run OSPF or EIGRP inside the overlay tunnels. However, the real magic is **IKEv2 Route Exchange**, where the IKEv2 protocol itself injects routes into the routing table!
*   **Full AAA Integration (RADIUS):** Deep integration with RADIUS servers (like Cisco ISE). We can even inject routes and ACLs directly from the RADIUS server per user.
*   **Configuration Exchange:** During tunnel setup, the entire configuration (IP address, DNS servers, split-tunneling rules) is sent to the client. In older VPNs, this was a bolted-on patch. Here, it is fully integrated into the IKEv2 payload.
*   **IKEv2 Redirect for Load Balancing:** If Hub A is choked to the brim, it can dynamically redirect the incoming VPN request to Hub B.
*   **IKEv2 Fragmentation:** Authenticating with large RSA certificates generates massive IKEv2 packets. This feature allows the router to chop these packets into smaller pieces *before* they leave the router, avoiding MTU drops on the internet.
*   **Call Admission Control (CAC):** In short: if the router's CPU is overloaded, it stops accepting new VPN connections. It is better to reject new users than to let the router crash and kill the service for everyone.
*   **Session Deletion on Certificate Revocation:** If an employee is fired or their certificate expires, FlexVPN instantly tears down their active connection.

---

### ✉️ IKEv2 + EAP (Extensible Authentication Protocol)

When we have a large user base for Remote Access VPNs, we don't create local user accounts on the firewall. Instead, we use the **EAP** protocol to connect to a RADIUS server (e.g., Cisco ISE).

<div align="center">
  <a href="IMAGES/IKEv2_EAP.png" target="_blank">
    <img src="IMAGES/IKEv2_EAP.png" style="max-width: none; width: 1000px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

**What is EAP?**
EAP is not an authentication mechanism itself; it is a universal "envelope". It is usually written with its transport type, for example, **EAP-MSCHAPv2**. 
*   **EAP:** The universal envelope.
*   **MSCHAPv2:** The actual letter inside the envelope. This protocol takes your login and password (e.g., from Active Directory), encrypts them, and verifies them.

**The Relationship between EAP and RADIUS:**
1. The laptop creates the EAP envelope and sends it to the network device (e.g., a firewall or a switch).
2. The network device **does not open or verify** the EAP envelope! It simply packs the EAP frame into a RADIUS packet and forwards it to the AAA server (Cisco ISE).
*(Note: 802.1X is simply the mechanism of blocking/unblocking a switch port based on what the EAP envelope tells it).*

#### The EAP Authentication Steps
<div align="center">
  <a href="IMAGES/EAP_Steps.png" target="_blank">
    <img src="IMAGES/EAP_Steps.png" style="max-width: none; width: 1000px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

Notice that the Server must *first* prove its identity to the Client using a Certificate so the Client knows it can trust the connection. 
After that, the FlexVPN server acts purely as a middleman (a proxy). 
*   Messages between the Client and the FlexVPN Server are transported as **IKEv2 EAP Payloads** inside `IKE_AUTH` requests and responses.
*   Messages between the FlexVPN Server and the RADIUS Server are transported inside the **RADIUS EAP-Message attribute**.
*   The RADIUS server sends an *EAP Request* to ask for info, and the supplicant (client) sends an *EAP Response*.

---

### 🚦 The FlexVPN Connection Flow (Step-by-Step)

How does the router process an incoming connection?

<div align="center">
  <a href="IMAGES/FlexVPN_Flow.png" target="_blank">
    <img src="IMAGES/FlexVPN_Flow.png" style="max-width: none; width: 1000px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

**1. Peer Authentication (Checking the ID Card)**
The router verifies who you are (via Certificate or PSK).

**2. User & Group Authorization and Attribute Merging (Mixing the Rules - EXAM TIP!)**
*   *What it is:* The router knows who you are. Now it checks what you are allowed to do.
*   *The Keyword:* **"Merging"**. The router downloads rules (e.g., ACLs, routes) from its local configuration AND from the AAA server (RADIUS). It then mixes them into one cohesive policy for this specific user.

**3. Configuration Request Processing (Asking for an IP)**
*   *What it is:* The client says: *"Okay, you let me in, now give me an IP address and DNS servers."*

**4. Per-Session Interface Creation (Building the Virtual Room - CRUCIAL!)**
*   *What it is:* FlexVPN is amazing. For EVERY user that connects, the router "welds" a brand new, virtual interface in a fraction of a second (called a `Virtual-Access` interface).
*   *Notice:* This interface is created ONLY AFTER authorization (Step 2). The router does not waste CPU/RAM building an interface for someone who hasn't passed verification!

**5. Generation of Configuration Reply (Handing over the Room Keys)**
*   *What it is:* The router sends back a packet (`IKE_AUTH Reply`) containing the IP address for the client, DNS servers, and confirmation that the tunnel is up.

---

> **💡 Legacy vs. Modern RADIUS Ports**
> When configuring the RADIUS server on your firewall, you might see ports `1645/1646`. These are the old (legacy) RADIUS ports. The modern standard is `1812/1813`. Cisco allows you to use both. 
> The password (`key`) configured here is the **Shared Secret** between the router and the ISE server, ensuring no stranger can query your RADIUS database.