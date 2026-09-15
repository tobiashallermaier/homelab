# My Homelab & Network setup

This is the documentation of my homelab. I built and configured this entire setup to learn how real networks and servers work.

---

The house has 4 floors (Basement, Ground Floor, 1st Floor, Attic). Everything is cabled back to the central switch in the basement.

A few rules I stuck to while wiring everything up:
* **Port 8** on every switch is always the uplink back to the basement
* **Port 1** always powers the UniFi U7 Pro AP via PoE+.
* The old FRITZ!Box is just an IP client now. It only handles DECT phones.

---
## Hardware
| Device | Hardware / OS | What it does |
| :--- | :--- | :---
| **Modem** | DrayTek Vigor 166 | Runs in pure bridge mode |
| **Firewall** | Protectli VP2420 (OPNsense) | Routing, firewall rules, DHCP, DNS |
| **Core Switch** | USW-Lite-8-PoE (Basement) | Connects the firewall, the AP, and the uplinks to other floors. |
| **Floor Switches** | 3x USW-Lite-8-PoE | One switch on each floor. |
| **VoIP** | AVM FRITZ!Box 7590 AX | Kept only for DECT phones. |
| **Proxmox Node 1** | Custom PC / 64GB RAM | Proxmox VE (Home Assistant & 24/7 services)
| **Proxmox Node 2** | HP EliteDesk 800 G3 SFF | Proxmox VE (24/7 services)

---
## VLANs
| VLAN | Name | Subnet | What's inside
| :--- | :--- | :--- | :--- |
| 1 | Admin | `10.10.1.0/24` | Web UIs (OPNsense, UniFi Controller) |
| 10 | Main | `10.10.10.0/24` | Main PCs, smart phones |
| 20 | IoT | `10.10.20.0/24` | Smart home gear. | 
| 30 | Trusted | `10.10.30.0/24` | Services only accessable in LAN |
| 40 | DMZ | `10.10.40.0/24` | Services exposed through port forwarding |
| 50 | Work | `10.10.50.0/24` | Work VLAN, can't talk to others |
| 60 | Guest | `10.10.60.0/24` |  Internet only access for guests. |

---
## Folders
* [`/networking`](networking/): Firewall rules, DNS configs, and switch port setups.
* [`/server`](server/): Proxmox config and what VMs are running.
* [`/services`](services/): Home Assistant setup and Docker Compose files.

---
## Notes on Security
* All sensitive info has been replaced with placeholders.
* Inter-VLAN routing is set to **Default-Deny**. Traffic between subnets is blocked unless explicitly allowed by a rule.
