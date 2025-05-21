# 01 - Architecture

## Lab Objective

The goal of this lab is to simulate a small enterprise SOC and IT environment for hands-on practice in system administration, network segmentation, SIEM operations, and incident response. It enables testing and learning across IT and security domains, including firewall configuration, centralized log collection, detection rule creation, and Active Directory management.

---

## Core Components

- **pfSense**  
Acts as the central router and firewall, handling NAT, DNS forwarding, and LAN/WAN segmentation. It simulates network boundaries such as internal LAN and (planned) DMZ zones.

- **Debian Log Collector**  
Serves as an intermediate log forwarder, configured with Filebeat and Logstash to collect logs from endpoints and send them to Security Onion and Splunk.

- **Splunk Free**  
Primary SIEM platform used for log ingestion, alert creation, search query development (SPL), and dashboard visualizations to support detection and response workflows.

- **Windows Server**  
Hosts Active Directory, DNS, and GPO configurations. Provides a realistic enterprise domain environment for account provisioning and policy enforcement.

- **Windows 10 Client**  
Domain-joined workstation used to simulate end-user behavior, generate logs, test GPOs, and validate endpoint visibility within the SIEM.

---

## Network Segmentation

pfSense is configured to segment the virtual environment into distinct internal networks:

- **LAN** – Internal network hosting Windows Server, Windows 10 client, and Debian log collector  
- **WAN** – Interface used to connect pfSense to the external internet via NAT  
- **(Planned)** DMZ or additional internal networks using VirtualBox’s internal adapters for isolated attacker boxes, public-facing services, or honeypot simulation

Each VM is assigned either a static IP or is managed via pfSense’s DHCP server. Firewall rules are implemented to control communication between network segments, replicating real-world access control and perimeter defense strategies.
