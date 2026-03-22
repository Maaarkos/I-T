# 🪚 MSS vs MTU & The Nightmare of Fragmentation

Imagine network encapsulation as an assembly line. The packet is built from the inside (data) to the outside (the cable).

### 📏 The Assembly Line (Scope of Counters)

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
+-------------+-------------+-------------+-----------------------+-------------+
| Layer 2     | Layer 3     | Layer 4     | Layer 7               | Layer 2     |
| (Cable)     | (Routing)   | (Transport) | (Application)         | (Cable)     |
+-------------+-------------+-------------+-----------------------+-------------+
| Ethernet    | IP Header   | TCP Header  | RAW DATA (Payload)    | FCS Trailer |
| 14 Bytes    | 20 Bytes    | 20 Bytes    | 1460 Bytes            | 4 Bytes     |
+-------------+-------------+-------------+-----------------------+-------------+
                            | <------- MSS (1460 Bytes) ------->  |
              | <----------------- MTU (1500 Bytes) ------------------------> |
| <------------------------- FRAME SIZE (1518 Bytes) ---------------------------------> |
</pre>

*   ▶ **MSS (Maximum Segment Size) = 1460 bytes**
    Covers ONLY the Raw Data. This is what the computer asks for in the TCP SYN packet.
*   ▶ **MTU (Maximum Transmission Unit) = 1500 bytes**
    Covers: IP Header (20) + TCP Header (20) + Data (1460). 
    *Golden Rule:* MTU is everything from Layer 3 upwards. This is the limit the router looks at to decide whether to pull out the chainsaw (fragmentation).
*   ▶ **Frame Size = 1518 bytes**
    Covers: Ethernet (14) + MTU (1500) + FCS (4). This is the physical size of the electrical impulses flying over the cable.

---

### 🪆 The IPsec Matryoshka (Tunnel MTU vs Physical MTU)

How does this look when we manipulate `tcp adjust-mss` to make a VPN work properly? We set the mechanism on the router to tell devices in the private network to negotiate a 1360 MSS. Thanks to this, we get a Tunnel MTU of 1400 bytes.

**1. The Inside (What the Tunnel Interface sees):**
<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
+----------------+----------------+--------------------------------+
| Inner IP Hdr   | Inner TCP Hdr  | RAW DATA (Reduced!)            |
| 20 Bytes       | 20 Bytes       | 1360 Bytes                     |
+----------------+----------------+--------------------------------+
| <---------------- TUNNEL MTU (1400 Bytes) ---------------------> |
</pre>
This is our original packet. Before the router starts encrypting it, it checks if it fits in the logical tunnel limit. 1400 bytes is the perfect size because it leaves room for the armor.

**2. The ESP Armor (What the Physical WAN Interface sees):**
Now the router takes that entire 1400 bytes, encrypts it, and wraps it in new headers (Tunnel Mode).
<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
+------------+-------------+-------------------------+-------------+-----------+
| New IP Hdr | ESP Hdr + IV| [ ENCRYPTED ORIGINAL ]  | ESP Trailer | ESP Hash  |
| 20 Bytes   | ~24 Bytes   | 1400 Bytes (Tunnel MTU) | ~12 Bytes   | ~16 Bytes |
+------------+-------------+-------------------------+-------------+-----------+
| <----------------------- PHYSICAL MTU (1472 Bytes) ------------------------> |
</pre>
The physical door of the router (WAN interface MTU) has a limit of 1500 bytes. Our packed, encrypted, fat IPsec packet weighs a total of 1472 bytes. **1472 < 1500. It passes perfectly!**

