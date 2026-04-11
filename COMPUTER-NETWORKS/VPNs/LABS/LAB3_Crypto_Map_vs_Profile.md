# 🛠️ LAB: GRE over IPsec (Crypto Map vs. IPsec Profile)

**Download the official lab instructions here:**
[📥 Download Lab: 16.1.4 Lab - Implement GRE over IPsec Site-to-Site VPNs](../16.1.4_Lab_GRE_over_IPsec.docx)
*(Note: Ensure the filename on your disk has no spaces, e.g., `16.1.4_Lab_GRE_over_IPsec.docx`, for the link to work properly).*

---

### 🧪 My Approach to this Lab

While we followed the official instructions for this lab, in a real-world scenario or for better learning, I would do it differently:
1.  **First, build the pure GRE tunnel** and run OSPF over it. This allows us to use Wireshark and actually *see* the unencrypted OSPF packets flying between the routers.
2.  **Then, apply the IPsec armor** over the GRE tunnel. This way, we can clearly see the "Before & After" effect of encryption.

In this lab, we configure **GRE over IPsec**. To make it interesting, we will use two different methods:
*   **Router R1:** The legacy method using **Crypto Maps** and ACLs.
*   **Router R3:** The modern method using **IPsec Profiles**.

> **💡 Clarification: Profiles vs. VTI**
> It's worth mentioning that an IPsec Profile is NOT exactly the same thing as a VTI (Virtual Tunnel Interface), although they are closely related.
> *   **Crypto Map:** The "glue" applied to a *physical* interface. The old-school way.
> *   **IPsec Profile (GRE over IPsec):** The "glue" applied to a *tunnel* interface. IOS creates a GRE pipe by default, and the profile simply glues encryption to it.
> *   **VTI:** Uses the EXACT SAME profile (glue) as above! The difference is that you manually remove the GRE encapsulation by typing `tunnel mode ipsec ipv4` on the interface. You just apply the profile to this new, "slimmed-down" pipe.

---

### 🪆 The "Matryoshka" Packet Structure (GRE over IPsec)

To truly understand what we are configuring, we need to look at the packet structure. In this lab, we are using **IPsec in Transport Mode**. 

Why Transport Mode? Because GRE already adds a new IP header (the "Delivery IP"). If we used IPsec Tunnel Mode, we would add *another* unnecessary IP header, creating a double matryoshka and wasting MTU space.

Here is what the final packet looks like on the physical wire:

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto;">
+-------------+-----------+------------+-------------+-------------+-------------+-----------+
| Delivery IP |  ESP Hdr  | GRE Header | Original IP |   TCP/UDP   |   Payload   | ESP Trail |
| (Public)    |           | (Protocol) | (Private)   |   Header    |   (Data)    | & Auth    |
+-------------+-----------+------------+-------------+-------------+-------------+-----------+
                          | <-------------------- ENCRYPTED -------------------> |
              | <------------------------- AUTHENTICATED ----------------------------------> |
</pre>

**How the Matryoshka is built (Step-by-Step):**
1.  **The Core:** The user generates a standard packet with private IPs (`Original IP` + `TCP/UDP` + `Payload`).
2.  **The GRE Wrapper:** The router takes this packet and wraps it in a `GRE Header`. It also adds a new `Delivery IP` header (the public IPs of the routers) so it can cross the internet. *(At this point, if you sniffed the traffic, you could read everything in plain text!)*
3.  **The IPsec Armor:** The Crypto Map (or IPsec Profile) kicks in. It takes the GRE packet, encrypts everything *after* the Delivery IP, and wraps it in `ESP`. Now the packet is secure.

---

### ⚖️ Configuration Comparison (Side-by-Side)

Here is the configuration comparison. As you can see, using IPsec Profiles saves us about 5-6 lines of code and eliminates the need for complex ACLs.

