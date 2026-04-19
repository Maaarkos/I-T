# 🛠️ LAB: FlexVPN Site-to-Site (IKEv2 + VTI)

Based on official Cisco documentation, we are going to build a **FlexVPN Site-to-Site** tunnel. 

<div align="center">
  <a href="IMAGES/FlexVPN_S2S.png" target="_blank">
    <img src="IMAGES/FlexVPN_S2S.png" style="max-width: none; width: 1000px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

### 🤔 What is FlexVPN?

FlexVPN is Cisco's universal VPN framework designed from the ground up to be based **exclusively on IKEv2**. 
What is so great about it? Whether you are building Site-to-Site, Remote Access, or Hub-and-Spoke topologies, you use the exact same, unified configuration blocks based on **VTI (Virtual Tunnel Interfaces)**, which natively support routing without the need for GRE.

*Wait, didn't we just do a "VTI over IPsec" lab? How is this different?*
Classic VTI historically relied on the old IKEv1 (ISAKMP). The main difference doesn't lie in the interface type itself (both use VTI in `mode ipsec ipv4`), but in the framework and the massive capabilities of IKEv2.

---

### ⚖️ FlexVPN Configuration (Side-by-Side)

Here is the complete configuration for both routers. Notice how modular and object-oriented the IKEv2 configuration is.

<table style="width:100%; border-collapse: collapse; font-family: monospace; font-size: 13px;">
  <tr style="background-color: #333; color: #00ff00; text-align: left;">
    <th style="padding: 10px; border: 1px solid #555; width: 50%;">Router A (Initiator)</th>
    <th style="padding: 10px; border: 1px solid #555; width: 50%;">Router B (Responder)</th>
  </tr>
  <tr>
    <td style="padding: 10px; border: 1px solid #555; vertical-align: top; background-color: #000; color: #e6e6e6;">
      <span style="color: #888;">! 1. IKEv2 Proposal (The Crypto Block)</span><br>
      crypto ikev2 proposal FLEXVPN_PROPOSAL<br>
       encryption aes-cbc-256<br>
       integrity sha256<br>
       group 14<br>
      !<br>
      <span style="color: #888;">! 2. IKEv2 Policy (Global Attachment)</span><br>
      crypto ikev2 policy FLEXVPN_POLICY<br>
       proposal FLEXVPN_PROPOSAL<br>
      !<br>
      <span style="color: #888;">! 3. Keyring (The Password Bag)</span><br>
      crypto ikev2 keyring FLEXVPN_KEYRING<br>
       peer FLEVPNPeers<br>
       address 192.168.0.20<br>
       pre-shared-key local cisco123<br>
       pre-shared-key remote cisco123<br>
      !<br>
      <span style="color: #888;">! 4. IKEv2 Profile (Identity & Time)</span><br>
      crypto ikev2 profile FLEXVPN_PROFILE<br>
       match identity remote address 192.168.0.20<br>
       authentication remote pre-share<br>
       authentication local pre-share<br>
       keyring local FLEXVPN_KEYRING<br>
       lifetime 86400<br>
       dpd 10 2 on-demand<br>
      !<br>
      <span style="color: #888;">! 5. IPsec Phase 2 (Transform Set)</span><br>
      crypto ipsec transform-set FLEXVPN_TRANSFORM esp-aes 256 esp-sha-hmac<br>
       mode tunnel<br>
      !<br>
      <span style="color: #888;">! 6. IPsec Profile (The Ultimate Glue)</span><br>
      crypto ipsec profile FLEXVPN_PROFILE<br>
       set transform-set FLEXVPN_TRANSFORM<br>
       <span style="color: #ff5555;">set ikev2-profile FLEXVPN_PROFILE</span><br>
      !<br>
      <span style="color: #888;">! 7. Interfaces & Routing</span><br>
      interface Tunnel0<br>
       ip address 10.1.120.10 255.255.255.0<br>
       tunnel source GigabitEthernet0/1<br>
       tunnel destination 192.168.0.20<br>
       tunnel protection ipsec profile FLEXVPN_PROFILE<br>
      !<br>
      interface GigabitEthernet0/1<br>
       ip address 192.168.0.10 255.255.255.0<br>
       no shutdown<br>
      !<br>
      router eigrp 100<br>
       no auto-summary<br>
       network 10.1.120.0 0.0.0.255
    </td>
    <td style="padding: 10px; border: 1px solid #555; vertical-align: top; background-color: #000; color: #e6e6e6;">
      <span style="color: #888;">! 1. IKEv2 Proposal (The Crypto Block)</span><br>
      crypto ikev2 proposal FLEXVPN_PROPOSAL<br>
       encryption aes-cbc-256<br>
       integrity sha256<br>
       group 14<br>
      !<br>
      <span style="color: #888;">! 2. IKEv2 Policy (Global Attachment)</span><br>
      crypto ikev2 policy FLEXVPN_POLICY<br>
       proposal FLEXVPN_PROPOSAL<br>
      !<br>
      <span style="color: #888;">! 3. Keyring (The Password Bag)</span><br>
      crypto ikev2 keyring FLEXVPN_KEYRING<br>
       peer FLEVPNPeers<br>
       address 192.168.0.10<br>
       pre-shared-key local cisco123<br>
       pre-shared-key remote cisco123<br>
      !<br>
      <span style="color: #888;">! 4. IKEv2 Profile (Identity & Time)</span><br>
      crypto ikev2 profile FLEXVPN_PROFILE<br>
       match identity remote address 192.168.0.10<br>
       authentication remote pre-share<br>
       authentication local pre-share<br>
       keyring local FLEXVPN_KEYRING<br>
       lifetime 86400<br>
       dpd 10 2 on-demand<br>
      !<br>
      <span style="color: #888;">! 5. IPsec Phase 2 (Transform Set)</span><br>
      crypto ipsec transform-set FLEXVPN_TRANSFORM esp-aes 256 esp-sha-hmac<br>
       mode tunnel<br>
      !<br>
      <span style="color: #888;">! 6. IPsec Profile (The Ultimate Glue)</span><br>
      crypto ipsec profile FLEXVPN_PROFILE<br>
       set transform-set FLEXVPN_TRANSFORM<br>
       <span style="color: #ff5555;">set ikev2-profile FLEXVPN_PROFILE</span><br>
      !<br>
      <span style="color: #888;">! 7. Interfaces & Routing</span><br>
      interface Tunnel0<br>
       ip address 10.1.120.20 255.255.255.0<br>
       tunnel source GigabitEthernet0/1<br>
       tunnel destination 192.168.0.10<br>
       tunnel protection ipsec profile FLEXVPN_PROFILE<br>
      !<br>
      interface GigabitEthernet0/1<br>
       ip address 192.168.0.20 255.255.255.0<br>
       no shutdown<br>
      !<br>
      router eigrp 100<br>
       no auto-summary<br>
       network 10.1.120.0 0.0.0.255
    </td>
  </tr>
