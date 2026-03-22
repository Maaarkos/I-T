# 📈 TCP Window Size, Scaling & The 64KB Limit

A computer is an electrical machine. It only understands two states: there is current (1) or there is no current (0). One bit is one "box" where you can only write a 0 or a 1. So, you have 2 possibilities.

### 🧮 The Math of Boxes: Where does 65,535 come from?

When the creators of the TCP protocol took a piece of paper and drew the TCP header, they said: *"Alright, for the Window Size information, we will allocate 16 of these boxes (16 bits)."*

The math works exactly like this:
*   **1 bit** = 2 possibilities
*   **2 bits** = 2^2 = 4 possibilities
*   **8 bits** = 2^8 = 256 possibilities *(This is why subnet masks end at 255, because we count from zero: 0-255).*
*   **16 bits** = 2^16 = 65,536 possibilities.

Because computers always start counting from zero (0 is also a value!), the maximum physical number you can fit into these 16 boxes of ones and zeros (which is `1111111111111111`) equals exactly **65,535** in the decimal system.

> **💡 Engineering Trivia:** 
> Have you noticed that the maximum number of TCP/UDP ports is also 65,535? Guess why! Because the "Port" field in the header was also given exactly 16 bits by the engineers. 
> And why are there only 4.2 billion IPv4 addresses? Because an IP address has 32 bits (2^32 = 4.2 billion). 
> **Everything in networking is simply the mathematics of boxes of a specific size!**

---

### 🪟 What is the TCP Sliding Window?

Before we talk about limits, we need to understand what this "window" actually does. 

TCP uses a mechanism called the **Sliding Window**. It dictates exactly **how much data (in bytes) a computer can send "blindly" over the network before it must stop and wait for an Acknowledgment (ACK)** from the receiver. 
If the window is 64 KB, the sender pushes 64 KB of data into the pipe and hits the brakes. It will not send a single byte more until the receiver says: *"I got it, send the next batch!"*. As ACKs come back, the window "slides" forward, allowing more data to be sent.

### 🐢 The Legacy 64KB Limit

So, what is the standard TCP window size? This requires opening the history book of the internet!

The short answer is: It used to be a rigid 64 KB, but today it can be up to 1 Gigabyte. However, every transfer starts at just... 14 KB.

When the TCP header was designed in the 1980s, the 16 bits allocated for the "Window Size" meant the maximum value was 65,535 bytes (approx. 64 KB).
In the era of dial-up modems, sending 64 KB blindly was more than enough. But when fiber optics arrived, this 64 KB became a massive bottleneck. Computers would send 64 KB in a fraction of a millisecond and then waste precious time waiting for the ACK. Transfers were tragically slow.

---

### 🚀 The Modern Hack: TCP Window Scaling (WS)

Because engineers couldn't simply enlarge the TCP header (it would break the entire internet), they invented a brilliant patch (RFC 1323): **The Window Scale (WS) multiplier.**

If you look at a Wireshark capture of a modern TCP handshake, inside the SYN and SYN-ACK packets, you will see an option at the very end: `WS=256`.

**How does it work?** 
The computer says: *"Hey, my official window in the header is 64 KB, but let's make a deal. Whatever number I put in that field, multiply it by 256!"*
Thanks to this multiplier, today's standard TCP window in Windows or Linux can dynamically swell up to **1 Gigabyte** of data sent blindly without waiting for an ACK!

---

### 🚦 Where do we start? (The Initial Window - initcwnd)

If we can send 1 GB, does an application like `iPerf3` immediately throw a gigabyte into the network? **NO.** That would instantly kill every router along the path.

According to modern standards (RFC 6928), every modern operating system (Windows 10/11, Linux) has an **Initial Window** set to 10 packets.

1.  The transfer begins.
2.  The PC sends exactly 10 packets (10 x 1460 bytes = approx. **14.6 KB**).
3.  It waits for an ACK.
4.  Did the ACK arrive? Great! It doubles the window to 20 packets. Then to 40, 80, 160... (This is called *TCP Slow Start*). It keeps doubling until the link says "enough" (a packet drops) or it reaches the maximum size negotiated by the multiplier.

---

### 😱 Real-World Scenario: The Engineer's Nightmare

Have you ever been in a situation where your company has a 1 Gbps fiber link, you are downloading a file from a server in another country, and it crawls at a pathetic 2 Mbps, even though no one else is using the network?

In 90% of these cases, the culprit is an old or poorly configured firewall along the path. 
Some firewalls inspect the SYN packet and "scrub" or strip away the Window Scaling option (`WS=256`) for "security reasons". 

As a result, your modern computers are forced to fall back to the rigid 64 KB limit from the 1980s. Because the server is in another country (high latency/ping), your computer sends 64 KB instantly and spends 99% of the time just waiting for the ACKs to travel across the world.