# Networking & Firewall Configuration

This section documents how traffic is routed and filtered between VLANs.

---

# Firewall Policy

The firewall uses a strict **Default-Deny** approach on all interfaces:
1. All traffic between internal VLANs is blocked by default.
2. Only explicitly allowed connections can pass.

### Traffic Matrix (Source --> Destination)

| From \ To | Admin (1) | Main (10) | IoT (20) | Trusted (30) | DMZ (40) | Work (50) | Guest (60) | WAN
| :--- | :--- | :--- | :--- | :--- | :--- | :---| :--- | :--- |
| **Admin (1)** | - | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Main (10)** | ❌ | - | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ |
| **IoT (20)** | ❌ | ❌ | - | ❌ | ❌ | ❌ | ❌ | Only NTP* |
| **Trusted (30)** | ❌ | ❌ | ❌ | - | ❌ | ❌ | ❌ | ✅ |
| **DMZ (40)** | ❌ | ❌ | ❌ | ❌ | - | ❌ | ❌ | ✅ |
| **WORK (50)** | ❌ | ❌ | ❌ | ❌ | ❌ | - | ❌ | ✅ |
| **Guest (60)** | ❌ | ❌ | ❌ | ✅* | ❌ | ❌ | - | ✅* |

*Legend:*
* ✅ **Allowed**: Full outbound access allowed.
* ❌ **Blocked**: Default-deny drop rule.
* `*` **Rule Exceptions**: Restricted to specific IPs/Ports.

---

## Key Firewall Rules & Exceptions

* **Intra-VLAN Client Isolation (DMZ & Work):**
  * Both **DMZ (40)** and **Work (50)** enforce strict client isolation.
  * Devices inside these subnets cannot communicate with each other - they can't even ping or discover neighboring devices in the same `/24` subnet.
* **Admin VLAN (1) Access is WireGuard Only:**
  * None of the local subnets can access the Admin VLAN directly.
  * Access to administrative Web UIs requires a WireGuard connection.
* **IoT (20) Internet Exception:**
  * Generic IoT hardware is strictly blocked from the internet and can only query NTP on OPNsense for time synchronization.
  * **Exception:** The Home Assistant OS VM (`10.10.20.251`) has outbound WAN access for updates.
* **Guest (60) Access**:
  * Outbound traffic to the WAN is bandwidth limited via OPNsense traffic shapers
  * Has access to Trusted (30) for dedicated shared services

---
## Switch Port Architecture

To keep cabling maintainable across all four floors, switches share a standardized baseline profile, with specific end devices patched per floor.

### Global Port Standard
* **Port 8:** `802.1Q Trunk (All VLANs tagged)` - Uplink to basement core switch
* **Port 1:** `Trunk / Native VLAN 1` - Powers the local UniFi U7 Pro AP via PoE+
* **Ports 2-7:** Standard client access ports (assigned as needed).

## DNS & DHCP setup

* **DHCP (Per Subnet):** Managed by Kea DHCP on OPNsense for dynamic subnets.
* **No DHCP in Admin (1):** The Admin VLAN has DHCP **disabled**. All management interfaces use documented static IP configuration to prevent unauthorized access.
* **DNS:** Unbound DNS on OPNsense handles internal hostname resolution.