</table>

---

### 🧠 Deep Dive: The IKEv2 Architecture (Explaining the Code)

Now that we see the configuration, let's break down how IKEv2 changed the rules of the game.

#### 1. The Divorce of "HAGLE" (Proposal vs. Profile)
In IKEv1, we memorized **HAGLE** (Hash, Authentication, Group, Lifetime, Encryption) and put it all in one ISAKMP policy. The creators of IKEv2 (and Cisco in FlexVPN) decided this was too rigid. They split HAGLE into two separate building blocks:

*   **Block 1: Cryptography (`crypto ikev2 proposal`)** - Here you define ONLY how you encrypt (Encryption), how you check integrity (Hash), and how you exchange keys (DH Group).
*   **Block 2: Identity & Time (`crypto ikev2 profile`)** - Here they moved Authentication (PSK or Certificates) and the Lifetime.

> **💡 Conclusion:** HAGLE hasn't disappeared; it just got a divorce! In IKEv2, crypto parameters (H, G, E) live in the *Proposal*, while identity/time parameters (A, L) live in the *Profile*.
> *Statistic:* In large networks, you can have ONE universal proposal (e.g., AES-256) and attach it to 10 different profiles (one for branches, one for remote workers). This reduces crypto config bloat by 60-70%!

#### 2. The AES-GCM Exception
If you configure a modern cipher like AES-GCM in your proposal, you might see an error if you try to add the `integrity` command. 
*Why?* AES-GCM is an **AEAD** (Authenticated Encryption with Associated Data) cipher. It is so smart that it encrypts and checks integrity *at the exact same time*. Therefore, the router forbids the `integrity` command. You only provide the PRF (Pseudo-Random Function) and the DH Group. (Plus, AES-GCM uses about 30% less CPU!).

#### 3. The "Killer Feature": Asymmetric Passwords (`keyring`)
A keyring is a "bag" where we throw our passwords. In IKEv2, you will notice two separate lines for the PSK: `local` and `remote`. 
This means Router A can authenticate to Router B using the password "Secret123", while Router B authenticates to Router A using "TotallyDifferent456". In the old IKEv1, both sides HAD to use the same password. Splitting this in FlexVPN drastically increases security in case one password leaks!

