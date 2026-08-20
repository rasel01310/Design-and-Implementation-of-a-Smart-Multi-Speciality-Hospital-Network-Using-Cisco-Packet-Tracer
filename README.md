# 🏥 Smart Multi-Speciality Hospital Network

### Design and Implementation of a Multi-Site Hospital Network Infrastructure Using VLSM, EIGRP, NAT, and VLAN Segmentation in Cisco Packet Tracer

[![Platform](https://img.shields.io/badge/Platform-Cisco%20Packet%20Tracer-blue)](https://www.netacad.com/courses/packet-tracer)
[![Routing](https://img.shields.io/badge/Routing-EIGRP-orange)]()
[![Status](https://img.shields.io/badge/Status-Completed-brightgreen)]()
[![Course](https://img.shields.io/badge/Course-CSE322%20Computer%20Networks%20Lab-lightgrey)]()

A fully simulated, six-site hospital network built in Cisco Packet Tracer — connecting a Main Hospital Headquarters with five specialized branches (Diagnostic Center, Emergency Clinic, Telemedicine, Maternity Hospital, and Children Care Hospital) through a hierarchical, VLSM-addressed, EIGRP-routed infrastructure with per-branch NAT/PAT, VLAN-based departmental segmentation, and centralized network services.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Network Architecture](#-network-architecture)
- [Key Features](#-key-features)
- [IP Addressing Scheme](#-ip-addressing-scheme)
- [VLAN Configuration](#-vlan-configuration)
- [Routing — EIGRP](#-routing--eigrp)
- [NAT / PAT Configuration](#-nat--pat-configuration)
- [Network Services](#-network-services)
- [Testing & Validation](#-testing--validation)
- [Tools & Technologies](#-tools--technologies)
- [Repository Structure](#-repository-structure)
- [How to Run This Project](#-how-to-run-this-project)
- [Limitations](#-limitations)
- [Future Work](#-future-work)
- [Team](#-team)
- [Course Information](#-course-information)
- [References](#-references)
- [License](#-license)

---

## 🎯 Project Overview

Modern multi-branch healthcare organizations require reliable, secure, and scalable network infrastructure to connect a main hospital with geographically distributed branches. This project designs and simulates such a network for a fictional **Smart Multi-Speciality Hospital**, consisting of:

- **1 Main Hospital Headquarters**
- **5 Branch Sites**: Diagnostic Center, Emergency Clinic, Telemedicine Branch, Maternity Hospital, Children Care Hospital
- **1 Simulated ISP Router** connecting all sites in a hub-and-spoke topology

Each site hosts multiple departments with distinct connectivity, addressing, and security requirements — solved through VLAN segmentation, hierarchical VLSM addressing, EIGRP dynamic routing, and NAT/PAT for internet access.

---

## 🏗️ Network Architecture

```
                              INTERNET
                                 │
                          R0 (ISP Router)
                    (Gi0/0 sub-interfaces, 200.1.1.0/24)
                                 │
                          ISP Access Switch
                  (802.1Q trunk — 6 VLANs, 1 per branch link)
        ┌──────┬──────┬──────┬──────┬──────┬──────┐
        │      │      │      │      │      │
       R1     R4     R6     R9     R11    R13        ← Edge Routers
     (HQ)  (Diag)  (Emer) (Tele) (Mat)  (Child)         (Public IP + NAT/PAT)
        │      │      │      │      │      │
       R2     R5     R7     R10    R12    R14        ← Core Routers
                                                          (Inter-VLAN routing,
                                                           DHCP, Router-on-a-Stick)
        │      │      │      │      │      │
    Access  Access  Access Access Access Access      ← Layer 2 Switches
    Switch  Switch  Switch Switch Switch Switch
        │      │      │      │      │      │
     VLANs   VLANs  VLANs  VLANs  VLANs  VLANs       ← Department Segments
  (Servers, PCs, IP Phones, Printers, Wireless APs, Medical Devices)
```
---
<img width="970" height="520" alt="Screenshot 2026-08-20 093508" src="https://github.com/user-attachments/assets/5e27d51c-97af-4ed8-a991-f9eac45b77be" />


**Design principle:** Every site follows an identical **Edge Router + Core Router** pattern —
- **Edge Router** → handles the public-facing WAN link and NAT/PAT (security boundary)
- **Core Router** → handles internal VLAN routing via Router-on-a-Stick sub-interfaces and DHCP

This separation keeps the public internet-facing layer isolated from internal routing, and makes every branch trivially reproducible from the same template.

---

## ✨ Key Features

| Feature | Implementation |
|---|---|
| **Hierarchical IP Addressing** | VLSM-based allocation — `/22` for HQ, `/24` per branch, `/26`–`/28` per department |
| **VLAN Segmentation** | Department-level isolation (DMZ, Doctors & ICU, Billing/Admin, Guest Wi-Fi, Lab, Emergency Ward, etc.) |
| **Inter-VLAN Routing** | Router-on-a-Stick with 802.1Q sub-interfaces on every Core Router |
| **Dynamic Routing** | EIGRP (Autonomous System 100), `no auto-summary` for discontiguous network support |
| **Internet Access** | NAT/PAT (interface overload) on every Edge Router, with inter-branch traffic explicitly excluded from translation |
| **Automatic IP Assignment** | DHCP pools configured per VLAN directly on Core Routers |
| **Network Security** | Extended ACLs isolating Guest Wi-Fi from sensitive internal VLANs (ICU, Billing/Admin) |
| **Centralized Services** | DNS, HTTP/Web, FTP, and Email (SMTP/POP3) hosted in the HQ DMZ |
| **Wireless Access** | WPA2-PSK secured access points at HQ and all five branches |

---

## 🌐 IP Addressing Scheme

### Site-Level Allocation

| Site | Router Pair | Network Block | VLAN IDs |
|---|---|---|---|
| Main Hospital HQ | R1 (Edge) / R2 (Core) | `10.0.0.0/22` | 10, 20, 30, 50 |
| Diagnostic Center | R4 (Edge) / R5 (Core) | `10.1.0.0/24` | 100, 110, 120 |
| Emergency Clinic | R6 (Edge) / R7 (Core) | `10.2.0.0/24` | 200, 210, 220 |
| Telemedicine Branch | R9 (Edge) / R10 (Core) | `10.3.0.0/24` | 300, 310, 320 |
| Maternity Hospital | R11 (Edge) / R12 (Core) | `10.4.0.0/24` | 400, 410, 420 |
| Children Care Hospital | R13 (Edge) / R14 (Core) | `10.5.0.0/24` | 500, 510, 520 |

### WAN Links

| Link Type | Address Space | Mask |
|---|---|---|
| ISP ↔ Edge Router | `200.1.1.0/24` | `/30` per link |
| Edge Router ↔ Core Router | `10.255.0.0/24` | `/30` per link |

> Point-to-point WAN links use a uniform `/30` mask since each requires exactly two usable host addresses — VLSM is applied where host requirements vary (department VLANs), not where they're fixed (WAN links).

Full subnet-by-subnet breakdown (network address, broadcast, usable range, gateway) is documented in **`Edge & Core Router IP Configuration.pdf`** and **`VLAN_Config_Report(All Branch).pdf`**.

---

## 🔀 VLAN Configuration

<details>
<summary><b>Click to expand full VLAN table</b></summary>

| Site | VLAN ID | Name | Subnet | Prefix | Gateway | Usable Hosts |
|---|---|---|---|---|---|---|
| HQ | 10 | DMZ | 10.0.1.160 | /28 | 10.0.1.161 | 14 |
| HQ | 20 | DOCTORS_ICU | 10.0.1.64 | /26 | 10.0.1.65 | 62 |
| HQ | 30 | BILLING_ADMIN | 10.0.1.0 | /26 | 10.0.1.1 | 62 |
| HQ | 50 | GUEST_WIFI | 10.0.0.0 | /24 | 10.0.0.1 | 254 |
| Diagnostic Center | 100 | LAB | 10.1.0.0 | /26 | 10.1.0.1 | 62 |
| Diagnostic Center | 110 | ADMIN | 10.1.0.64 | /27 | 10.1.0.65 | 30 |
| Diagnostic Center | 120 | STAFF | 10.1.0.96 | /28 | 10.1.0.97 | 14 |
| Emergency Clinic | 200 | EMERGENCY | 10.2.0.0 | /26 | 10.2.0.1 | 62 |
| Emergency Clinic | 210 | ADMIN | 10.2.0.64 | /27 | 10.2.0.65 | 30 |
| Emergency Clinic | 220 | IP_PHONE | 10.2.0.96 | /28 | 10.2.0.97 | 14 |
| Telemedicine | 300 | CONSULTATION | 10.3.0.0 | /26 | 10.3.0.1 | 62 |
| Telemedicine | 310 | ADMIN | 10.3.0.64 | /27 | 10.3.0.65 | 30 |
| Telemedicine | 320 | WIFI | 10.3.0.128 | /27 | 10.3.0.129 | 30 |
| Maternity Hospital | 400 | MATERNITY | 10.4.0.0 | /26 | 10.4.0.1 | 62 |
| Maternity Hospital | 410 | ADMIN | 10.4.0.64 | /27 | 10.4.0.65 | 30 |
| Maternity Hospital | 420 | WIFI | 10.4.0.128 | /27 | 10.4.0.129 | 30 |
| Children Care | 500 | CHILDREN | 10.5.0.0 | /26 | 10.5.0.1 | 62 |
| Children Care | 510 | ADMIN | 10.5.0.64 | /27 | 10.5.0.65 | 30 |
| Children Care | 520 | WIFI | 10.5.0.128 | /27 | 10.5.0.129 | 30 |

</details>

---

## 🔁 Routing — EIGRP

All routers run **EIGRP Autonomous System 100** with `no auto-summary` enabled to correctly handle the discontiguous `10.x.x.x` address space across branches.

```
router eigrp 100
 no auto-summary
 network <directly-connected-subnet> <wildcard-mask>
```

Verified using:
```
show ip eigrp neighbors
show ip route eigrp
show ip protocols
```

Full per-router EIGRP configuration is in `/configs/eigrp/`.

---

## 🔐 NAT / PAT Configuration

Each branch's **Edge Router** performs PAT (overload) for internet-bound traffic only. An extended ACL explicitly **excludes** inter-branch traffic (destined to other `10.0.0.0/8` subnets) from translation, so branch-to-branch communication is routed natively via EIGRP without unnecessary double-NAT.

```
ip access-list extended NAT_ACL
 deny ip <own-subnet> <wildcard> 10.0.0.0 0.255.255.255
 permit ip <own-subnet> <wildcard> any

ip nat inside source list NAT_ACL interface g0/0 overload
```

> **Design note:** An earlier version of this design NATed *all* `10.0.0.0/8`-bound traffic, which caused intermittent inter-branch communication failures due to double translation. This was identified and fixed by adding the explicit `deny` rule above — see the Discussion section of **`Project Report.pdf`** for the full debugging process.

---

## 🌍 Network Services

Hosted centrally in the HQ DMZ (VLAN 10):

| Service | Purpose |
|---|---|
| DNS | Resolves internal hostnames (e.g., `hospital.com`, `ftp.hospital.com`, `mail.hospital.com`) |
| HTTP / Web | Hospital internal services portal |
| FTP | Medical report / file transfer between departments |
| SMTP / POP3 (Email) | Inter-departmental communication (e.g., diagnostic reports to admin) |
| DHCP | Automatic IP assignment, configured per-VLAN on each Core Router |

---

## ✅ Testing & Validation

| Test | Method | Result |
|---|---|---|
| Basic Connectivity | `ping` between devices | ✅ Pass |
| Inter-VLAN Communication | `ping` across VLANs via Router-on-a-Stick | ✅ Pass |
| DHCP Assignment | `ipconfig` on client devices | ✅ Pass |
| EIGRP Route Exchange | `show ip route` / `show ip eigrp neighbors` | ✅ Pass |
| Inter-Branch Connectivity | `ping` between different hospital sites | ✅ Pass |
| DNS Resolution | Client → configured DNS server | ✅ Pass |
| HTTP Access | Web Browser → internal web server | ✅ Pass |
| FTP Transfer | `ftp` upload/download | ✅ Pass |
| NAT/PAT Translation | `show ip nat translations` | ✅ Pass |
| Email Send/Receive | SMTP/POP3 client test | ✅ Pass |
| End-to-End | Full topology verification | ✅ Pass |

Full test case table with expected vs. actual results is available in **`Project Report.pdf`** (Section 3.2.10).

---

## 🛠️ Tools & Technologies

- **Cisco Packet Tracer** — network design, configuration, and simulation
- **Cisco IOS CLI** — router and switch configuration (VLAN, EIGRP, NAT, DHCP, ACL)
- **VLSM** — hierarchical, efficient IP address allocation
- **EIGRP** — dynamic interior gateway routing protocol
- **NAT/PAT** — internet connectivity via address/port translation
- **802.1Q Trunking** — VLAN tagging across trunk links

---

## 📁 Repository Structure

This repository uses a mostly flat structure, with one folder for screenshots:

```
Smart-Hospital-Network-Project/
├── README.md
├── Computer Network Project.pkt              ← Full Cisco Packet Tracer topology file
├── Project Report.pdf                        ← Complete CSE322 lab project report
├── Edge & Core Router IP Configuration.pdf   ← IP addressing for all Edge/Core routers
├── ISP Router & ISP-Side Switch — VLAN...pdf ← R0 and ISP switch VLAN trunk configuration
├── VLAN_Config_Report(All Branch).pdf        ← VLAN, DHCP, and switch config for all 6 sites
├── Hospital_EIGRP_Config_Report(...).pdf     ← EIGRP configuration for all routers
├── NAT_PAT Configuration Summary...pdf       ← NAT/PAT and ACL configuration for all Edge routers
└── Screenshots/                              ← Topology, config, and verification test screenshots
```

| File / Folder | Contents |
|---|---|
| `Computer Network Project.pkt` | Open this in Cisco Packet Tracer to explore/run the live topology |
| `Project Report.pdf` | Full academic report — introduction, architecture, implementation, testing, conclusion |
| `Edge & Core Router IP Configuration.pdf` | WAN link addressing (ISP↔Edge, Edge↔Core) for all 6 sites |
| `ISP Router & ISP-Side Switch — VLAN Configuration.pdf` | R0 sub-interfaces + ISP switch trunk config |
| `VLAN_Config_Report(All Branch).pdf` | Per-branch VLAN, switch port, and DHCP pool configuration |
| `Hospital_EIGRP_Config_Report.pdf` | EIGRP AS 100 configuration for every router |
| `NAT_PAT Configuration Summary.pdf` | NAT/PAT + extended ACL configuration for every Edge router |
| `Screenshots/` | Full topology view, per-branch topology, CLI verification output (ping, `show ip route`, `show ip nat translations`, DHCP, EIGRP neighbors), and service test screenshots (DNS, HTTP, FTP, Email) |

> **Why `Screenshots/` is named this way:** Capitalized to match the Title Case naming convention already used across this repo (`Project Report.pdf`, `Computer Network Project.pkt`). Keeping it as a single plural word avoids spaces/special characters in the path, which keeps GitHub links and relative Markdown image references (`![](Screenshots/topology.png)`) clean and easy to type.

---

## ▶️ How to Run This Project

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (version 8.x or later recommended).
2. Clone this repository:
   ```
   git clone https://github.com/<your-username>/Design-and-Implementation-of-a-Smart-Multi-Speciality-Hospital-Network.git
   ```
3. Open **`Computer Network Project.pkt`** in Cisco Packet Tracer.
4. Switch to **Simulation Mode** to trace packet flow, or use CLI commands (`ping`, `show ip route`, `show ip nat translations`, etc.) on any device to verify connectivity.
5. Refer to the individual configuration PDFs (VLAN, EIGRP, NAT/PAT, IP addressing) if you want to rebuild any part of the topology from scratch, or read **`Project Report.pdf`** for the full design rationale and test results.

---

## ⚠️ Limitations

- Developed and tested only in a simulated environment (Cisco Packet Tracer) — not validated on physical hardware.
- No redundancy or failover mechanisms (e.g., HSRP/VRRP, secondary WAN links); a single Edge or Core router failure isolates its branch.
- EIGRP is a Cisco-proprietary protocol — not suitable for multi-vendor environments.
- No dedicated firewall or IDS/IPS; security relies solely on ACLs.
- Not tested under real-world traffic load or failure conditions.

---

## 🔮 Future Work

- Implement gateway redundancy (HSRP/VRRP) at each site
- Add a secondary WAN path for failover
- Migrate from EIGRP to OSPF for vendor-neutral interoperability
- Integrate a dedicated firewall and IDS/IPS at the network edge
- Deploy and evaluate performance in a real hospital environment

---

## 👥 Team

| Name | Student ID | Role |
|---|---|---|
| **Sultan Mahmud Rasel** | 241-15-521 | Project Lead & Network Implementation Lead |
| **Md. Foridul Islam** | 241-15-429 | Network Design, IP Configuration & Documentation |
| **Arpita Kundu** | 241-15-143 | Network Implementation, Testing & Troubleshooting |

---

## 🎓 Course Information

- **Course:** CSE322 — Computer Networks Lab
- **Department:** Computer Science and Engineering (CSE)
- **University:** Daffodil International University (DIU), Dhaka, Bangladesh
- **Supervisor:** Mr. Tanvirul Islam, Lecturer, Dept. of CSE, DIU

---

## 📚 References

1. Kurose, J. F., & Ross, K. W. (2021). *Computer Networking: A Top-Down Approach* (8th ed.). Pearson.
2. Tanenbaum, A. S., & Wetherall, D. J. (2011). *Computer Networks* (5th ed.). Pearson.
3. Stallings, W. (2017). *Data and Computer Communications* (10th ed.). Pearson.
4. Cisco Networking Academy. *Cisco Packet Tracer: Network Simulation and Visualization Tool*. Cisco Systems. Accessed: June 20, 2026; July 15, 2026; August 5, 2026.
5. Cisco Networking Academy. *Introduction to Networks: Networking Fundamentals and Configuration*. Cisco Systems. Accessed: June 25, 2026; July 18, 2026; August 6, 2026.
6. Cisco Systems. *Cisco IOS Configuration Guides and Command References*. Cisco Systems. Accessed: June 22, 2026; July 20, 2026; August 8, 2026.
7. Cisco Systems. *Network Address Translation (NAT) Configuration Guide*. Cisco Systems. Accessed: July 5, 2026; July 25, 2026; August 10, 2026.
8. Cisco Systems. *VLAN and Inter-VLAN Routing Configuration Guide*. Cisco Systems. Accessed: June 30, 2026; July 22, 2026; August 4, 2026.
9. Cisco Systems. *Dynamic Host Configuration Protocol (DHCP) Configuration Guide*. Cisco Systems. Accessed: July 3, 2026; July 24, 2026; August 6, 2026.
10. Cisco Systems. *IP Access Control Lists (ACLs) Configuration Guide*. Cisco Systems. Accessed: July 8, 2026; July 27, 2026; August 8, 2026.
11. GeeksforGeeks. *Computer Network Tutorials*. Accessed: June 18, 2026; July 14, 2026; August 3, 2026.
12. Cisco Networking Academy. *Networking Basics*. Cisco Systems. Accessed: June 24, 2026; July 19, 2026; August 9, 2026.
13. OpenAI. (2026). *ChatGPT*. AI-assisted tool used for project documentation, report organization, language refinement, troubleshooting assistance, and technical explanation. Accessed: July 20, 2026; July 25, 2026; August 10, 2026.

---

## 📄 License

This project was developed for academic purposes as part of the CSE322 Computer Networks Lab course at Daffodil International University. Feel free to reference or build upon it for educational use, with attribution.

---

<p align="center">Made with ❤️ by the CSE322 project team at Daffodil International University</p>
