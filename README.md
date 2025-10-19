### High Availability Configuration with HSRP

#### **Objective**
To implement a first-hop gateway redundancy protocol (FHRP) solution using Cisco's Hot Standby Router Protocol (HSRP). The goal is to ensure that hosts on a subnet can maintain connectivity to external networks even if their default gateway router fails, thus eliminating a single point of failure (SPOF).

#### **Network Topology**
![GNS3 Network Topology](img/HSRP.png)

The topology consists of two Local Area Networks (LANs) and two routers (R1 and R2) that provide mutual redundancy:
*   **Network A:** `192.168.100.0/24`
*   **Network B:** `192.168.200.0/24`
*   **Routers:** R1 and R2 are connected to both networks.
*   **Virtual Gateway (HSRP):**
    *   Network A: `192.168.100.100` (HSRP Group 100)
    *   Network B: `192.168.200.100` (HSRP Group 200)

#### **Key Concepts Implemented**
1.  **Virtual Gateway:** A shared virtual IP and virtual MAC address were configured for R1 and R2. The client PCs use this virtual IP (`.100`) as their gateway, abstracting the presence of two physical routers.

2.  **Active/Standby Election:** HSRP elects one `Active` and one `Standby` router.
    *   In this configuration, both routers have the default priority (100).
    *   The tie-breaker is the **highest physical IP address** on the HSRP-enabled interface.
    *   Since `192.168.100.254` (R1) > `192.168.100.253` (R2), **R1 is elected as the Active router** for both networks, while R2 assumes the `Standby` role.

3.  **Preemption:** The `standby preempt` command was enabled. This ensures that if router R1 (which has the higher tie-breaking priority due to its IP) comes online after R2, it will "reclaim" its Active role, forcing a failover and keeping the network behavior deterministic.

4.  **Interface Tracking:** This is a key component for intelligent failover. HSRP is configured not just to monitor the local interface, but also the health of the "uplink" or "external" connections.
    *   **On Network A (Group 100):** R1's `e0/0` interface tracks the state of its other interface, `e0/1`. If `e0/1` fails, R1's HSRP priority on Network A is decremented by 60 (dropping from 100 to 40).
    *   **Intelligent Failover:** With a priority of 40, R1 is now lower than R2 (which remains at 100). R2, as the Standby router with a now-higher priority, immediately takes over the Active role, redirecting Network A's traffic through its own functional path. The same logic is applied reciprocally for Network B.
  
#### **Configuration Files**
The full running configurations for each device can be found in the `config` directory:
[`config\R1`](config/R1-Config.txt)
[`config\R2`](config/R2-Config.txt)
