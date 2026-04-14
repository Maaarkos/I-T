# 🕸️ DMVPN Phase 1: The Godfather of SD-WAN

**DMVPN (Dynamic Multipoint VPN)** is a Cisco-proprietary solution. While it doesn't always play nicely with equipment from other vendors, it is an absolute cornerstone of modern networking. In this article, we will focus specifically on **Phase 1** architecture.

### 🤔 Is DMVPN still used in practice today?

**Absolutely.** Even though the networking world has gone crazy over SD-WAN, thousands of companies worldwide still rely on classic DMVPN. Why? Because it’s free (included in the router license), doesn't require expensive controllers, and it simply works. It is estimated that a massive chunk of legacy corporate networks (up to 40-50%) still run on this technology or are slowly migrating away from it.

**Why should you learn it?**
1.  **The Foundation:** DMVPN is the "Godfather" of today's SD-WAN. Technologies like mGRE, NHRP, and IPsec are the absolute baseline. If you understand DMVPN, you will understand how modern WANs work under the hood.
2.  **Troubleshooting:** When the magical, modern SD-WAN suddenly crashes, you still have to dig into the CLI to analyze tunnels, encryption, and routing.
3.  **Ignorance Hurts:** Learning SD-WAN without knowing DMVPN is like trying to fly a fighter jet after taking a scooter riding course. You can click pretty, colorful graphs in the GUI, but when something breaks, all you can do is cry in a cold server room. 😉

In short: This is mandatory knowledge for every true Network and Security Engineer.

---

### 🧩 The "Big Four" of DMVPN

<div align="center">
  <a href="IMAGES/DMVPN.png" target="_blank">
    <img src="IMAGES/DMVPN.png" style="max-width: none; width: 1000px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

DMVPN is built upon mGRE and NHRP, but to make it a fully-fledged technology (especially in Security!), we must add two more elements. Together, they form the "Big Four":

1.  **mGRE (Multipoint GRE):** Builds a *single* tunnel interface to reach multiple destinations. No more manually typing hundreds of point-to-point tunnels. Pure relief for an engineer's fingers! 😌 *(In our lab, this will be the `Tunnel0` interface on the Hub).*
2.  **NHRP (Next Hop Resolution Protocol):** Acts like a phonebook (or an ARP/DNS hybrid) for routers. It answers the question: *"Behind which public IP address is this specific branch hiding?"*
3.  **IPsec:** Because mGRE and NHRP do not encrypt traffic by themselves. Without IPsec, we would be sending corporate data in plain text, which is a mortal sin in Security! 💀 *(Note: For the sake of learning the routing mechanics, we often skip IPsec configuration in initial Phase 1 labs).*
4.  **Routing Protocol (EIGRP, OSPF, BGP):** So the routers can dynamically learn which LAN subnets are hiding behind the tunnels.

---

### 🏛️ Phase 1 Architecture: The Hub & Spoke

In this infrastructure, we have a **HUB** (e.g., Router R1 at HQ) and **SPOKES** (e.g., Routers R2 and R3 at the branches).

The defining characteristic of **DMVPN Phase 1** is that **ALL traffic must go through the Hub**. If Spoke R2 wants to talk to Spoke R3, the packet must fly to the HQ first, and the HQ forwards it to R3.

> **💡 Crucial Concept: The NHRP Table**
> The most important element here is the NHRP table on the Hub. It maps the *Public (NBMA)* IP addresses of the Spokes to their *Private (Tunnel)* IP addresses. *(Do not confuse NHRP with FHRP like HSRP/VRRP!)*

#### What is NBMA?
NBMA stands for **Non-Broadcast Multi-Access**.
In the DMVPN world, this is simply the physical underlying network (the Underlay)—usually the Internet or an ISP's network.
*   **Multi-Access:** Because many routers are connected to it (like a giant switch).
*   **Non-Broadcast:** Because you cannot send a broadcast packet into the Internet shouting, *"Hey, who has IP 10.0.0.5?!"*. If that were possible, the Internet would explode from garbage traffic in 3 seconds. 💥

---

### 🛠️ Engineering Pro-Tips & Configurations

#### 1. Why use Loopback Interfaces?
A major best practice is tying the tunnel to a Loopback interface rather than a physical interface (like `GigabitEthernet0/1`). Why? 
A Loopback interface is ALWAYS `up`. If you tie the tunnel to a physical port and the cable gets unplugged for 3 seconds, the interface goes `down`. The routing protocol immediately drops neighbor adjacencies, flushes routes, and when you plug the cable back in, it has to recalculate everything from scratch. 
With a Loopback, the interface never flaps. The router simply uses its routing table to find an alternate physical path to push the packets out.

#### 2. The Magic of `ip nhrp map multicast dynamic`
This command often confuses engineers. 
As we know, you cannot send Multicast or Broadcast traffic over the public Internet (NBMA). However, routing protocols (like OSPF or EIGRP) rely heavily on Multicasts to find neighbors!

When you type this command on the Hub, you are telling the router: *"Look at the NHRP table. Take the Multicast packet generated by OSPF/EIGRP, make individual Unicast clones of it, and send a copy to every public IP listed in the NHRP table."*
*(Note: In a standard Point-to-Point GRE lab, we didn't need this because we explicitly typed `tunnel destination X.X.X.X`, so the router already knew exactly where to send the cloned unicast packets).*

#### 3. The "Split Horizon" Trap
The Split Horizon rule is a network anti-gossip mechanism. It tells a router: *"Never advertise a route back out the exact same interface you learned it from."* This prevents routing loops and is enabled by default.

However, in DMVPN Phase 1, **we MUST disable Split Horizon on the Hub (R1)**!
*   **The Scenario:** Spoke R2 sends its LAN route to the Hub (R1) via the `Tunnel0` interface. 
*   **The Problem:** If Split Horizon is on, R1 will refuse to advertise that route back out of `Tunnel0` to Spoke R3. 
*   **The Result:** Spoke R3 will never know how to reach R2, and since all traffic in Phase 1 *must* go through the Hub, communication between branches completely fails.

<pre style="background-color: #000000; color: #ffffff; padding: 15px; font-size: 15px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
R1(config)# interface Tunnel 0
R1(config-if)# no ip split-horizon eigrp 100
</pre>