Inter-VLAN Routing with Router-on-a-Stick

Overview

This was a mini lab I built to bring together the networking concepts I've been learning while preparing for the CCNA.

The goal was to configure three departments within a faculty, place their devices in separate VLANs, and enable communication between them using Router-on-a-Stick.

Rather than studying VLANs, trunking, subnetting, and router subinterfaces as separate topics, I wanted to see how they work together in a functioning network.

I built the lab using PingMyNetwork Sandbox, a browser-based networking environment that allowed me to design, configure, and test the network without downloading software or setting up a virtual machine.

A big shout-out to the PingMyNetwork team for providing an opportunity to practise networking this way. Having a browser-based environment made it easier to turn what I had been learning into something practical.

Lab Objectives

- Create separate VLANs for three departments.
- Subnet an IPv4 network into three "/26" networks.
- Configure access ports and assign them to the appropriate VLANs.
- Configure a trunk link to carry traffic for multiple VLANs.
- Implement Router-on-a-Stick using 802.1Q subinterfaces.
- Configure default gateways for the three subnets.
- Verify communication between devices in different VLANs.

Network Topology

The network contains eight PCs distributed across three departments.

Department| Number of PCs| VLAN
Cybersecurity| 2| 10
Software Engineering| 3| 20
Data Science| 3| 30
Total| 8| 

Each department has its own VLAN and IPv4 subnet.

The PCs connect to a switch, which connects to a router through a trunk link. The router uses three logical subinterfaces to route traffic between the VLANs.

Logical Topology

 Cybersecurity       Software Engineering       Data Science
     2 PCs                  3 PCs                  3 PCs
       |                      |                      |
    VLAN 10                VLAN 20                VLAN 30
       |                      |                      |
       +----------------------+----------------------+
                              |
                         Access Switch
                              |
                         802.1Q Trunk
                              |
                            Router
                              |
                   Router-on-a-Stick
                    /        |        \
                G0/0.10   G0/0.20   G0/0.30
                   |         |         |
                VLAN 10   VLAN 20   VLAN 30

Note: This is a simplified logical representation of the lab.

IP Addressing and VLAN Plan

I used three "/26" subnets from the "192.168.10.0/24" network.

A "/26" subnet has a subnet mask of "255.255.255.192" and provides 62 usable IPv4 addresses.

Department| VLAN ID| Network| Subnet Mask| Default Gateway| Broadcast Address
Cybersecurity| 10| "192.168.10.0/26"| "255.255.255.192"| "192.168.10.62"| "192.168.10.63"
Software Engineering| 20| "192.168.10.64/26"| "255.255.255.192"| "192.168.10.126"| "192.168.10.127"
Data Science| 30| "192.168.10.128/26"| "255.255.255.192"| "192.168.10.190"| "192.168.10.191"

Gateway Addressing Decision

One detail worth documenting is my choice of default gateway addresses.

Instead of assigning the first usable address in each subnet to the router, I assigned the last usable address.

VLAN| First Usable Address| Last Usable Address| Router Gateway
VLAN 10| "192.168.10.1"| "192.168.10.62"| "192.168.10.62"
VLAN 20| "192.168.10.65"| "192.168.10.126"| "192.168.10.126"
VLAN 30| "192.168.10.129"| "192.168.10.190"| "192.168.10.190"

This reinforced my understanding of subnet boundaries, usable host ranges, and how gateway addresses fit into an IP addressing plan.

Configuration and Implementation

1. VLAN Configuration

I created three VLANs on the switch:

- VLAN 10 — Cybersecurity
- VLAN 20 — Software Engineering
- VLAN 30 — Data Science

The switch ports connected to the PCs were assigned to their respective VLANs.

Each VLAN represents a separate Layer 2 broadcast domain.

2. Trunk Configuration

The switch-to-router connection was configured as a trunk.

A trunk allows traffic from multiple VLANs to travel over a single physical link. In this lab, IEEE 802.1Q tagging identifies the VLAN associated with each Ethernet frame.

