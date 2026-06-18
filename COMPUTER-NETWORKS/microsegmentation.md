# 🧩 Microsegmentation: Cisco ACI vs. Cisco ISE (TrustSec)

The modern approach to network security is moving away from traditional, IP-based Access Control Lists (ACLs). IP addresses change, users roam, and maintaining massive ACLs is a nightmare. 

**Microsegmentation** solves this by decoupling security from the network topology. We block or allow traffic based on *logical tags* or *groups*, regardless of the IP address or subnet. 

Here is how Cisco achieves this in the Data Center (ACI) and the Campus (ISE).

---

### 🏢 A. Data Center Microsegmentation (Cisco ACI)

In the Data Center, Cisco ACI (Application Centric Infrastructure) handles microsegmentation.

*   **EPGs (Endpoint Groups):** Inside a Tenant, we group resources into EPGs (e.g., a "Web Servers" EPG and a "Database Servers" EPG). Policies (Contracts) dictate if these groups can talk to each other.
*   **uSeg (Micro-EPGs):** What if we want to isolate servers that are in the *exact same* EPG and the *exact same* subnet? We use uSeg (Microsegmentation) groups. 
    *   *Example:* We have Production and Development servers in the same `10.1.1.0/24` subnet. We dynamically assign them to different uSeg groups based on their VM name or OS. Thanks to this, they cannot see each other, even though they share the same IP space!
*   **VMM Integration:** ACI can dynamically pull these VM attributes by integrating directly with hypervisor managers like **VMware vCenter** or **Microsoft SCVMM** (System Center Virtual Machine Manager).

---

### 🏢 B. Campus/LAN Microsegmentation (Cisco ISE & TrustSec)

In the enterprise campus (users, laptops, printers), microsegmentation is driven by **Cisco ISE** using the **TrustSec** architecture.

*   **SGT (Security Group Tags):** Instead of IPs, ISE assigns a unique tag to a user/device upon connection. 
    *   *Example:* The Accounting department gets SGT `10`, and IT Admins get SGT `20`.
*   **Assignment Methods:** ISE assigns these tags dynamically after successful authentication via **802.1X**, **MAB** (MAC Authentication Bypass for dumb devices like printers), or a **Web Auth Portal**.

#### The Tagging Problem: Inline Tagging vs. SXP

Ideally, the access switch takes the SGT tag from ISE and physically injects it into the Ethernet frame header (this is called **Inline Tagging** using the CMD - Cisco Meta Data field). 

*The Problem:* What if you have legacy switches, third-party vendor switches, or a router along the path that doesn't understand TrustSec and drops these modified frames?

*The Solution:* **SXP (SGT Exchange Protocol)**.
If the hardware cannot carry the tag *inside* the packet, we send the tag "out-of-band" alongside the traffic.

*   **How SXP works:** It establishes a TCP connection on **Port 64999**.
*   **SXP Speaker:** (Usually ISE or a TrustSec-capable edge switch). It tells the listener: *"Hey, IP address 192.168.5.50 belongs to SGT 10."*
*   **SXP Listener:** (e.g., a core switch or a Firepower firewall). It receives this message and stores the `IP-to-SGT` mapping in its RAM. When the actual untagged data packet arrives, the Listener looks at the source IP, checks its RAM, and says: *"Ah, I know this IP belongs to SGT 10. I will apply the security policy accordingly!"*