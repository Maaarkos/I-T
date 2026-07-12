# 🔵🍄 Smurf Attack: ICMP Amplification

The **Smurf Attack** is a classic, highly clever Distributed Denial of Service (DDoS) attack. It exploits the standard ICMP protocol (ping) and network broadcast addresses to overwhelm a target.

> **🍕 The Pizza Analogy:**
> It works exactly like a malicious prank where you call a pizzeria and order 100 pizzas, but you give them your neighbor's address. The pizzeria (the amplifier) does the heavy lifting, and your neighbor (the victim) gets overwhelmed by the delivery guys.

---

### 💥 How it works (Step-by-Step)

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 13px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto;">
[ HACKER ] --- 1. (1 Ping, Spoofed Src: VICTIM) ---> [ AMPLIFIER ROUTER ]
                                                            |
                                                 2. (Broadcasts to 200 PCs)
                                                            |
                                                            v
[ VICTIM ] <--------- 3. (200 Ping Replies!) --------- [ 200 PCs ]
</pre>

**1. IP Spoofing (The Disguise)**
The hacker creates an `ICMP Echo Request` (a standard Ping). However, instead of putting their own IP address as the sender, they spoof the Source IP and insert the **Victim's IP address**.

**2. The Amplifier (The Broadcast)**
The hacker doesn't send this ping directly to the victim. Instead, they send it to the **Broadcast address** of a large, third-party network (e.g., `192.168.1.255`).

**3. The Hit (Flooding the Victim)**
The router in that third-party network receives the packet destined for the broadcast address. Following standard network rules, it forwards the ping to *every single computer* in that subnet (e.g., 200 computers). 
All 200 computers politely reply to the ping by sending an `ICMP Echo Reply`. But because the hacker spoofed the sender's address, all 200 replies don't go back to the hacker—they fly straight at the victim!

**The Result:** The hacker sent one tiny packet, and the victim received 200 packets in the exact same second. The victim's internet link gets choked, and their network crashes. This is called an *Amplification Attack*.

---

### 🛡️ Mitigation: How do we stop it?

To prevent your network from being used as an amplifier for Smurf attacks, a very simple command is used on Cisco router interfaces:

<pre style="background-color: #000000; color: #ffffff; padding: 15px; font-size: 15px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
Router(config-if)# no ip directed-broadcast
</pre>

This command stops the router from allowing anyone from the outside internet to send packets to an internal broadcast address. It simply drops them. 
*(Note: In all modern Cisco IOS versions, this command is enabled by default!)*

---

### 🕰️ Historical Context

In the 1990s, the Smurf attack was devastating because many end devices had public IP addresses and routers allowed directed broadcasts. 

Today, thanks to the widespread use of **NAT** (hiding PCs behind a single public IP) and the `no ip directed-broadcast` command being the default, this attack is mostly a relic of the past on the global internet. 
However, a Smurf attack can still be highly dangerous if executed by a malicious insider from *within* a Local Area Network (LAN)!