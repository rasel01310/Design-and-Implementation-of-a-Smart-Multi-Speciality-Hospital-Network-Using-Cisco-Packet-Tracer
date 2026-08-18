# Design-and-Implementation-of-a-Smart-Multi-Speciality-Hospital-Network-Using-Cisco-Packet-Tracer
## 📌 Project Overview

The **Smart Multi-Speciality Hospital Network** is a Cisco Packet Tracer-based
network design and implementation project developed for the **CSE322:
Computer Networks Lab** course in the Department of Computer Science and
Engineering at **Daffodil International University**.

The project provides a structured, scalable, and efficient network
infrastructure for a multi-speciality hospital environment, connecting
**six major hospital sites**, each containing multiple departments with
dedicated network segments.

The network follows a hierarchical architecture consisting of **ISP Router,
ISP Switch, Edge Routers, Core Routers, Access Switches, VLANs, and End Devices.**

- **VLSM (Variable Length Subnet Masking)** is used for efficient IP address
  allocation based on departmental host requirements.
- **VLANs** separate departments, with **trunking** and **Router-on-a-Stick**
  enabling inter-VLAN communication.
- **DHCP** automatically assigns IP addresses to end devices.
- **EIGRP (AS 100)** provides dynamic routing between internal hospital
  networks.
- **NAT/PAT** on the Edge Routers provides Internet access to private
  internal networks.
- **Network services** — HTTP, FTP, DNS, and Email — are configured and
  tested.
- The complete network is verified using connectivity, routing, DHCP, NAT,
  and service tests.