#### 4. DPD (Dead Peer Detection) - `on-demand`
What happens if an ISP excavator cuts the fiber cable? Without DPD, the router might wait 24 hours before realizing the tunnel is dead. DPD sends "Are you alive?" packets. But how does `on-demand` work?
*   **Scenario 1 (Night):** No user traffic. The router sends nothing. DPD is silent. The router assumes the tunnel is fine (why check if no one is using it?).
*   **Scenario 2 (Day):** Users send files, replies come back. DPD is silent. The router knows the neighbor is alive because it sees return user traffic.
*   **Scenario 3 (Outage - DPD kicks in!):** A user sends a file. The router encrypts it and sends it into the tunnel. 1 second passes, 2 seconds, 10 seconds... no reply comes back. The router gets suspicious: *"Wait, I sent real data, and he is silent?"*. ONLY THEN does the router generate a special DPD packet and ask: *"Hello, are you alive?!"*.

#### 🧩 5. Wait, why do we have TWO profiles attached to each other?
Look at Step 6 in the config. We put the `ikev2-profile` INSIDE the `ipsec profile`. Why?
In the old IKEv1, Phase 1 and Phase 2 lived their own separate lives. The router had to guess and dynamically match Phase 1 parameters to Phase 2 based on the IP address. It was chaos.
The creators of IKEv2 said: *"No more chaos!"*. They introduced a hard, direct link. By attaching the IPsec profile to the interface, the router follows the string straight to Phase 1. It knows EXACTLY how to tie everything together. *(Note: This specific CLI hierarchy is a Cisco programmer invention for object-oriented order).*

#### 🌍 6. Where is the IKEv2 Policy attached?
Nowhere! When a peer knocks on UDP port 500, our router scans through all globally configured policies from top to bottom until it finds a match. Cisco IOS-XE also has **Smart Defaults** (built-in algorithms) that will act as a fallback if your policy fails.

#### 🛣️ 7. Routing Note: `no auto-summary`
Why do we use this in EIGRP? It forces the protocol to send VLSM (exact) subnet masks. In the old days, networks were rounded up to their classful boundaries. Imagine having `10.1.1.0/24` in NY and `10.2.2.0/24` in LA. Without this command, both routers would advertise `10.0.0.0/8`, causing a massive routing conflict! *(Since IOS 15, this command is enabled by default).*

---

### 🔍 CLI Troubleshooting & Verification

#### 1. The "Interface Down" Trap
During the lab, I checked my interfaces and saw this:
<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
RouterA#show ip interface brief 
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/1         192.168.0.10    YES manual administratively down down    
Tunnel0                    10.1.120.10     YES manual up                    down 
</pre>
Both tunnels had the protocol `down`. After a moment of panic, I realized the physical `Gi0/1` interfaces were `administratively down` (shutdown). Remember: **If the physical underlay is down, the tunnel overlay will never come up!**

#### 2. Testing with Loopbacks & Traceroute
To properly test the tunnel, I added two Loopbacks simulating LAN subnets (`172.16.1.1/24` on Router A, and `172.16.2.1/24` on Router B) and ran a traceroute:
<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
RouterA#traceroute 172.16.2.1
Type escape sequence to abort.
Tracing the route to 172.16.2.1
  1 10.1.120.20 10 msec 8 msec *
</pre>
It works beautifully! It goes straight through the tunnel IP.

#### 3. The 3 Tables of EIGRP (Where is my route?!)
I checked my EIGRP neighbors:
<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
RouterA#show ip eigrp neighbors 
EIGRP-IPv4 Neighbors for AS(100)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
0   10.1.120.20             Tu0                      13 1d00h      18  1470  0  4
</pre>
*Wait, where is the `172.16.2.0/24` subnet?!* 
It's not there, and it shouldn't be! The `neighbors` command is ONLY a "phonebook" showing the routers you have a direct relationship with. It does not show the subnets they are talking about.

To find the subnets, we must use the other tables. **The EIGRP architecture relies on three independent tables.** According to engineering best practices (troubleshooting methodology), 80% of routing problems are solved by checking them in this exact order:

1.  **Neighbors Table (`show ip eigrp neighbors`):** Is the neighbor there?
2.  **Topology Table (`show ip eigrp topology`):** Did the neighbor send the route to the database?
3.  **Routing Table (`show ip route eigrp`):** Did the route win the algorithm and make it to the main routing table?

Let's verify the Topology and Routing tables:
<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
RouterA#show ip eigrp topology 
P 172.16.2.0/24, 1 successors, FD is 27008000
        via 10.1.120.20 (27008000/128256), Tunnel0

RouterA#show ip route eigrp
D        172.16.2.0/24 [90/27008000] via 10.1.120.20, 00:06:41, Tunnel0
</pre>
Everything is perfect. The route is in the topology database, and it was successfully injected into the main routing table!