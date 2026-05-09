# 🔀 Remote Access VPN: Split Tunneling & Split DNS

When configuring a Remote Access VPN (like Cisco AnyConnect), one of the most critical architectural decisions is how to route the user's traffic. We have two main options: **Full Tunnel** (No Split) and **Split Tunneling**.

### 🗺️ Traffic Flow: Full Tunnel vs. Split Tunnel

Here is a visual representation of how the traffic flows in both scenarios:

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 13px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto; white-space: pre;">
[ SCENARIO 1: FULL TUNNEL (No Split) ]

                       [ REMOTE LAPTOP ]
                               |
                               | (ALL Traffic: Corporate + YouTube/Netflix)
                               v
                     [ LOCAL HOME ROUTER ]
                               |
                     ( IPsec / SSL TUNNEL )
                               |
                               v
                    [ CORPORATE FIREWALL ]
                       /              \
     (Corporate Traffic)              (Internet Traffic)
             /                              \
            v                                v
      [ CORP LAN ]                      [ INTERNET ]
    (e.g., 10.0.0.0/8)               (Traffic hairpins out of the HQ)


=================================================================================


[ SCENARIO 2: SPLIT TUNNELING ]

                       [ REMOTE LAPTOP ]
                         /           \
      (Corporate Traffic)             (Internet Traffic)
      (e.g., 10.0.0.0/8)              (e.g., YouTube, Netflix)
              /                               \
             v                                 v
   ( IPsec / SSL TUNNEL )             [ LOCAL HOME ROUTER ]
             |                                 |
             v                                 v
   [ CORPORATE FIREWALL ]                 [ INTERNET ]
             |                          (Goes directly to the web)
             v
       [ CORP LAN ]
</pre>

---

### ✂️ What is Split Tunneling?

*   **Full Tunnel (No Split):** Everything goes through the VPN server. It doesn't matter if the user is accessing an internal corporate server or watching Netflix. The laptop encrypts it, sends it to the HQ firewall, and the firewall has to route the Netflix traffic back out to the internet.
*   **Split Tunneling:** Only traffic destined for private/corporate resources goes through the VPN. When the user browses the public internet, the traffic bypasses the VPN and goes directly out their local home router.

#### The Pros & Cons of Split Tunneling
*   ✅ **The Pros (Performance):** It massively offloads the Corporate Firewall's CPU and internet bandwidth. You don't waste expensive corporate bandwidth encrypting and decrypting employees' YouTube videos.
*   ❌ **The Cons (Security Risk):** If the laptop gets infected by malware from the public internet (because it's bypassing corporate web filters), the attacker can use the active VPN tunnel to pivot directly into the corporate network! Therefore, if you use Split Tunneling, **robust Endpoint Protection (EDR/XDR) on the laptop is absolutely mandatory.**

#### How does it work under the hood?
When the VPN connects, the firewall pushes a "Care Package" to the AnyConnect client. This package contains specific routes (defined via ACLs or GUI on the firewall). AnyConnect injects these routes directly into the Windows/Mac routing table. 
> **💡 Pro-Tip:** You can verify exactly which routes the VPN endpoint is using by opening the Windows Command Prompt and typing:
> `C:\> route print`

---

### 🗂️ What is Split DNS?

If you implement Split Tunneling for IP traffic, you almost certainly need to implement **Split DNS** as well. 

Split DNS separates DNS queries based on the domain name:
*   Queries for `*.mycompany.local` are sent **through the tunnel** to the Corporate DNS server.
*   General queries (like `facebook.com`) are sent **locally** to the ISP's DNS server.

**The Nightmare without Split DNS:**
Imagine you have Split Tunneling enabled (so internet traffic goes local), but the VPN injects the Corporate DNS server as the *primary* DNS for ALL queries. 
If the Corporate DNS server blocks public resolution, the user won't be able to resolve any public websites. Even if it allows it, you are causing unnecessary latency by sending every single DNS query for public websites all the way to the HQ and back.

> **🎙️ The Ultimate Summary:**
> *   **Split Tunneling** decides which **IP traffic** goes through the VPN (Corporate IPs vs. Public IPs).
> *   **Split DNS** decides which **DNS queries** go through the VPN (Corporate domains vs. Public domains).