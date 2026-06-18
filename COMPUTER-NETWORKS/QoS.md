# 🚦 QoS (Quality of Service): The Quick Cheat Sheet

Here is a quick summary of the core QoS mechanisms used to manage and prioritize network traffic:

*   **CBWFQ (Class-Based Weighted Fair Queuing) - Bandwidth Guarantee:** 
    If the router recognizes critical traffic (e.g., Voice), it guarantees it a specific amount of bandwidth. This ensures that users watching YouTube won't choke the link and degrade the quality of phone calls.
*   **Policing - Hard Limits:** 
    Simply enforces strict, hard limits on bandwidth. If traffic exceeds the configured rate, the router ruthlessly drops it (or remarks it).
*   **Marking (DSCP Coloring):** 
    Instead of forcing every subsequent router in the network to deeply analyze packets (which eats CPU), you do it only *once* on the first ingress switch. NBAR recognizes the application, and you "color" the packet by assigning a specific DSCP value (e.g., `EF` for Voice). Every subsequent router, including your ISP, just looks at this tag and instantly knows how to treat the packet.
*   **WRED (Weighted Random Early Detection) - Congestion Avoidance:** 
    Prevents network congestion by proactively dropping the least important traffic *before* the queues get completely full.