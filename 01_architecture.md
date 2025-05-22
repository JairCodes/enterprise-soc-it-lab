# 01 - Architecture

## Lab Objective

The goal of this lab is to simulate a small enterprise SOC and IT environment for hands-on practice in system administration, network segmentation, SIEM operations, and incident response. It enables testing and learning across IT and security domains, including firewall configuration, centralized log collection, detection rule creation, and Active Directory management.

---

## Core Components

- **pfSense**  
 Acts as the central router and firewall, handling NAT, DNS forwarding, and network segmentation. Configured with three interfaces (WAN, LAN, OPT1) to separate trusted and untrusted zones for layered defense testing.

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

pfSense is configured with three virtual interfaces to create a segmented, enterprise-style environment:

- **WAN (em0)** – Connects to the external internet via NAT (VirtualBox)  
- **LAN (em1)** – Trusted internal network hosting Windows Server, Windows 10 Client, and the Debian log collector  
- **OPT1 / targetnet (em2)** – Isolated attacker network hosting the Kali Linux VM

The Kali VM is intentionally placed on `OPT1` to simulate external or untrusted threat activity. pfSense firewall rules are configured to:

- Allow outbound internet access from OPT1 
- Block traffic from OPT1 → LAN by default  
- Allow selective, temporary access from OPT1 to LAN to test detection logic and Splunk alerting

Each segment uses either static IPs or pfSense-managed DHCP. This configuration enables safe, controlled testing of lateral movement, reconnaissance, and log visibility across isolated environments, reflecting real-world SOC conditions.
