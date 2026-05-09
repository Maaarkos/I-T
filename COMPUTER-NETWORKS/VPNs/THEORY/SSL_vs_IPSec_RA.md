# 💻 Remote Access VPN: SSL vs. IPsec (IKEv2)

When deploying a Remote Access (RA) VPN (e.g., using Cisco AnyConnect / Secure Client), the eternal question arises: Should we use SSL VPN or IPsec VPN? 

### 🥇 The Golden Rule (Ports & Firewalls)

The choice often comes down to what ports are open on the networks your users are connecting from (hotels, airports, cafes).
*   **IPsec (IKEv2)** requires UDP port **500** and UDP port **4500** (for NAT-T). Many restrictive public Wi-Fi networks block these ports.
*   **SSL VPN** requires TCP port **443**. In 99.99% of cases, port 443 is wide open because it is the standard port for HTTPS web browsing. If you block 443, the internet stops working.

Therefore, SSL VPN is often chosen as the "guaranteed to work anywhere" fallback. However, if IPsec ports are open, **IPsec is technically far superior.** Here is why.

---

### 🐢 The SSL Problem: TCP Meltdown

If you use a standard SSL VPN, the entire tunnel is built using the TCP protocol. This creates a massive architectural flaw known as **TCP over TCP (TCP Meltdown)**.

Imagine you are downloading a file (TCP) inside your SSL VPN tunnel (also TCP). 
If a single packet gets lost on the internet:
1.  Your inner TCP (the file download) notices the drop and wants to retransmit it.
2.  Your outer TCP (the SSL tunnel) *also* notices the drop and wants to retransmit it!
They start fighting each other. The congestion control timers go crazy, retransmissions multiply exponentially, and the speed of your VPN drops to near zero.

#### The Cisco Fix: DTLS (Datagram Transport Layer Security)
To fix this, in 2006, engineers figured out how to modify the TLS protocol so it could encrypt loose UDP packets instead of requiring a strict TCP connection. Cisco was the first to implement this in AnyConnect.

When you connect via Cisco AnyConnect using SSL, it actually builds two tunnels:
1.  **Control Tunnel (TCP 443):** Used for authentication and setup (guaranteed to pass firewalls).
2.  **Data Tunnel (UDP 443 / DTLS):** Used to push the actual user data. Because it uses UDP, it completely avoids the "TCP Meltdown" problem, giving you the stability of TCP and the speed of UDP.
*(Note: If the strict hotel firewall blocks UDP 443, AnyConnect will gracefully fall back to the slow TCP 443 tunnel).*

---

### 🚀 Why IPsec (IKEv2) is the Ultimate Winner

If the network allows it, you should always prefer IKEv2 for Remote Access. Here are two massive reasons why:

#### 1. Battery Life (Mobile Devices)
SSL VPN is "chatty". It constantly has to check if the other side is alive using keepalives, which keeps the radio antenna active. IPsec (IKEv2) is much more efficient. By using IKEv2 on mobile devices (phones/laptops), you can save **15% to 30% of battery life** compared to SSL!

#### 2. The Killer Feature: MOBIKE (Mobility and Multihoming)
Imagine you are working on your laptop on a train. You switch from the train's Wi-Fi to your phone's LTE hotspot.
*   **With SSL VPN:** The TCP session breaks. The VPN disconnects, prompts you to reconnect, and the router's CPU has to perform a heavy cryptographic handshake all over again.
*   **With IKEv2 (MOBIKE):** IKEv2 says: *"Oh, you changed your IP from Wi-Fi to LTE? Cool, no problem."* Thanks to the MOBIKE extension, IKEv2 **does not drop the tunnel!** It simply sends one tiny, lightweight informational packet: *"Hey Router, it's me again. I have the exact same encryption keys, but my new IP address is X.X.X.X."* The router accepts it, and traffic continues flowing instantly. Zero key renegotiation, zero CPU spike!

---

### 📊 Summary: Which one to choose?

| Feature | SSL VPN (TLS/DTLS) | IPsec (IKEv2) |
| :--- | :--- | :--- |
| **Firewall Traversal** | Excellent (TCP 443 is never blocked). | Poor (UDP 500/4500 often blocked in hotels). |
| **Performance** | Good (if DTLS is active). Terrible if forced to TCP. | Excellent (Native kernel processing). |
| **Mobile Roaming** | Drops connection on IP change. | **MOBIKE** (Seamless transition Wi-Fi <-> LTE). |
| **Battery Usage** | High | Low (Saves up to 30%). |

> **💡 The Engineer's Verdict:** 
> Always configure your Remote Access VPN to prefer **IKEv2**. If the user is in a restrictive network that blocks IPsec, configure the client to automatically fall back to **SSL (DTLS)**.