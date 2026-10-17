# CMPG 325 — COMPUTER NETWORKS
## PROJECT SUBMISSION & REPOSITORY DOCUMENTATION

**Student Name:** Mateye Sydney Selota  
**Student Number:** 37633805  
**Project ID:** CMPG325-2026-125  
**Client ID:** CLI-125  
**Assigned Organization:** Leano Gaming Lounge (Mahikeng)  
**Assigned Network Block:** `192.168.55.0/24`  

---

## 1. PROJECT OVERVIEW

The Leano Gaming Lounge, located in Mahikeng, required a robust, highly secure, and scalable local area network (LAN) infrastructure to support its modern entertainment operations. 

This project encompasses the complete engineering lifecycle-ranging from requirements analysis and Variable Length Subnet Masking (VLSM) subnet planning to Cisco Packet Tracer simulation, core network service deployment, and extended Access Control List (ACL) security enforcement.

### Key Implementation Features:
* **802.1Q VLAN Trunking:** Multi-switch VLAN trunking configured between switches `SW1` and `SW2` across interface `FastEthernet0/24`.
* **Router-on-a-Stick (ROAS):** Subinterface routing implemented on Cisco 2911 Router `R1` (`Gig0/0.10` through `Gig0/0.99`).
* **CCTV Segmentation:** Isolated surveillance network (`VLAN 30`) secured against administrative office traffic.
* **Guest Wi-Fi Isolation (CR3):** Public visitor Wi-Fi network (`VLAN 40`) isolated from internal servers and office systems.
* **Core Network Services:** Internal DHCP, DNS, and HTTP server (`192.168.55.66`) and external web server (`192.168.55.82` hosting `www.leanogaminglounge.com`).

---

## 2. IP ADDRESSING & SUBNET ARCHITECTURE

The network utilizes Variable Length Subnet Masking (VLSM) derived from the assigned base block **`192.168.55.0/24`**.

| VLAN | Segment Name | Network Address | Subnet Mask / CIDR | Usable Range | Allocation Type |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **10** | Gaming Lounge | `192.168.55.0` | `255.255.255.240` (`/28`) | `.1` – `.14` | DHCP Pool (`192.168.55.2–14`) |
| **20** | Office / Admin | `192.168.55.16` | `255.255.255.240` (`/28`) | `.17` – `.30` | DHCP Pool (`192.168.55.18–30`) |
| **30** | CCTV Security | `192.168.55.32` | `255.255.255.240` (`/28`) | `.33` – `.46` | Static (`192.168.55.34–38`) |
| **40** | Guest Wi-Fi (CR3)| `192.168.55.48` | `255.255.255.240` (`/28`) | `.49` – `.62` | DHCP via Router R1 |
| **50** | Core Server | `192.168.55.64` | `255.255.255.248` (`/29`) | `.65` – `.70` | Static Server (`192.168.55.66`) |
| **60** | Management | `192.168.55.72` | `255.255.255.248` (`/29`) | `.73` – `.78` | Static Switch SVIs (`SW1`: `.74`, `SW2`: `.75`) |
| **70** | Printer Network | `192.168.55.88` | `255.255.255.248` (`/29`) | `.89` – `.94` | Static Printer (`192.168.55.90`) |
| **WAN**| Internet Link | `192.168.55.80` | `255.255.255.252` (`/30`) | `.81` – `.82` | Static Link (Server: `192.168.55.82`) |

---

## 3. SECURITY ACCESS CONTROL LISTS (ACLS)

Security policies are enforced at the Router `R1` subinterface level:

* **`CCTV-IN` (VLAN 30):** Permits CCTV webcams to reach the gateway (`.33`) and Server Subnet (`.64/29`), while denying access to the Office network (`.16/28`).
* **`GUEST-IN` (VLAN 40):** Permits DHCP traffic and outbound Internet access while dropping all packets destined for internal LAN subnets.
* **`GAMING-IN` (VLAN 10):** Prevents gaming stations from reaching CCTV cameras and administrative network printers.
* **`PRINTER-IN` (VLAN 70):** Restricts printer access exclusively to administrative staff on `VLAN 20`.

---

## 4. VERIFICATION & TESTING MATRIX

| Test ID | Category | Test Procedure | Expected Result | Status |
| :---: | :--- | :--- | :--- | :---: |
| **T01** | DHCP | Request dynamic IP on Gaming PC1 | Obtains `192.168.55.x` lease with DNS `.66` | **PASS** |
| **T02** | DHCP | Request dynamic IP on Office PC1 | Obtains `192.168.55.x` lease with gateway `.17` | **PASS** |
| **T03** | DNS | Run `nslookup server.leano.local` on Office PC1 | Resolves to `192.168.55.66` | **PASS** |
| **T04** | HTTP | Open `www.leanogaminglounge.com` in browser | Custom Leano Gaming Lounge portal renders | **PASS** |
| **T05** | Static IP | Ping `CAM4` (`192.168.55.37`) from Core Server | Successful ICMP replies across trunk | **PASS** |
| **T06** | Printer | Ping Printer (`192.168.55.90`) from Office PC1 | Successful cross-VLAN printing access | **PASS** |
| **T07** | Wi-Fi | Connect Wireless Laptop to `Leano-Guest` | Obtains `192.168.55.50` from gateway `.49` | **PASS** |
| **T08** | ACL | Ping Office PC1 from CCTV `CAM1` | Blocked by `CCTV-IN` ACL | **PASS** |
| **T09** | ACL | Ping Network Printer from Gaming PC1 | Blocked by `GAMING-IN` ACL | **PASS** |
| **T10** | ACL | Ping Core Server from Guest Wireless Laptop | Blocked by `GUEST-IN` ACL | **PASS** |
| **T11** | Trunking | Run `show interfaces trunk` on switches | `Fa0/24` operating as active 802.1Q trunk | **PASS** |
| **T12** | ROAS | Run `show ip interface brief` on Router R1 | Subinterfaces `Gig0/0.10–.99` display `up/up` | **PASS** |
| **T13** | Inter-VLAN| Ping Office PC3 from Office PC1 across switches| ICMP communication succeeds via ROAS | **PASS** |
| **T14** | Services | View DHCP lease table on Core Server GUI | Active MAC-to-IP leases logged correctly | **PASS** |
| **T15** | NVRAM | Run `copy running-config startup-config` | Configuration written to startup memory | **PASS** |

---
