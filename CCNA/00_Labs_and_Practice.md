# CCNA Labs and Practice Tasks

This document outlines practice labs and tasks to reinforce your CCNA knowledge.

## How to Use This Guide

1. Complete these labs in order of increasing complexity
2. Use Packet Tracer, GNS3, EVE-NG, or CML for virtual labs
3. Document your configurations and observations in separate lab notes
4. Link your lab notes back to relevant theory notes

## Basic Labs

### Lab 1: Basic Device Configuration

**Objective:** Configure basic settings on a Cisco router and switch

- [ ] Configure hostnames, passwords, and banners
- [ ] Configure console, VTY, and enable passwords
- [ ] Set up IP addressing on router interfaces
- [ ] Save configurations
- [ ] Verify configurations with show commands
- [ ] Practice password recovery

### Lab 2: Basic Switching

**Objective:** Configure and verify basic switching operations

- [ ] Create VLANs and assign ports
- [ ] Configure trunk links between switches
- [ ] Verify VLAN configuration and port assignments
- [ ] Configure port security
- [ ] Test connectivity between devices in the same and different VLANs

### Lab 3: Static Routing

**Objective:** Configure and verify static routes

- [ ] Set up a three-router topology
- [ ] Configure IP addressing
- [ ] Configure static routes between networks
- [ ] Test end-to-end connectivity
- [ ] Add a default route
- [ ] Explore floating static routes

## Intermediate Labs

### Lab 4: VLANs and Inter-VLAN Routing

**Objective:** Configure inter-VLAN routing using router-on-a-stick

- [ ] Configure multiple VLANs on switches
- [ ] Configure subinterfaces on router
- [ ] Configure inter-VLAN routing
- [ ] Test connectivity between VLANs
- [ ] Troubleshoot inter-VLAN routing issues

### Lab 5: OSPF Routing

**Objective:** Configure and verify single-area OSPF

- [ ] Set up multi-router topology
- [ ] Configure OSPF on all routers
- [ ] Verify OSPF adjacencies and database
- [ ] Test routing and failover
- [ ] Configure OSPF authentication

### Lab 6: ACLs and NAT

**Objective:** Configure and verify ACLs and NAT

- [ ] Create standard and extended ACLs
- [ ] Apply ACLs to interfaces
- [ ] Verify ACL operation
- [ ] Configure static NAT
- [ ] Configure dynamic NAT and PAT
- [ ] Verify NAT operations

## Advanced Labs

### Lab 7: EtherChannel and Spanning Tree

**Objective:** Configure and verify EtherChannel and STP

- [ ] Configure LACP EtherChannel between switches
- [ ] Verify EtherChannel operation
- [ ] Configure Rapid PVST+
- [ ] Set root bridge and priority
- [ ] Configure PortFast and BPDU Guard
- [ ] Test STP failover and convergence

### Lab 8: DHCP and DNS

**Objective:** Configure and verify DHCP and DNS services

- [ ] Configure DHCP server on Cisco router
- [ ] Configure DHCP client on devices
- [ ] Configure DHCP relay
- [ ] Set up basic DNS services
- [ ] Verify DHCP and DNS operation

### Lab 9: Wireless Configuration

**Objective:** Configure basic wireless network

- [ ] Set up a WLC (or AP in autonomous mode)
- [ ] Configure WLAN with WPA2 security
- [ ] Configure client connectivity
- [ ] Verify wireless connectivity
- [ ] Configure guest wireless access

### Lab 10: Network Services and Security

**Objective:** Configure various network services and security features

- [ ] Configure SSH for remote access
- [ ] Configure NTP client/server
- [ ] Set up Syslog
- [ ] Configure SNMP
- [ ] Implement AAA with local authentication
- [ ] Configure port security and DHCP snooping

## Comprehensive Practice Scenarios

### Scenario 1: Small Business Network

**Objective:** Design and implement a complete small business network

- [ ] Design appropriate addressing scheme
- [ ] Configure routing and switching infrastructure
- [ ] Implement appropriate security measures
- [ ] Configure network services
- [ ] Test and troubleshoot end-to-end connectivity

### Scenario 2: Branch Office Connectivity

**Objective:** Set up a branch office with connectivity to HQ

- [ ] Configure branch routers and switches
- [ ] Set up WAN connectivity
- [ ] Implement routing between sites
- [ ] Configure security between sites
- [ ] Set up centralized services

## Troubleshooting Exercises

### Exercise 1: VLAN and Trunking Issues

- [ ] Identify and fix misconfigured VLANs
- [ ] Troubleshoot trunk links
- [ ] Resolve native VLAN mismatches
- [ ] Fix incorrect port assignments

### Exercise 2: Routing Problems

- [ ] Diagnose and fix routing protocol issues
- [ ] Resolve incorrect routing tables
- [ ] Fix packet forwarding problems
- [ ] Troubleshoot adjacency issues

### Exercise 3: Access Control Issues

- [ ] Troubleshoot ACL problems
- [ ] Identify incorrect NAT configurations
- [ ] Fix broken remote access
- [ ] Resolve security policy conflicts

## Additional Resources

- [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer)
- [GNS3](https://www.gns3.com/)
- [EVE-NG](https://www.eve-ng.net/)
- [Cisco DevNet](https://developer.cisco.com/)

## Lab Documentation Template

```markdown
# Lab Title
Date: [Date]
Duration: [Time taken]

## Objective
[Brief description of what you aimed to accomplish]

## Topology
[Diagram or description of the network setup]

## Equipment/Software Used
[List of devices or software used]

## Configuration Steps
1. [Step 1]
2. [Step 2]
3. [etc.]

## Verification
[Commands used to verify configuration and their output]

## Observations and Challenges
[What you learned, any issues encountered and how you resolved them]

## References
[Any reference materials used]
```
