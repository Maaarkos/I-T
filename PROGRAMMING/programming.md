# 💻 Programming, Python Basics & SDN Architecture

### Why do compiled languages (like C++, Go) need to be translated upfront?

They require a "Build" (Compilation) phase. Before you can even run the program, a special tool (the compiler) takes your entire source code and translates it all at once into a ready-made machine code file (e.g., an `.exe` file in Windows). Only this finished, ready-to-run file is deployed to the server.

### Why is Python different?

Python is a scripting (interpreted) language. There is no "Build" phase here because the code is translated on the fly. 

When you run a script, a special program installed on the server (called the Python Interpreter) reads your `.py` text file line by line. In a fraction of a second, it translates the code into zeros and ones for the CPU in real-time as the program executes.

---

## 🐍 Python Basics for Network Engineers

When writing automation scripts (e.g., to connect to Cisco Catalyst Center via API), you don't have to write everything from scratch. You use libraries (ready-made modules of code). 

### 1. What is the famous `pip`?

Think of `pip` as the "Google Play" or "App Store", but for Python. 

When you type a command like `pip install requests` in your terminal, the system automatically reaches out to the internet, downloads the ready-made code package, and installs it on your computer.

<div align="center">
  <a href="IMAGES/pip-install.png" target="_blank">
    <img src="IMAGES/pip-install.png" style="max-width: none; width: 400px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div> 

The image above shows the download and installation of the `requests` library. This specific library is essential for handling HTTP/HTTPS connections, allowing our script to talk to REST APIs (like Cisco Catalyst Center).

### 2. The `import` command

Just because a library is installed on your hard drive doesn't mean your script knows about it. You must use the `import` command.

<div align="center">
  <a href="IMAGES/pip-import.png" target="_blank">
    <img src="IMAGES/pip-import.png" style="max-width: none; width: 400px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div> 

The `import` command loads the library from your hard drive into the RAM, making its functions available for your script to use.

### 3. Data Formats & The Built-in `json` Library

When interacting with modern network APIs (like Catalyst Center via RESTful API), the data is almost always exchanged in a specific format called **JSON**. 

In our Python script, we need a "translator" to convert raw text into a structured JSON object that Python can understand (and vice versa). 

<div align="center">
  <a href="IMAGES/tlumacz.png" target="_blank">
    <img src="IMAGES/tlumacz.png" style="max-width: none; width: 400px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div> 

*Notice:* We do not need to use `pip install json` to download this library. The `json` library is built directly into the core Python installation! All we have to do is `import json` at the top of our script, and the translator is ready to work.

---

## 🏗️ SDN Architecture & The API Landscape

To understand how our Python scripts actually talk to the network, we must look at the **SDN (Software-Defined Networking)** architecture. 

In SDN, the Control Plane (the brain) is separated from the Data Plane (the forwarding muscles). The Controller sits in the middle, and we use different APIs to talk to the Controller vs. talking to the actual switches.

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 13px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto; white-space: pre;">
                           [ NETWORK APPLICATIONS / SCRIPTS ]
                           (Python, Postman, Custom Dashboards)
                                            |
                                            |
==========================================================================================
  NORTHBOUND APIs              (REST API, JSON, XML)
==========================================================================================
                                            |
                                            v
                           +----------------------------------+
                           |          SDN CONTROLLER          |
                           | (Cisco Catalyst Center, SD-WAN)  |
                           +----------------------------------+
                                            |
                                            |
==========================================================================================
  SOUTHBOUND APIs       (NETCONF, RESTCONF, gNMI, OpenFlow, SNMP, CLI)
==========================================================================================
                                            |
                                            v
                           +----------------------------------+
                           |         NETWORK DEVICES          |
                           |      (Routers, Switches, APs)    |
                           |                                  |
                           |      [ CONTROL PLANE ]           |
                           |      [ DATA PLANE    ]           |
                           +----------------------------------+
</pre>

*   **Northbound APIs:** Used by your Python scripts to talk to the Controller. They are highly abstracted and almost always use **REST APIs** with JSON.
*   **Southbound APIs:** Used by the Controller to talk to the physical devices. They do the heavy lifting of pushing configurations and pulling telemetry.

---

## 🚀 Deep Dive: gNMI & Streaming Telemetry

Among the Southbound APIs, there is a new, absolute king: **gNMI (gRPC Network Management Interface)**.

Since the industry already had a super-fast courier protocol (**gRPC**, developed by Google) and universal configuration templates (**OpenConfig/YANG**), they needed to combine them. 
gNMI is simply a network management protocol built specifically on top of the gRPC framework.

### 🎯 Why is gNMI so important? (Exam Hook - Telemetry!)

The greatest strength of gNMI is *not* configuring routers. Its absolute superpower is **Streaming Telemetry**.

*   **The Old Way (SNMP Polling):** The server asks the router every 5 minutes: *"What is your CPU usage?"*. The router replies. This is slow, resource-heavy, and leaves massive 5-minute blind spots where CPU spikes can go unnoticed.
*   **The New Way (gNMI Streaming Telemetry):** You use gNMI to tell the router: *"Open a fast gRPC tunnel and continuously spit out your stats every single second in a lightweight binary format."* 

The router pushes the data proactively. This is the absolute future of network monitoring, providing real-time, microsecond visibility into the network!