<table style="width:100%; border-collapse: collapse; font-family: monospace; font-size: 13px;">
  <tr style="background-color: #333; color: #00ff00; text-align: left;">
    <th style="padding: 10px; border: 1px solid #555; width: 50%;">Router R1 (Legacy Crypto Map)</th>
    <th style="padding: 10px; border: 1px solid #555; width: 50%;">Router R3 (Modern IPsec Profile)</th>
  </tr>
  <tr>
    <td style="padding: 10px; border: 1px solid #555; vertical-align: top; background-color: #000; color: #e6e6e6;">
      <span style="color: #888;">! 1. Phase 1 (IKEv1)</span><br>
      R1(config)# crypto isakmp policy 10<br>
      R1(config-isakmp)# encryption aes 256<br>
      R1(config-isakmp)# hash sha256<br>
      R1(config-isakmp)# authentication pre-share<br>
      R1(config-isakmp)# group 14<br>
      R1(config-isakmp)# lifetime 3600<br>
      R1(config-isakmp)# exit<br><br>
      R1(config)# crypto isakmp key cisco123 address 64.100.1.2<br><br>
      <span style="color: #888;">! 2. Phase 2 (Transform Set)</span><br>
      R1(config)# crypto ipsec transform-set GRE-VPN esp-aes 256 esp-sha256-hmac<br>
      R1(cfg-crypto-trans)# mode transport<br><br>
      <span style="color: #ff5555;">! 3. The ACL (Identifying Traffic)</span><br>
      <span style="color: #ff5555;">R1(config)# ip access-list extended GRE-VPN-ACL</span><br>
      <span style="color: #ff5555;">R1(config-ext-nacl)# permit gre host 64.100.0.2 host 64.100.1.2</span><br><br>
      <span style="color: #ff5555;">! 4. The Crypto Map (The Glue)</span><br>
      <span style="color: #ff5555;">R1(config)# crypto map GRE-CMAP 10 ipsec-isakmp</span><br>
      <span style="color: #ff5555;">R1(config-crypto-map)# match address GRE-VPN-ACL</span><br>
      <span style="color: #ff5555;">R1(config-crypto-map)# set transform-set GRE-VPN</span><br>
      <span style="color: #ff5555;">R1(config-crypto-map)# set peer 64.100.1.2</span><br><br>
      <span style="color: #ff5555;">! 5. Applying to PHYSICAL Interface</span><br>
      <span style="color: #ff5555;">R1(config)# interface g0/0/0</span><br>
      <span style="color: #ff5555;">R1(config-if)# crypto map GRE-CMAP</span><br><br>
      <span style="color: #888;">! 6. The GRE Tunnel</span><br>
      R1(config)# interface Tunnel1<br>
      R1(config-if)# bandwidth 4000<br>
      R1(config-if)# ip address 172.16.1.1 255.255.255.252<br>
      R1(config-if)# ip mtu 1400<br>
      R1(config-if)# tunnel source 64.100.0.2<br>
      R1(config-if)# tunnel destination 64.100.1.2<br>
    </td>
    <td style="padding: 10px; border: 1px solid #555; vertical-align: top; background-color: #000; color: #e6e6e6;">
      <span style="color: #888;">! 1. Phase 1 (IKEv1)</span><br>
      R3(config)# crypto isakmp policy 10<br>
      R3(config-isakmp)# encryption aes 256<br>
      R3(config-isakmp)# hash sha256<br>
      R3(config-isakmp)# authentication pre-share<br>
      R3(config-isakmp)# group 14<br>
      R3(config-isakmp)# lifetime 3600<br>
      R3(config-isakmp)# exit<br><br>
      R3(config)# crypto isakmp key cisco123 address 64.100.0.2<br><br>
      <span style="color: #888;">! 2. Phase 2 (Transform Set)</span><br>
      R3(config)# crypto ipsec transform-set GRE-VPN esp-aes 256 esp-sha256-hmac<br>
      R3(cfg-crypto-trans)# mode transport<br><br>
      <span style="color: #00ff00;">! 3. The IPsec Profile (The Glue)</span><br>
      <span style="color: #00ff00;">R3(config)# crypto ipsec profile GRE-PROFILE</span><br>
      <span style="color: #00ff00;">R3(ipsec-profile)# set transform-set GRE-VPN</span><br><br><br><br><br><br><br><br>
      <span style="color: #888;">! 4. The GRE Tunnel & Applying Profile</span><br>
      R3(config)# interface Tunnel1<br>
      R3(config-if)# bandwidth 4000<br>
      R3(config-if)# ip address 172.16.1.2 255.255.255.252<br>
      R3(config-if)# ip mtu 1400<br>
      R3(config-if)# tunnel source 64.100.1.2<br>
      R3(config-if)# tunnel destination 64.100.0.2<br>
      <span style="color: #00ff00;">R3(config-if)# tunnel protection ipsec profile GRE-PROFILE</span>
    </td>
  </tr>
</table>

---

### 🏢 Real-World Use Cases: Why build this "Matryoshka"?

You might ask: *"Why go through the trouble of building this matryoshka doll (OSPF stuffed inside GRE, and GRE stuffed inside IPsec)? Why not just type static routes by hand?"*

In the real world, this topology (or its newer VTI sibling) is used in two massive scenarios:

#### 1. The "Poor Man's MPLS" (Cheap Branch Links)
A company has its HQ in New York and a branch in Chicago. Instead of paying a telecom $5,000 a month for a private, dedicated MPLS circuit, they buy a standard, cheap internet connection for $100. They establish a GRE over IPsec tunnel, run OSPF through it, and instantly have a fully encrypted, dynamically routed network.
> **💸 Emotion & Statistics:** Over 60% of medium-sized businesses use this model. The accountant cries tears of joy looking at the bills, and you get a bonus for cost optimization.

#### 2. The Foundation for DMVPN (Corporate & Retail Networks)
This lab is the absolute foundation for understanding **DMVPN (Dynamic Multipoint VPN)**. 
Imagine you are the network admin for 500 gas stations or retail stores. Nobody in their right mind is going to manually type thousands of static routes to every single store! You create GRE tunnels (in Multipoint mode), wrap them in IPsec, fire up OSPF or BGP, and the network literally "learns itself" where all the cash registers are located.

**In short:** You use this everywhere you have cheap, untrusted internet (ISP), you need security (IPsec), and you don't want to manually manage routing (GRE + OSPF).