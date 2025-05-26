# 01 - Architecture

## Lab Objective

The goal of this lab is to simulate a small enterprise SOC and IT environment for hands-on practice in system administration, network segmentation, SIEM operations, and incident response. It enables testing and learning across IT and security domains, including firewall configuration, centralized log collection, detection rule creation, and Active Directory management.

---

## Core Components

- **pfSense**  
 Acts as the central router and firewall, handling NAT, DNS forwarding, and network segmentation. Configured with four interfaces (WAN, LAN, OPT1, OPT2) to separate trusted and untrusted zones for layered defense testing.

- **Debian Log Collector**  
Serves as an intermediate log forwarder, configured with Filebeat and Logstash to collect logs from endpoints and send them to Security Onion and Splunk.

- **Splunk Free**  
Primary SIEM platform used for log ingestion, alert creation, search query development (SPL), and dashboard visualizations to support detection and response workflows.

- **Windows Server**  
Hosts Active Directory, DNS, and GPO configurations. Provides a realistic enterprise domain environment for account provisioning and policy enforcement.

- **Windows 10 Client**  
Domain-joined workstation used to simulate end-user behavior, generate logs, test GPOs, and validate endpoint visibility within the SIEM.

- **Kali Linux (Attacker VM)**  
Deployed on an isolated network to simulate threat actor behavior, generate malicious traffic, and validate log correlation and detection logic.
---

## Network Segmentation
<img width="1886" alt="Architecture" src="https://github.com/user-attachments/assets/13da18ad-b338-45aa-8aa8-13a7626887b8" />

*Figure 1: High-level network topology illustrating segmentation between the Management, Internal, and Attacker networks routed through pfSense.*

pfSense is configured with three virtual interfaces to create a segmented, enterprise-style environment:

- **WAN (em0)** – Connects to the external internet via NAT (VirtualBox)  
- **LAN / int-net (em1)** – Internal network (`10.0.10.1/24`) hosting Windows Server and Windows 10 Client  
- **OPT1 / attack-net (em2)** – Isolated attacker network (`10.0.20.1/24`) hosting the Kali Linux VM  
- **OPT2 / mgmt-net (em3)** – Management network (`10.0.30.1/24`) hosting the Splunk SIEM server and Debian log collector

The **mgmt-net** is restricted to authorized administrative access and is used for secure monitoring, log analysis, and system administration. It is segmented to prevent exposure to the attacker and internal networks.

The Kali VM is intentionally placed on `OPT1` to simulate external or untrusted threat activity. pfSense firewall rules are configured to:

- Allow outbound internet access from OPT1 
- Block traffic from OPT1 → LAN by default  
- Allow selective, temporary access from OPT1 to LAN to test detection logic and Splunk alerting

Each segment uses either static IPs or pfSense-managed DHCP. This configuration enables safe, controlled testing of lateral movement, reconnaissance, and log visibility across isolated environments, reflecting real-world SOC conditions.
