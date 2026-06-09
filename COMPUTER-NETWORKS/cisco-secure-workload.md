# 🏢 Cisco Secure Workload (formerly Tetration)

**Cisco Tetration** is now officially known as **Cisco Secure Workload**. 

*Trivia:* The original name "Tetration" comes from mathematics (tetration is the next hyperoperation after exponentiation). Cisco chose this name to emphasize the massive, Big Data scale of the platform, as it is designed to process and analyze billions of packets in real-time.

### 🔍 Secure Workload vs. Secure Network Analytics (Stealthwatch)

To truly understand this platform, it is best to compare it to Stealthwatch.

*   **Cisco Secure Network Analytics (Stealthwatch):** Dedicated primarily to Enterprise networks (Campus/LAN). It relies on NetFlow exported by standard routers and switches. It tells you *who* connected to *where* and on *what port*.
*   **Cisco Secure Workload (Tetration):** Dedicated strictly to the **Data Center** and Cloud environments. 

**Where does Secure Workload get its telemetry?**
1.  **Hardware:** Modern Nexus switches have built-in ASIC chips specifically designed to stream deep telemetry directly to Tetration.
2.  **Software Agents:** We install lightweight agents directly on Virtual Machines (Linux or Windows) and bare-metal servers.

**The "Agent" Advantage:**
While Stealthwatch only sees the network flow (IPs and Ports), Secure Workload's agent looks deep into the Operating System. It knows exactly **which specific process** (e.g., `nginx.exe` or `python.exe`) initiated that network connection!
Furthermore, because the agent lives in the OS, Secure Workload can actively **block traffic** by dynamically modifying the host's local firewall (e.g., `iptables` in Linux or Windows Firewall).

> **💡 Summary:** Cisco Secure Workload is the ultimate tool for enforcing **Microsegmentation** across an entire Data Center (VMs, Public Cloud, Bare-Metal Servers).

---

### 🏛️ Deployment Architecture (The Brain)

The Secure Workload cluster (the brain) can be deployed in two ways:

1.  **On-Premises (Local):** You buy massive, dedicated physical servers (often referred to as a "Tetration Rack") or deploy Virtual Machines (Tetration-V) directly inside your own Data Center.
2.  **SaaS (Cloud):** The management cluster lives in the Cisco Cloud, but it manages your local Data Center. 
    *   *Security Note:* We obviously do not open inbound firewall ports from the internet to our servers! Instead, we install a **Secure Connector** inside our network. This connector initiates an *outbound* connection to the cloud, keeping the firewall happy and the network secure.

---

### 🗺️ ADM (Application Dependency Mapping)

This is the killer feature of the platform.

If you implement Zero Trust and block everything on day one, you will inevitably break critical business services because no human knows exactly which server talks to which database on which port in a massive Data Center.

**How ADM solves this:**
1.  You give the system about 2 weeks to just "listen and learn".
2.  Tetration maps out every single connection, drawing a beautiful visual map of your application dependencies.
3.  It allows you to simulate policies "dry run" (*What-If analysis*). It shows you exactly what *would* break if you enforced a specific rule today.
4.  Once you are happy with the simulated results, Tetration **automatically generates the exact firewall rules** needed to secure the application.

> **🎙️ The Engineer's Definition:**
> ADM is like an automated security analyst who discovers exactly how your applications work and writes safe, micro-segmented firewall rules for them, so you don't have to guess in the dark.
> ADM is like an automated security analyst who discovers exactly how your applications work and writes safe, micro-segmented firewall rules for them, so you don't have to guess in the dark.