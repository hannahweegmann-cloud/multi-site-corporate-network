# Multi-Site Corporate Network

Cisco Packet Tracer project connecting a segmented headquarters network and branch office through a simulated ISP. The design uses dynamic routing, centralized DHCP, access controls, address translation, and redundant switching.

![Network design]

## Project goals

- Segment HQ users, administrators, and servers with VLANs.
- Connect HQ and the branch through a routed ISP network.
- Assign client addresses through DHCP.
- Exchange routes dynamically with OSPF.
- Protect the HQ Admin network with an extended ACL.
- Demonstrate PAT for HQ internet-bound traffic.
- Provide redundant HQ switch links with LACP EtherChannel and STP.
- Validate normal traffic, denied traffic, and link-failure recovery.

## Topology

The network contains three Cisco 2911 routers, four Cisco 2960 switches, six PCs, one internal server, and one simulated public server.

### Packet Tracer implementation

![Packet Tracer topology]
| Zone | Devices | Purpose |
| --- | --- | --- |
| HQ | HQ-R1, HQ-SW1, HQ-SW2, four PCs, internal server | Department segmentation, inter-VLAN routing, PAT, redundant switching |
| ISP/Public | ISP, ISP-SW, public server | WAN transit and simulated public services |
| Branch | BRANCH-R1, BR-SW1, two PCs | Routed branch access with policy enforcement |

## Addressing plan

| Network | VLAN | Subnet | Gateway or router address | Use |
| --- | ---: | --- | --- | --- |
| HQ Admin | 10 | `192.168.10.0/24` | `192.168.10.1` | Administrative endpoints |
| HQ Users | 20 | `192.168.20.0/24` | `192.168.20.1` | General users |
| HQ Servers | 30 | `192.168.30.0/24` | `192.168.30.1` | Internal server segment |
| Branch Users | 40 | `192.168.40.0/24` | `192.168.40.1` | Branch endpoints |
| HQ to ISP | - | `10.0.0.0/30` | HQ `10.0.0.1`, ISP `10.0.0.2` | WAN transit |
| ISP to Branch | - | `10.0.0.4/30` | ISP `10.0.0.5`, Branch `10.0.0.6` | WAN transit |
| Public server | - | `203.0.113.0/24` | ISP `203.0.113.1` | Simulated internet network |

Static servers:

- HQ internal server: `192.168.30.10/24`
- Public server: `203.0.113.10/24`

DHCP excludes addresses `.1` through `.20` in each client subnet. Client leases begin at approximately `.21`.

## Key configurations

### VLANs and inter-VLAN routing

HQ uses 802.1Q router-on-a-stick on HQ-R1. Subinterfaces `G0/1.10`, `G0/1.20`, and `G0/1.30` provide the default gateways for VLANs 10, 20, and 30. The branch uses a single access VLAN on `G0/1` because only one branch user segment is required.

### Dynamic routing

All three routers participate in OSPF area 0. LAN-facing interfaces are passive so the routers advertise those networks without attempting unnecessary neighbor adjacencies with endpoints.

### Security controls

- An extended ACL on BRANCH-R1 blocks VLAN 40 from reaching the HQ Admin network (`192.168.10.0/24`).
- The ACL permits branch users to reach the HQ server and simulated public server.
- HQ-R1 applies PAT to internet-bound traffic from the three HQ VLANs.
- HQ-to-branch traffic is exempted from PAT to preserve private source addressing and return routing.

### Availability

HQ-SW1 and HQ-SW2 use LACP EtherChannel `Po1`. Two physical FastEthernet links operate as one logical trunk. STP treats the bundle as a single logical path. A shutdown test on one member link confirmed that traffic continued across the remaining member.

## Validation results

| Test | Expected result | Result |
| --- | --- | --- |
| HQ client to local gateway | Allowed | Passed |
| HQ user VLAN to HQ server VLAN | Allowed through router-on-a-stick | Passed |
| Branch client to HQ server | Allowed through OSPF | Passed |
| Branch client to HQ Admin gateway | Denied by extended ACL | Passed |
| HQ client to public server | Allowed through PAT | Passed |
| Branch client to public server | Allowed through OSPF | Passed |
| HQ EtherChannel with one member shut down | Connectivity remains available | Passed |

## Troubleshooting performed

- Used `show interfaces status` and CDP to correct physical-to-logical switch-port mappings.
- Corrected access and trunk assignments after DHCP requests failed.
- Simplified the single-VLAN branch uplink from a tagged subinterface to a routed access link.
- Verified OSPF neighbor states and learned routes before end-to-end tests.
- Corrected NAT inside/outside interface roles.
- Removed branch PAT after identifying that it translated site-to-site traffic and disrupted return traffic.
- Verified ACL match counters to distinguish policy enforcement from routing failures.
- Tested EtherChannel convergence by shutting down and restoring one member interface.

## Repository contents

```text
multi-site-corporate-network/
├── README.md
├── Multi-Site-Corporate-Network-Final.pkt
├── assets/
│   ├── network-diagram.svg
│   └── packet-tracer-topology.png
└── configs/
    ├── HQ-R1.txt
    ├── ISP.txt
    ├── BRANCH-R1.txt
    ├── HQ-SW1.txt
    ├── HQ-SW2.txt
    └── BR-SW1.txt
```

## Open the project

1. Install Cisco Packet Tracer 9.0 or a compatible version.
2. Download `Multi-Site-Corporate-Network-Final.pkt`.
3. Open the file in Packet Tracer.
4. Use the CLI on each router or switch to review the configuration and verification commands.

## Skills demonstrated

Cisco IOS, IPv4 subnetting, VLANs, 802.1Q, router-on-a-stick, DHCP, OSPF, NAT/PAT, extended ACLs, LACP EtherChannel, STP, connectivity testing, and structured troubleshooting.
