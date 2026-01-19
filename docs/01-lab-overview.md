# Lab Overview

This is a virtual Enterprise Network Security Lab constructed inside VMware Workstation to provide a simulated small Windows and Linux environment to practice security.

## Objectives

- Design and draw up a virtual internal network with statical IP addressing.
- Implement fundamental enterprise applications: Active Directory, DNS and file services.
- Load Linux workloads and targets to scan and harden and work on vulnerabilities.
- Reconnaissance and assessment with a specific security tools VM.


## Virtual Machines

| VM Name     | OS                   | Role                                       | IP Address    |
|------------|----------------------|--------------------------------------------|--------------|
| DC01       | Windows Server 2022  | Domain Controller, DNS                     | 192.168.10.10 |
| FS01       | Windows Server 2022  | File Server                                | 192.168.10.11 |
| WIN10      | Windows 10           | Domain-joined client workstation           | 192.168.10.30 |
| UBUNTU-SRV | Ubuntu Server        | Linux services / scan target               | 192.168.10.20 |
| SEC-TOOLS  | Kali Linux           | Security tools and attacker / analyst box  | 192.168.10.50 |

## Network Design

- Network: **192.168.10.0/24**
- Subnet mask: **255.255.255.0**
- Default gateway: **192.168.10.1**
- Primary DNS server: **192.168.10.10 (DC01)**
- All VMs use a single VMware host-only / custom VMnet for the lab LAN.
- Extra virtual NICs and external DHCP networks are disabled to keep routing simple and controlled.

## Initial Build Summary

- Installed windows server on DC 01 and advanced it to a new corp.local forest with inbuilt DNS.
- Set all VMs to have configured fixed IP addresses and directed the DNS of the windows systems to DC01.
- Added FS01 and WIN10 to the domain of the corp.local.
- Ubuntu Server and Kali were installed, and the hosts were allocated static IP addresses and confirmed that all the hosts were connected (ping tests).
  
This initial configuration will serve as a stable base point towards subsequent steps: group policy hardening, file server permissions, vulnerability scanning using the Kali and other tools, log collection and monitoring security.

