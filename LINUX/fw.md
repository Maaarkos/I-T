# 🐧 Linux Firewall: Netfilter, iptables & UFW

The Debian/Ubuntu Linux family relies on the **netfilter** framework, which is a module built directly into the Linux kernel (succeeded by **nftables** in newer releases). 

While you can write firewall rules directly using the `iptables` CLI tool, its syntax can be quite complex and hard to memorize. To solve this, the **UFW (Uncomplicated Firewall)** wrapper was created. It acts as a user-friendly frontend for `iptables`/`nftables`, making firewall management incredibly easy.

### 🛠️ Installation & Basic Configuration

First, you need to install the UFW service and enable it:

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 15px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
root@linux-server:~# apt update
root@linux-server:~# apt install ufw
root@linux-server:~# ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
</pre>

*(Note: Always make sure to allow SSH before enabling the firewall on a remote server to avoid locking yourself out!)*

### 🔓 Allowing Ports

If you want to allow specific traffic, for example, web traffic on HTTP (80) and HTTPS (443):

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 15px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
root@linux-server:~# ufw allow 80,443/tcp
Rule added
Rule added (v6)
</pre>

### 📊 Checking Firewall Status

To verify that your rules have been applied correctly, use the status command. It will show you exactly what is currently allowed or denied:

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 15px; border-radius: 8px; border: 1px solid #444; line-height: 1.2;">
root@linux-server:~# ufw status
Status: active

To                         Action      From
--                         ------      ----
80,443/tcp                 ALLOW       Anywhere
80,443/tcp (v6)            ALLOW       Anywhere (v6)
</pre>