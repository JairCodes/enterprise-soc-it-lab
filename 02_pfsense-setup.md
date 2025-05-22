# 02 - pfsense setup

## Purpose

pfSense acts as the core router and firewall for the lab environment. It enables segmented networking, firewall enforcement, and traffic control between the WAN, trusted internal network (LAN), and an isolated attacker network (OPT1 / targetnet). This setup mimics enterprise firewall segmentation for both IT and SOC use cases.

---

## Network Adapter Configuration (VirtualBox)

The pfSense VM uses **three network adapters** in VirtualBox:


| Adapter | Network Type       | Interface | Name        | Purpose                        |
|---------|--------------------|-----------|-------------|--------------------------------|
| 1       | NAT                | em0       | WAN         | Internet access for updates    |
| 2       | Internal Network   | em1       | LAN         | Trusted systems (AD, Clients)  |
| 3       | Internal Network   | em2       | OPT1        | Isolated attacker network      |

- **WAN (NAT):** Gets a dynamic IP (`10.0.2.15/24`)
- **LAN (labnet):** Assigned IP `10.0.10.1/24`
- **OPT1 (targetnet):** Assigned IP `10.0.20.1/24`
