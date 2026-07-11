# 📡 SNMP: Architecture, Versions & MIBs

There are 3 main versions of the SNMP protocol:

*   **SNMPv1 (`noAuthNoPriv`)** -> Obsolete. It is practically never used today. It provides no authentication and no encryption. The *community string* acts as a sort of password, but we do not consider it true authentication.
*   **SNMPv2c (`noAuthNoPriv`)** -> Similar to version 1, but slightly improved. It includes the ability to fetch data in bulk (via `GetBulk`) and features better error handling.
*   **SNMPv3:**
    *   `noAuthNoPriv`: No authentication, no encryption.
    *   `authNoPriv`: Authentication via HMAC-MD5 or HMAC-SHA algorithms, but no encryption.
    *   `authPriv`: Full authentication and encryption.

---

### 🏛️ The 3 Pillars of SNMP Architecture

<div align="center">
  <a href="IMAGES/snmp.png" target="_blank">
    <img src="IMAGES/snmp.png" style="max-width: none; width: 400px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

<div align="center">
  <a href="IMAGES/snmp-1.png" target="_blank">
    <img src="IMAGES/snmp-1.png" style="max-width: none; width: 400px;" title="Kliknij, aby otworzyć w pełnym rozmiarze">
  </a>
</div>

The SNMP architecture consists of three main elements:

1.  **SNMP Manager:** The central monitoring server (e.g., PRTG or Zabbix) that collects and displays the data.
2.  **SNMP Agent:** The software module activated on the network devices (routers, switches, servers, or firewalls).
3.  **MIB (Management Information Base):** The virtual database that acts as a shared dictionary between the Manager and the Agent.

#### 🗂️ Deep Dive: MIB and OID

The MIB is the entire database (structured like a tree). Inside the MIB, we have **OIDs (Object Identifiers)**. 
An OID is a long string of numbers (e.g., `1.3.6.1.4.1.9...`) that serves as an exact address for a specific piece of information inside the MIB. The SNMP Manager sends an OID to ask for a specific metric.

**The MIB on the Agent (e.g., on a switch):**
This is a "live" database built into the operating system. Physical values are recorded here in real-time. The agent knows that under a specific OID number (e.g., `1.3.6.1.2.1.2.2.1.8`), it holds the current status of the `GigabitEthernet0/1` port.

**The MIB on the SNMP Manager (e.g., PRTG / Zabbix):**
Here, MIBs are simply text files (templates / dictionaries) that you upload to the system. 
*Why does the Manager need this file?* When the SNMP Manager queries the router, the router only sends back raw numbers: *"The value for 1.3.6.1.2.1.2.2.1.8 is 1"*. The Manager is dumb. It doesn't know what that means. So, it looks into the uploaded MIB file (the dictionary) and translates it into human language: *"Aha! This long string of digits means 'ifOperStatus', and the value '1' means 'Up'. So the port is working!"*.

---

### 📥 PULL vs 📤 PUSH (Crucial Concept)

Remember the mechanics of how data moves:
*   **PULL:** Achieved via **`GET`** requests.
*   **PUSH:** Achieved via **`TRAP`** messages.

> **💡 The Engineering Conclusion:** 
> SNMP polling (querying via `GET`) is performed periodically (e.g., every 5 minutes). Sometimes it is much better to configure a `TRAP` so that the device immediately triggers sending information to the server the exact moment an important event occurs (e.g., a port going down).