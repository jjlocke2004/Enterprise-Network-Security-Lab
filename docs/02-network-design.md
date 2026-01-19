# Network Design

This document describes the IP addressing plan, adapter design, and a high-level diagram of the lab network.

## IP Addressing Plan

- Network: 192.168.10.0/24
- Subnet mask: 255.255.255.0
- Default gateway: 192.168.10.1 (virtual router / VMware NAT)
- DNS server: 192.168.10.10 (DC01)

### Reserved Ranges

- 192.168.10.1 – Gateway
- 192.168.10.10–192.168.10.19 – Infrastructure (DC01, file server, core Ubuntu services)
- 192.168.10.20–192.168.10.99 – Servers and tools
- 192.168.10.100–192.168.10.200 – Clients and test systems

This layout uses a single /24 network for the lab, which is common for small homelab or small office environments and leaves room for growth. [web:124][web:125]

## Adapter Design

Each virtual machine in the lab is connected to a single lab LAN network to keep routing simple and predictable.

- Each VM has **one active NIC** attached to the same VMware VMnet that represents the 192.168.10.0/24 lab LAN.
- Any extra NICs that picked up addresses on other networks (for example, 192.168.159.0/24 or 192.168.61.0/24) are disabled to avoid routing confusion and overlapping paths.

Initially, extra adapters obtained DHCP addresses on 192.168.159.0/24 and 192.168.61.0/24. I disabled these interfaces and standardized all lab traffic on a single 192.168.10.0/24 network to simplify routing and match a small office LAN.

## Network Diagram

The lab network is represented by a simple hub-and-spoke design centered on the virtual router and lab switch.

- The virtual router/NAT device provides the default gateway at 192.168.10.1 for all lab systems.
- A virtual or physical switch connects core infrastructure (DC01, file server, Ubuntu services) and any additional lab VMs and clients in the 192.168.10.0/24 network.

The diagram is stored as an image in the `diagrams` folder

