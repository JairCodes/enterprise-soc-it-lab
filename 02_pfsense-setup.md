# 02 - pfsense setup

## Purpose

pfSense acts as the core router and firewall for the lab environment. It enables segmented networking, firewall enforcement, and traffic control between the WAN, trusted internal network (int-net), and an isolated attacker network (attack-net), and a secure management network (mgmt-net). This setup mimics enterprise firewall segmentation for both IT and SOC use cases.

---

## Network Adapter Configuration (VirtualBox)

The pfSense VM uses **four network adapters** in VirtualBox:


| Adapter | Network Type       | Interface | Name        | Purpose                                      |
|---------|--------------------|-----------|-------------|----------------------------------------------|
| 1       | NAT                | em0       | WAN         | Internet access for updates                  |
| 2       | Internal Network   | em1       | int-net     | Internal enterprise systems (AD, Clients)    |
| 3       | Internal Network   | em2       | attack-net  | Isolated network for the attacker (kali)     |
| 4       | Internal Network   | em3       | mgmt-net    | Secure admin/SOC network (Splunk, Log Server)|

## IP Addressing & Acess

Each interface was manually assigned the following IPs via the console:

- **WAN (em0):** `10.0.2.15/24` via DHCP
- **int-net (em1):** `10.0.10.1/24`
- **attack-net (em2):** `10.0.20.1/24`
- **mgmt-net (em3):** `10.0.30.1/24`

![image](https://github.com/user-attachments/assets/68ac6ed0-df9a-410f-8008-b0aa328d0bb8)
