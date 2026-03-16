# 💣 IP Fragmentation & DDoS: The "Smart Fridge" Attack

One PC clearing the **DF (Don't Fragment)** bit and sending large packets is like a mosquito bite to a massive ISP router. The router slices it up in a fraction of a second and doesn't even notice. 

But a botnet? That's a whole different league.

### 🧟‍♂️ The "Smart Fridge" Attack Scenario

Imagine a hacker takes over 100,000 poorly secured IoT devices (CCTV cameras, smart bulbs, and even Wi-Fi-enabled fridges).

1. The hacker issues a command: *"Clear the DF bit and send 1500-byte packets to this specific company's IP."*
2. The target company has a WAN link that passes through an interface with an **MTU of 1400**.
3. The edge router suddenly receives millions of packets per second that it **MUST** fragment.
4. The CPU hits 100%. The router stops processing normal traffic (like BGP routing updates), drops connections to the outside world, and the company vanishes from the internet.

> **🌍 Real-World Emotion:** 
> This isn't science fiction. The infamous **Mirai botnet** in 2016 operated on very similar principles (though using various vectors) and took down half the internet in the US.

---

### 🛡️ Modern Defenses: Why this plan wouldn't work today

As Security Engineers, we had to react. Today, if you tried this attack on a modern, well-designed network, you would hit two massive walls:

**1. CoPP (Control Plane Policing)**
This is a feature in Cisco routers that protects the router's "brain" (the CPU). We configure it to say: *"Dear CPU, you can fragment packets, but spend a maximum of 5% of your power on it. If more packets arrive needing fragmentation, ruthlessly drop them and focus on more important things."*

**2. Scrubbing Centers (Traffic Laundromats)**
ISPs have massive Anti-DDoS systems. When they see an unnatural wave of packets requiring fragmentation flying towards you from all over the world, they reroute this traffic to a "laundromat". There, giant machines filter out the botnet garbage, and only clean, normal traffic is forwarded to your router.

---

### 📊 Visualizing the Attack & Defense

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 14px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
[ THE PAST: UNPROTECTED NETWORK ]

(100k Smart Fridges) ===> [ 1500B Packets / DF=0 ] ===> [ EDGE ROUTER ]
                                                           |
                                                    [ CPU Hits 100% ]
                                                    [ BGP Drops     ]
                                                    [ NETWORK DEAD  ]

=========================================================================

[ TODAY: MODERN SECURITY ARCHITECTURE ]

(100k Smart Fridges) ===> [ ISP SCRUBBING CENTER ]  <-- "The Laundromat"
                                 | (Drops Botnet Garbage)
                                 v
                          [ Clean Traffic Only ]
                                 |
                                 v
                          [ EDGE ROUTER ]
                          [ CoPP Enabled: Max 5% CPU for fragmentation ]
                          [ NETWORK SAFE ]
</pre>