# 💻 Programming, APIs & SDN Architecture

### Why do compiled languages (like C++, Go) need to be translated upfront?
They require a "Build" (Compilation) phase. Before you can even run the program, a special tool (the compiler) takes your entire source code and translates it all at once into a ready-made machine code file (e.g., an `.exe` file in Windows). Only this finished, ready-to-run file is deployed to the server.

### Why is Python different?
Python is a scripting (interpreted) language. There is no "Build" phase here because the code is translated on the fly. 
When you run a script, a special program installed on the server (called the Python Interpreter) reads your `.py` text file line by line. In a fraction of a second, it translates the code into zeros and ones for the CPU in real-time as the program executes.

---

## 🐍 Python Basics for Network Engineers

When writing automation scripts (e.g., to connect to Cisco Catalyst Center via API), you don't have to write everything from scratch. You use libraries (ready-made modules of code). 

*   **`pip` (The App Store):** When you type `pip install requests` in your terminal, the system downloads a ready-made code package from the internet. The `requests` library is essential for handling HTTP/HTTPS connections to REST APIs.
*   **`import`:** Just because a library is installed on your hard drive doesn't mean your script knows about it. You must use the `import` command to load it into RAM.
*   **`json` (The Translator):** When interacting with modern APIs, data is exchanged in the **JSON** format. We don't need to `pip install` it because it is built into Python. We just type `import json` to translate raw text into structured JSON objects.

---

## 🏗️ SDN Architecture & The API Landscape

To systematize our knowledge, we must look at the **SDN (Software-Defined Networking)** architecture. 

In SDN, the Control Plane (the brain) is separated from the Data Plane (the forwarding muscles). The Controller sits in the middle. We use completely different APIs to talk to the Controller (Northbound) versus talking to the actual switches (Southbound).

<pre style="background-color: #000000; color: #00ff00; padding: 15px; font-size: 13px; border-radius: 8px; border: 1px solid #444; line-height: 1.2; overflow-x: auto; white-space: pre;">
                           [ NETWORK APPLICATIONS / SCRIPTS ]
                           (Python, Postman, Custom Dashboards)
                                            ^
                                            |
==========================================================================================
  NORTHBOUND APIs:   [ REST API (JSON) ]   [ SOAP (XML) ]   [ GraphQL ]
  API Docs/Specs:    [ Swagger/OpenAPI ]   [ WSDL ]         [ WADL ]
==========================================================================================
                                            ^
                                            |
                           +----------------------------------+
                           |          SDN CONTROLLER          |
                           | (Cisco Catalyst Center, SD-WAN)  |
                           +----------------------------------+
                                            |
                                            v
==========================================================================================
  SOUTHBOUND APIs:
  Management:  [ NETCONF ]   [ RESTCONF ]   [ gNMI / gRPC ]  --> (Uses YANG Data Models)
  Controller:  [ OpenFlow ]  [ OpFlex (ACI) ]
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

---

### ⬆️ Northbound APIs (Talking to the Controller)

*   **REST API:** The absolute most popular standard today. It almost exclusively uses the **JSON** data format.
*   **SOAP (Simple Object Access Protocol):** An older standard originally designed by Microsoft. It uses ONLY the **XML** format.
*   **GraphQL:** A query language designed for mobile apps and online dashboards. It allows the client to request exactly the specific data it needs, nothing more.

#### API Instruction Manuals (Specifications)
How do we know what URLs to query or what data an API expects? We use API specifications:
*   **Swagger (OpenAPI):** The modern framework and instruction manual for REST APIs (used heavily in Cisco DNA/Catalyst Center and ISE). It is the foundation of the OpenAPI Specification (OAS).
*   **WSDL:** The instruction manual for the old SOAP protocol (XML only).
*   **WADL:** An old attempt to create an instruction manual for REST APIs using XML. Swagger won this war.

---

### ⬇️ Southbound APIs (Talking to the Hardware)

#### Controller-Driven Protocols
*   **OpFlex:** The heart of the Cisco ACI architecture. It pushes policies down to the leaf switches.
*   **OpenFlow:** The original SDN protocol. It completely "decapitates" the switches by stripping away their Control Plane. Only the central Controller has the brain to make routing decisions.

#### Management Protocols (The Big Three)
To manage devices programmatically, we use three main protocols. **All of them rely on YANG.**
> **What is YANG?** YANG is a data modeling language. It provides strict templates. Devices on both ends must use the exact same YANG templates so they can understand each other's configurations perfectly.

---

### 📌 NETCONF vs RESTCONF (The SCOR Exam Cheat Sheet)

Both protocols are used for network automation and rely on YANG models as a strict contract for data. However, they operate very differently:

| Feature | 🛠️ NETCONF (The Hardcore Legacy) | 🌐 RESTCONF (The Modern Web Approach) |
| :--- | :--- | :--- |
| **Transport** | **SSH** (Dedicated TCP Port **830**). | **HTTPS** (TCP Port **443**). |
| **Data Format** | **XML ONLY**. | **JSON** or **XML** (Chosen via HTTP Headers). |
| **Architecture**| **Stateful** - Establishes a continuous, persistent session with the router. | **Stateless** - Every query is an independent action. No persistent session. |
| **Operations** | Based on **RPC** (Remote Procedure Calls), e.g., `<get-config>`, `<edit-config>`. | Uses standard HTTP methods (**GET, POST, PUT, PATCH, DELETE**). |
| **Killer Feature**| **Candidate Datastore (The Scratchpad):** Allows you to send a config, validate it (`<validate>`), and only then apply it (`<commit>`) or roll it back if there's an error. | **No Scratchpad:** Changes made via RESTCONF are immediately injected directly into the Running Datastore. |

---

### 🚀 gNMI & Streaming Telemetry

**What is gNMI? (gRPC Network Management Interface)**
Since the industry already had a super-fast courier protocol (**gRPC**, developed by Google) and universal configuration templates (**OpenConfig/YANG**), they needed to combine them. gNMI is simply a network management protocol built specifically on top of the gRPC framework.

**Why is gNMI so important? (Exam Hook - Telemetry!)**
The greatest strength of gNMI is *not* configuring routers. Its absolute superpower is **Streaming Telemetry**.

*   **The Old Way (SNMP Polling):** The server asks the router every 5 minutes: *"What is your CPU usage?"*. The router replies. This is slow, resource-heavy, and leaves massive 5-minute blind spots.
*   **The New Way (gNMI Streaming Telemetry):** You use gNMI to tell the router: *"Open a fast gRPC tunnel and continuously spit out your stats every single second in a lightweight binary format."* 

The router pushes the data proactively. This is the absolute future of network monitoring!