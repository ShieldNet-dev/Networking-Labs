# Inter-VLAN Routing with Router-on-a-Stick

![CCNA Practice Lab](https://img.shields.io/badge/CCNA%20Practice%20Lab-PingMyNetwork%20Sandbox-0969DA?style=for-the-badge&labelColor=1F2328)

A practical networking lab combining VLAN configuration, IPv4 subnetting, 802.1Q trunking, and inter-VLAN routing.

---

## 1. Overview

This mini lab brings together key networking concepts from my CCNA studies. I configured three departmental VLANs and subnets, then implemented **Router-on-a-Stick** to enable inter-VLAN communication. Using the **PingMyNetwork Sandbox**, I built and tested the network directly in my browser without installing software or setting up a virtual machine.

A big thank-you to the PingMyNetwork team for making hands-on networking practice accessible.

## 2. Lab Objectives

- Create three departmental VLANs and subnet an IPv4 network using `/26` prefixes.
- Assign switch access ports to the appropriate VLANs.
- Configure an 802.1Q trunk between the switch and router.
- Implement Router-on-a-Stick using router subinterfaces.
- Configure default gateways for each subnet.
- Verify end-to-end connectivity between devices across VLANs.

## 3. Network Topology

The lab consists of **eight PCs distributed across three departments**, each assigned to a dedicated VLAN. All PCs connect to an access switch, which connects to a router through a single **802.1Q trunk link**.

The router uses the **Router-on-a-Stick** configuration to enable communication between devices in different VLANs.

### Department and VLAN Summary

| Department              | Number of PCs | VLAN ID |
|:------------------------|:-------------:|:-------:|
| Cybersecurity           | 2             | 10      |
| Software Engineering    | 3             | 20      |
| Data Science            | 3             | 30      |
| **Total**               | **8**         | **3 VLANs** |

### Logical Topology

```text
       CYBERSECURITY       SOFTWARE ENGINEERING       DATA SCIENCE
           2 PCs                  3 PCs                   3 PCs
             |                      |                       |
          VLAN 10                VLAN 20                 VLAN 30
             \                      |                      /
              \_____________________|_____________________/
                                    |
                             ┌───────────────┐
                             │ ACCESS SWITCH │
                             └───────┬───────┘
                                     |
                              802.1Q TRUNK
                                     |
                             ┌───────────────┐
                             │    ROUTER     │
                             └───────┬───────┘
                                     |
                          ROUTER SUBINTERFACES
                              /      |      \
                             /       |       \
                        G0/0.10  G0/0.20  G0/0.30
                           |        |        |
                        VLAN 10  VLAN 20  VLAN 30
```

---

## 4. IP Addressing Plan

I divided the `192.168.10.0/24` network into three `/26` subnets, assigning one subnet to each department.

A `/26` subnet uses the subnet mask `255.255.255.192` and provides **64 total IPv4 addresses, including 62 usable host addresses**.

### Subnet Allocation

| Department              | VLAN ID | Network Address          | Subnet Mask         | Default Gateway      | Broadcast Address    |
|:------------------------|:-------:|:-------------------------|:--------------------|:---------------------|:---------------------|
| Cybersecurity           | 10      | `192.168.10.0/26`        | `255.255.255.192`   | `192.168.10.62`      | `192.168.10.63`      |
| Software Engineering    | 20      | `192.168.10.64/26`       | `255.255.255.192`   | `192.168.10.126`     | `192.168.10.127`     |
| Data Science            | 30      | `192.168.10.128/26`      | `255.255.255.192`   | `192.168.10.190`     | `192.168.10.191`     |

### Default Gateway Selection

For this lab, I assigned the **last usable IP address in each subnet** to the corresponding router subinterface rather than the first usable address.

| VLAN    | First Usable Address | Last Usable Address / Gateway |
|:-------:|:---------------------|:------------------------------|
| VLAN 10 | `192.168.10.1`       | `192.168.10.62`               |
| VLAN 20 | `192.168.10.65`      | `192.168.10.126`              |
| VLAN 30 | `192.168.10.129`     | `192.168.10.190`              |


## 5. Implementation

### 5.1 VLAN Configuration

I created three VLANs on the switch and assigned the PC-facing access ports to their respective departments.

| VLAN ID | Department            |
|:-------:|:----------------------|
| 10      | Cybersecurity         |
| 20      | Software Engineering  |
| 30      | Data Science          |

Each VLAN represents a separate Layer 2 broadcast domain.

### 5.2 Trunk Configuration

I configured the switch-to-router connection as a trunk.

A trunk allows traffic from multiple VLANs to travel across a single physical link. In this lab, IEEE 802.1Q tagging identifies the VLAN associated with Ethernet frames travelling over the trunk.

### 5.3 Router-on-a-Stick Configuration

I used Router-on-a-Stick to route traffic between the three VLANs.

Instead of requiring a separate physical router interface for each VLAN, this approach uses one physical interface with multiple logical subinterfaces.

Each subinterface is associated with a VLAN through 802.1Q encapsulation and configured with an IP address from that VLAN's subnet.

| Router Subinterface | VLAN ID | IP Address              | Purpose                        |
|:--------------------|:-------:|:------------------------|:-------------------------------|
| `G0/0.10`           | 10      | `192.168.10.62/26`      | Cybersecurity gateway          |
| `G0/0.20`           | 20      | `192.168.10.126/26`     | Software Engineering gateway   |
| `G0/0.30`           | 30      | `192.168.10.190/26`     | Data Science gateway           |

---

## 6. How Inter-VLAN Routing Works

Consider a PC in the Cybersecurity VLAN communicating with a PC in the Software Engineering VLAN.

1. The source PC determines that the destination IP address belongs to a different subnet.
2. It sends the traffic to its default gateway, `192.168.10.62`.
3. The switch forwards the frame across the trunk toward the router, with VLAN 10 identified through 802.1Q tagging.
4. The router examines the destination IP address and routes the packet toward the Software Engineering subnet.
5. The router forwards the traffic through subinterface `G0/0.20`, with the outgoing frame tagged for VLAN 20.
6. The switch forwards the frame to the destination PC's access port in VLAN 20.

The same principle applies when devices communicate between any of the three VLANs.

> **Key concept:** The VLANs remain separate Layer 2 broadcast domains, while the router provides Layer 3 connectivity between their IP networks.

## 7. Verification and Results

After configuring the VLANs, trunk, and router subinterfaces, I tested connectivity between devices in different VLANs using `ping`.

**Result:** Successful

All inter-VLAN pings worked successfully.

This confirmed that devices in the three separate subnets could communicate through the router.

### Verification Checklist

- [x] Created VLANs 10, 20, and 30.
- [x] Assigned access ports to their respective VLANs.
- [x] Configured the switch-to-router trunk.
- [x] Configured router subinterfaces with 802.1Q encapsulation.
- [x] Assigned the last usable IP address in each subnet to the router gateway.
- [x] Configured the appropriate IP addressing and default gateways.
- [x] Successfully tested connectivity between devices in different VLANs.

---

## 8. Key Takeaways

This mini lab helped me connect several networking concepts I had been studying individually.

- **VLANs:** Separate devices into different Layer 2 broadcast domains.
- **Trunking:** Allows multiple VLANs to share a physical link.
- **802.1Q:** Identifies VLANs through Ethernet frame tagging.
- **Router-on-a-Stick:** Uses router subinterfaces to route traffic between VLAN subnets.
- **IPv4 subnetting:** Defines network boundaries, usable host ranges, and default gateway addresses.

Most importantly, I was able to bring these concepts together, configure the network, and verify that communication between all three VLANs worked.

---

Part of my ongoing hands-on networking labs while preparing for the CCNA.