*(Note: Even though Cisco automatically adjusts MTU when you use the MSS feature, it's always better to set the MTU manually for peace of mind).*

---

### 🎥 What about UDP? (The Video Call Dilemma)

MSS only applies to TCP. What if we want to push a massive amount of UDP video traffic? We have two mechanisms: one network-based (which often fails) and one application-based (which saves the world).

**1. The Network Mechanism: PMTUD (Path MTU Discovery)**
This is the official internet standard. 
*   Your PC sends a large UDP packet (1500B) and intentionally sets the **DF (Don't Fragment)** bit to 1. It says: *"I'm sending this, but I forbid you to cut it!"*
*   The packet hits your router (Tunnel MTU 1400). The router says: *"It's too big, and I'm not allowed to cut it. I'm throwing it in the trash!"*
*   **The Magic:** The router doesn't drop it silently. It sends a small return letter (**ICMP Type 3, Code 4 - Fragmentation Needed**) to your PC: *"Hey PC, I killed your packet. My door is only 1400 bytes. Shrink your packets and try again."* The OS receives this and starts sending smaller packets.
*   **Why engineers hate it:** Overzealous security admins block ALL ICMP on firewalls "for security". The return letter never reaches the PC. The PC keeps sending huge packets, the router keeps killing them, and the video fails (The famous "Black Hole" issue).

**2. The App Mechanism: Smart Software (The Real Solution)**
Since the network can be unreliable and firewalls break PMTUD, app developers (MS Teams, Webex, Zoom) took matters into their own hands. They stopped trusting network engineers!
When you start a video call, the app assumes a worst-case scenario (e.g., there's an IPsec tunnel somewhere) and proactively encodes the video into small chunks, like 1100 or 1200 bytes (RTP payload). Even if a router adds 80 bytes of IPsec headers, it easily passes through any MTU.

> **⚠️ The Tragedy of Cut UDP:**
> If a dumb app sends a huge UDP packet with DF=0, the router cuts it. If piece #1 arrives but piece #2 gets lost in the internet, what happens?
> *   **TCP:** The PC asks for a retransmission.
> *   **UDP:** The PC cannot reassemble the packet without the missing piece. It throws the ENTIRE packet in the trash.
> *   **Result:** You see giant green squares (artifacts) on the screen, the video freezes, and the voice sounds like a robot.

---

### 🧟‍♂️ Hacker Attacks: The Fragmentation Nightmare

Fragmentation is a nightmare for routers in two main ways:
1.  **Cutting (On the fly):** The router must take a large packet, divide it, copy IP headers to each piece, recalculate checksums, and send it. This eats the **CPU**.
2.  **Reassembling (At the destination):** The receiving router must catch all pieces, put them in RAM, wait for the missing parts, and glue them together. This eats **RAM**.

Hackers have invented several brilliant attacks based on the lack of the DF bit:

*   🪓 **The Lumberjack Attack (Killing the CPU):**
    The hacker sends millions of 1500-byte packets (DF=0) through a link they know has an MTU of 1400. The poor ISP router becomes a "lumberjack". It does nothing but cut packets. The CPU spikes to 100%, and normal user traffic drops.
*   💧 **The Teardrop Attack (Killing the RAM / OS):**
    An absolute hit in the 90s. The hacker didn't force the router to cut; they cut the packets themselves and sent them to the victim. 
    *The Catch:* They manipulated the "Fragment Offset" so the pieces overlapped (e.g., piece #2 claimed it started in the middle of piece #1). When an old OS (like Windows 95) tried to glue it together, it got a mathematical error (kernel panic) and threw a Blue Screen of Death.
*   ⏳ **"Waiting for Godot" (Buffer Exhaustion):**
    The hacker sends fragmented packets but intentionally *never* sends the final piece. Your VPN concentrator catches pieces 1, 2, and 3. It puts them in RAM and waits for piece 4. It waits, and waits... The hacker sends millions of these unfinished puzzles. The RAM fills up with "garbage" waiting for endings until the router crashes.

---

### 🛠️ MTU Sweeping & PMTUD (The Engineer's Radar)

When we suspect that a VPN tunnel (or an ISP along the path) is dropping or fragmenting our packets, we don't guess. We use a tool built into every operating system to "probe" the exact size of the pipe. This technique is called **MTU Sweeping**.

**What is PMTUD?**
**PMTUD** stands for *Path MTU Discovery*. It is a standard internet mechanism that allows computers to dynamically discover the narrowest bottleneck (the smallest MTU) on the entire route between the sender and the receiver. This allows the computer to send packets of the correct size from the start and avoid fragmentation.

**How does the `ping -f -l` command work?**
To manually trigger the PMTUD mechanism in Windows, we use the command:
`ping <IP> -f -l <size>`

*   **The `-f` switch (Force DF / Do Not Fragment):** This goes into the IPv4 header (Layer 3) and hardcodes the **DF (Don't Fragment)** bit to 1. It tells all routers along the path: *"If I am too big for your doors, you must kill me, but under no circumstances are you allowed to cut me into pieces!"*
*   **The `-l` switch (Length):** Specifies the size of the payload (the raw data) that we put inside the ICMP packet.

**The Return Letter (How the router reports an error)**
When our giant packet with the `-f` flag hits a router that has an MTU that is too small (e.g., the entrance to an IPsec tunnel), the router kills it. But it doesn't do it quietly!

The router sends a special return letter back to our computer. This is an **ICMP Type 3, Code 4** packet.
*   **Type 3:** Destination Unreachable.
*   **Code 4:** Fragmentation Needed and Don't Fragment was Set.

> **🛑 The Blocked Return Letter (The Silent Killer)**
> Assume normal ping works. You send a large packet (1472 bytes) with the `-f` flag. The packet hits the IPsec router. The router kills it and tries to send you the "Fragmentation Needed" letter.
> If an overzealous firewall along the path blocks this return letter, your Windows console will simply say:
> `Request timed out.`
> Instead of a beautiful, clear MTU error message, you get dead silence. This breaks PMTUD completely!

---

### 🔬 Practical Lab: Sweeping the Firepower Tunnel

Let's look at a real-world example on our Cisco Firepower (FPR) with the IPsec VTI tunnel.

First, we check the MTU of the tunnel interface on the firewall:
<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
FPR# show interface Tunnel1 | include MTU
  Hardware is Virtual Tunnel, MAC address N/A, MTU 1445
      IPsec MTU Overhead : 55
</pre>
The Tunnel MTU is **1445 bytes**.

Now, from a Windows PC behind the firewall, we start our MTU Sweep. We try sending 1417 bytes of data:
<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
C:\Users\Administrator>ping 192.168.99.10 -f -l 1417

Pinging 192.168.99.10 with 1417 bytes of data:
Reply from 192.168.99.10: bytes=1417 time=2ms TTL=128
Reply from 192.168.99.10: bytes=1417 time=2ms TTL=128
</pre>
*Success! It passed through the tunnel.*

Now, let's increase the payload by just 1 byte (to 1418):
<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
C:\Users\Administrator>ping 192.168.99.10 -f -l 1418

Pinging 192.168.99.10 with 1418 bytes of data:
Packet needs to be fragmented but DF set.
Packet needs to be fragmented but DF set.
</pre>
*Boom! The router killed the packet and sent us the ICMP Type 3 Code 4 message.*

**🧮 The Math Behind the Sweep**
Why did 1417 work, but 1418 failed? Let's look at the headers we added to the payload:
*   ICMP Header = **8 bytes**
*   IP Header = **20 bytes**
*   Total Headers = **28 bytes**

If we take our successful payload (`1417`) and add the headers (`28`), we get exactly **1445 bytes**. This matches the Tunnel MTU perfectly! Any single byte more (1418 + 28 = 1446) exceeds the tunnel's limit and triggers the fragmentation error.

### 🛡️ Modern Defenses & The "Smart Fridge" Botnet

If hackers are so smart, why does the internet still work? Engineers had to invent defensive shields:

*   **ASIC Hardware Cutting:** Modern, expensive routers don't cut packets with the main CPU anymore. They have dedicated chips (ASICs) that can slice millions of packets per second without breaking a sweat.
*   **VFR (Virtual Reassembly):** A brilliant feature in firewalls (like Cisco ASA). The firewall looks at the incoming pieces. If it sees a missing ending or overlapping pieces (like Teardrop), it immediately throws the whole puzzle in the trash before passing it to the main CPU.
*   **Rate Limiting:** Routers have built-in limits: *"If I get more than 10,000 fragmented packets per second from one IP, that IP gets banned for 5 minutes."*

**The Botnet Scenario (The Ultimate Test):**
One PC sending large packets is a mosquito bite to an ISP. But what if a hacker takes over 100,000 poorly secured IoT devices (CCTV, smart bulbs, Wi-Fi fridges) and tells them to send 1500B packets (DF=0) to your 1400 MTU link?
*(This is exactly how the famous Mirai botnet took down half the US internet in 2016).*

Today, you would survive this thanks to two massive walls:
1.  **CoPP (Control Plane Policing):** A Cisco feature that protects the router's brain. *"Dear CPU, spend a max of 5% of your power on fragmentation. Drop the rest ruthlessly."*
2.  **Scrubbing Centers (Traffic Laundromats):** ISPs see the unnatural wave of fragmentation traffic, reroute it to a "laundromat", filter out the smart fridge garbage, and send only clean traffic to your router.