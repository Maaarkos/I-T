# 🛠️ LAB: VTI over IPsec (The Evolution of GRE)

**Download the official lab instructions here:**
[📥 Download Lab: 16.1.5 Lab - Implement IPsec VTI Site-to-Site VPNs](../16.1.4_Lab_GRE_over_IPsec.docx)
*(Note: Ensure the filename on your disk has no spaces, e.g., `16.1.4_Lab_GRE_over_IPsec.docx`, for the link to work properly).*

---

This lab is practically identical to the **GRE over IPsec** lab. The concept of using an IPsec Profile (the "glue") remains exactly the same. 

However, we are evolving the architecture. Instead of wrapping our data in GRE and then encrypting it, we are going to remove the GRE wrapper entirely and let IPsec handle both the tunneling and the encryption natively. This is what we call a **VTI (Virtual Tunnel Interface)**.

To make this transition, we only need to remember a few cosmetic, yet crucial, changes in the configuration.

---

### 🔄 The 3 Cosmetic Changes (From GRE to VTI)

1.  **Change to Tunnel Mode:** In GRE over IPsec, we used `mode transport` because GRE already provided the new IP header. Since we are removing GRE, IPsec must now provide that new IP header itself. Therefore, the Transform Set must be changed to `mode tunnel`.
2.  **Change the Tunnel Mode:** By default, a Cisco tunnel interface operates as a GRE tunnel. We must explicitly tell the interface to stop using GRE and switch to native IPsec encapsulation using the command `tunnel mode ipsec ipv4`.
3.  **Apply the Profile:** Just like before, we create a Transform Set, attach it to a Profile, and attach that Profile to the Tunnel interface.

---

### ⚖️ Configuration Comparison (GRE Profile vs. VTI)

Here is a side-by-side comparison of Router R3. The Phase 1 (IKEv1) configuration is identical in both scenarios, so the focus here is strictly on Phase 2 and the Tunnel Interface.

Notice how we simply "swap out" the GRE encapsulation for native IPsec.

<table style="width:100%; border-collapse: collapse; font-family: monospace; font-size: 13px;">
  <tr style="background-color: #333; color: #00ff00; text-align: left;">
    <th style="padding: 10px; border: 1px solid #555; width: 50%;">Router R3 (GRE over IPsec Profile)</th>
    <th style="padding: 10px; border: 1px solid #555; width: 50%;">Router R3 (Pure VTI)</th>
  </tr>
  <tr>
    <td style="padding: 10px; border: 1px solid #555; vertical-align: top; background-color: #000; color: #e6e6e6;">
      <span style="color: #888;">! Phase 1 (IKEv1) remains the same...</span><br><br>
      <span style="color: #888;">! 1. Phase 2 (Transform Set)</span><br>
      R3(config)# crypto ipsec transform-set GRE-VPN esp-aes 256 esp-sha256-hmac<br>
      <span style="color: #ff5555;">R3(cfg-crypto-trans)# mode transport</span><br><br>
      <span style="color: #888;">! 2. The IPsec Profile (The Glue)</span><br>
      R3(config)# crypto ipsec profile GRE-PROFILE<br>
      R3(ipsec-profile)# set transform-set GRE-VPN<br><br><br>
      <span style="color: #888;">! 3. The Tunnel Interface</span><br>
      R3(config)# interface Tunnel1<br>
      R3(config-if)# bandwidth 4000<br>
      R3(config-if)# ip address 172.16.1.2 255.255.255.252<br>
      R3(config-if)# ip mtu 1400<br>
      R3(config-if)# tunnel source 64.100.1.2<br>
      R3(config-if)# tunnel destination 64.100.0.2<br>
      <span style="color: #ff5555;">! (By default, it is 'tunnel mode gre ip')</span><br><br>
      R3(config-if)# tunnel protection ipsec profile GRE-PROFILE
    </td>
    <td style="padding: 10px; border: 1px solid #555; vertical-align: top; background-color: #000; color: #e6e6e6;">
      <span style="color: #888;">! Phase 1 (IKEv1) remains the same...</span><br><br>
      <span style="color: #888;">! 1. Phase 2 (Transform Set)</span><br>
      R3(config)# crypto ipsec transform-set VTI-VPN esp-aes 256 esp-sha256-hmac<br>
      <span style="color: #00ff00;">R3(cfg-crypto-trans)# mode tunnel</span><br><br>
      <span style="color: #888;">! 2. The IPsec Profile (The Glue)</span><br>
      R3(config)# crypto ipsec profile VTI-PROFILE<br>
      R3(ipsec-profile)# set transform-set VTI-VPN<br><br><br>
      <span style="color: #888;">! 3. The Tunnel Interface</span><br>
      R3(config)# interface Tunnel1<br>
      R3(config-if)# bandwidth 4000<br>
      R3(config-if)# ip address 172.16.1.2 255.255.255.252<br>
      R3(config-if)# ip mtu 1400<br>
      R3(config-if)# tunnel source 64.100.1.2<br>
      R3(config-if)# tunnel destination 64.100.0.2<br>
      <span style="color: #00ff00;">R3(config-if)# tunnel mode ipsec ipv4</span><br><br>
      R3(config-if)# tunnel protection ipsec profile VTI-PROFILE
    </td>
  </tr>
</table>

> **💡 The Golden Rule of Profiles:**
> The logic is always a chain of three links:
> 1. You create a **Transform Set** (The Armor).
> 2. You attach the Transform Set to a **Profile** (The Glue).
> 3. You attach the Profile to the **Tunnel Interface** (The Pipe).