Without the trunk, the physical connection would not carry traffic for all three VLANs in the intended configuration.

3. Router-on-a-Stick Configuration

I used Router-on-a-Stick to enable communication between the three VLANs.

Instead of requiring a separate physical router interface for each VLAN, this approach uses one physical router interface with multiple logical subinterfaces.

Each subinterface is associated with a VLAN through 802.1Q encapsulation and configured with an IP address belonging to that VLAN's subnet.

Router Subinterface| VLAN| IP Address| Purpose
"G0/0.10"| 10| "192.168.10.62/26"| Cybersecurity default gateway
"G0/0.20"| 20| "192.168.10.126/26"| Software Engineering default gateway
"G0/0.30"| 30| "192.168.10.190/26"| Data Science default gateway

The router uses these subinterfaces to route traffic between the separate IP networks.

4. How Inter-VLAN Routing Works

Consider a PC in the Cybersecurity VLAN communicating with a PC in the Software Engineering VLAN.

1. The source PC determines that the destination IP address belongs to a different subnet.
2. It forwards the traffic to its default gateway, "192.168.10.62".
3. The switch forwards the frame through the trunk toward the router.
4. The router processes the packet and determines that the destination network is reachable through the Software Engineering subinterface.
5. The router forwards the packet through the appropriate subinterface, using the VLAN 20 tagging on the trunk.
6. The switch forwards the traffic to the destination PC's access port in VLAN 20.

The same principle applies when devices communicate between any of the three VLANs.

The important distinction is that the VLANs remain separate Layer 2 broadcast domains while the router enables communication between their IP subnets.

Verification and Results

After completing the configuration, I tested connectivity using ping.

Result: All inter-VLAN pings worked successfully.

The successful tests confirmed that devices in different VLANs could communicate through the router.

This was an important part of the lab because it allowed me to verify the result rather than assuming the configuration was correct simply because the commands had been accepted.

Verification Checklist

- [x] Created three department VLANs.
- [x] Assigned access ports to their respective VLANs.
- [x] Configured the switch-to-router trunk.
- [x] Configured router subinterfaces with 802.1Q encapsulation.
- [x] Assigned the last usable IP address in each subnet to the router subinterface.
- [x] Configured the three default gateways.
- [x] Successfully tested communication between devices in different VLANs.

Troubleshooting Commands

These Cisco IOS commands are useful when verifying or troubleshooting a similar configuration.

Command| Purpose
"show vlan brief"| Verify VLANs and access-port assignments
"show interfaces trunk"| Check trunk status and VLANs carried
"show ip interface brief"| Check interface addresses and operational status
"show running-config"| Inspect the active configuration
"show ip route"| Verify connected networks and routing information

If connectivity fails, I would check VLAN membership, trunk operation, 802.1Q VLAN IDs, interface status, IP addressing, subnet masks, and default gateways.

Key Takeaways

This mini lab helped me connect several networking concepts I had been studying individually.

VLANs separate devices into different Layer 2 broadcast domains.

Trunking allows multiple VLANs to share a physical connection.

802.1Q identifies VLAN membership through Ethernet frame tagging.

Router-on-a-Stick allows one physical router interface to provide Layer 3 connectivity for multiple VLANs through subinterfaces.

Subnetting determines which IP addresses belong to each department's network and which address can serve as its default gateway.

Most importantly, I was able to put these concepts together, configure the network, and verify that communication between all three VLANs worked.

Tools and Technologies

- PingMyNetwork Sandbox
- Cisco IOS-style CLI
- VLANs
- IEEE 802.1Q
- Router-on-a-Stick
- IPv4 subnetting
- "/26" subnet masks
- Ping-based connectivity testing

Acknowledgements

A big thank you to PingMyNetwork for making the Sandbox available.

Being able to build and configure this network directly in my browser, without downloading software or setting up a virtual machine, gave me an opportunity to practise what I have been learning.

I appreciate the opportunity to learn by building, testing, and verifying.

---

This project is part of my ongoing practical networking labs as I study for the CCNA.