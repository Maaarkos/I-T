# 📜 The History of IT: From ENIAC to the Internet

<div align="center">
  <a href="IMAGES/history.png" target="_blank">
    <img src="IMAGES/history.png" style="max-width: none; width: 1200px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

Sometimes, to understand how modern networks work, we have to look back at how they were built. Here is a brief history of the Internet, protocols, and the very first computers.

---

### 🌍 The Birth of the Internet (ARPANET & TCP/IP)

**ARPANET** stands for *Advanced Research Projects Agency Network*. It was created by ARPA (today known as DARPA)—the US military agency for advanced research projects. 

While the internet was born from this military project in the late 1960s, the official "birthday" of the modern internet is considered to be **January 1, 1983**. On this day, all computers on the network transitioned to the TCP/IP protocol suite. 

The "Fathers of the Internet" are two brilliant engineers: **Vint Cerf** and **Bob Kahn**. They are the ones who invented IP addresses, TCP, UDP, and the concept of fragmentation we deal with today!

### 🚂 The Dark Ages Before 1983 (Did they only use Layer 2?)

*Short answer: No, they had higher layers, but they were "dumb" and couldn't connect different networks together.*

Before TCP/IP, ARPANET used a protocol called **NCP (Network Control Program)**. It was a primitive ancestor of Layers 3 and 4. It worked well, but it had one massive flaw: it could only support one specific network (ARPANET).

If another university built its own satellite network (like ALOHAnet in Hawaii) or a radio network (PRNET), NCP couldn't talk to them. 
> **💡 The Train Analogy:** It was as if every country had different railway track gauges, meaning trains simply couldn't cross borders.

Vint Cerf and Bob Kahn invented TCP/IP precisely to create **"Internetworking"** (hence the word *Internet*!)—a universal language that allowed all these different, isolated networks to connect into one global web.

### 🐎 What about Layer 2 (Ethernet)?

If TCP/IP was born in 1983, what was happening at Layer 2? 
Layer 2 (Ethernet and MAC addresses) was actually invented much earlier, in **1973**, by a brilliant engineer named **Bob Metcalfe** at the Xerox PARC laboratories. 

He figured out how computers in a single building could send data to each other over a single coaxial cable without jamming each other's signals (the CSMA/CD protocol). 
So yes, Ethernet (L2) was already fully built and waiting for TCP/IP (L3/L4) to jump on its back and ride it like a horse!

---

### 🖥️ The First Electronic Computer (1945)

Long before the internet, we needed computers. The first true electronic computer (as we understand the concept today) was built in 1945.

It was called **ENIAC** (*Electronic Numerical Integrator and Computer*). It was built in the USA by John Mauchly and J. Presper Eckert specifically to calculate artillery shell trajectories for the military.

> **🐛 Physics Trivia & The Origin of the "Bug"**
> ENIAC did not have a single transistor inside it... because transistors hadn't been invented yet! Instead, it used **18,000 vacuum tubes** (like the ones you see in vintage guitar amplifiers).
> 
> It occupied an entire massive room, weighed 27 tons, and consumed so much electricity that when it was turned on, the lights in the nearby city would dim. 
> Because these glass tubes ran so hot, they constantly burned out. Furthermore, real moths and insects would fly into the machine, getting fried and causing electrical short circuits. This is exactly where the term **"Bug"** comes from when we talk about errors in software or hardware today!