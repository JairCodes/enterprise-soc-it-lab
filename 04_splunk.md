# 04 - Splunk Server

## Purpose

The Splunk VM acts as the central SIEM (Security Information and Event Management) platform for the SOC & IT lab environment. It ingests and analyzes logs collected from various systems across the network to support visibility, threat detection, and troubleshooting.

Deployed on a Debian-based server and connected to the **mgmt-net** (10.0.30.0/24), Splunk receives logs from the Debian log collector via the Filebeat and Logstash pipeline. This setup enables centralized monitoring and investigation of events such as login activity, firewall traffic, and system alerts across the lab environment.

## Network Adapter Configuration (VirtualBox)

The Splunk VM uses **one network adapter** in VirtualBox:

| Adapter | Network Type     | Name     | Purpose                                      |
|---------|------------------|----------|----------------------------------------------|
| 1       | Internal Network | mgmt-net | Connects to Debian log collector and GUI VM  |

## Installation & Configuration Steps

### A. VM and OS Preparation

1. **VM Creation:** Created a new Debian 12 “netinst” VM named `splunk-server` and attached it to the `mgmt-net` internal network.
2. **Hostname and Domain:**
   - Hostname: `splunk-server`
   - Domain: `lab.local`
3. **Static IP Configuration:**
   - Interface: `enp0s3`
   - IP: `10.0.30.11/24`
   - Gateway: `10.0.30.1`
   - DNS: `10.0.10.10`, `8.8.8.8` (set in `/etc/network/interfaces`)
4. **DNS Fixes:**
   - Updated `/etc/resolv.conf` or made it immutable to fix resolution issues preventing `apt update`.

---

### B. System Updates & Dependencies

```bash
apt update && apt upgrade -y
apt install wget curl libfreetype6 libfontconfig1 unzip -y
```

---

### C. Splunk Enterprise Installation

1. **Download:**
  ```bash
  wget -O splunk-enterprise.deb "<splunk-deb-url>"
  ```
2. **Install:**
  ```bash
  dpkg -i splunk-enterprise.deb
  apt -f install -y  # Resolve dependencies
  dpkg -i splunk-enterprise.deb
  ```
3. **Verify Install:**
   - Check for `/opt/splunk` directory
  
---

### D. Start & Enable Splunk
```bash
/opt/splunk/bin/splunk enable boot-start
/opt/splunk/bin/splunk start --accept-license
```
  - You’ll be prompted to create a Splunk admin password (this is not the Linux user)
  - Confirmed Splunk is running with:
```bash
/opt/splunk/bin/splunk status
```
![image](https://github.com/user-attachments/assets/c474da17-36a3-4663-8f28-d08f24d51089)

### E. Splunk Web UI
- Once Splunk is running, access the web interface from any system on `mgmt-net`:
```bash
http://10.0.30.11:8000
```
- Login using the `admin` credentials set at first launch.

---

## Accessing the Splunk Web Interface
![image](https://github.com/user-attachments/assets/b53df2b4-9762-4716-88ce-d94bf477a950) <br>
To interact with Splunk via its GUI, we used an Ubuntu Desktop VM connected to the `mgmt-net` network.

- **Purpose:** Serve as a user-facing system to access the Splunk web interface.
- **Adapter Configuration:** One network adapter on `mgmt-net`.
- **Splunk Access:** Open a browser and navigate to `http://<splunk-ip>:8000` (e.g., `http://10.0.30.11:8000`).

No additional configuration was required on this VM beyond basic network connectivity.

---

## Log Ingestion Verification

### 1. Splunk TCP Input on Port 9997

To ingest logs from the Debian log collector via Logstash, a TCP data input was created in Splunk:

- **Settings → Data Inputs → TCP**
  - Port: `9997`
  - Source type: `syslog`
  - Index: `main` <br>
  
![image](https://github.com/user-attachments/assets/07dcc900-4ee0-43cd-beec-19a51aafd7ce) 
![image](https://github.com/user-attachments/assets/4a63f52e-bb8e-4753-96ec-f2de15722663)

To verify the listener:

```bash
echo "direct-to-splunk test $(date)" | nc 10.0.30.11 9997
```

### 2. Direct TCP Test (Bypasses Filebeat & Logstash)
From the Debian log collector VM, we sent a direct TCP message:
```bash
echo "direct-to-splunk test $(date)" | nc 10.0.30.11 9997
```
![image](https://github.com/user-attachments/assets/652e05cb-3f5b-4e9c-83e1-de1bcf9fbbbe)

### 3. Filebeat -> Logstash -> Splunk Pipeline
Filebeat and Logstash were verified as follows:
1. **Start Filebeat**
  ```bash
  systemctl enable filebeat
  systemctl start filebeat
  filebeat test output
  # Should show: logstash: OK
  ```
2. **Test Logstash Configuration**
  ```bash
  /usr/share/logstash/bin/logstash -t -f /etc/logstash/conf.d
  # Output: Configuration OK
  ```
3. **Send test log entry**
  ```bash
  echo "FILEBEAT → LOGSTASH → SPLUNK: $(date)" | tee -a /var/log/syslog
  ```
4. **Monitor Logs (optional)**
  ```bash
  journalctl -u filebeat -f
  journalctl -u logstash -f
  ```
5. **Search in Splunk**
  ```bash
  index=main sourcetype=syslog "FILEBEAT → LOGSTASH → SPLUNK"
  ```
![image](https://github.com/user-attachments/assets/afbf995e-b039-4702-8582-37b4ff67b3c6)


### 4. pfSense Syslog Ingestion (UDP 514)
To ingest logs from pfSense:
1. **Create a new index in Splunk:**
- Settings -> Indexes
  - Name: `pfsense`
2. **Enable Remote Logging in pfSense:**
- Status -> System Logs -> Settings -> Remote Logging
  - Remote server: `10.0.30.11:514`
  - IP Protocol: IPv4
  - Enabled: Everything (just to ensure it works)
3. **Create a UDP input in Splunk:**
- Settings -> Data Inputs -> UDP
  - Port: `514`
  - Source Type: `syslog`
  - Index: `pfsense` <br>

![image](https://github.com/user-attachments/assets/6b5a4f5d-dcb6-4c3c-8554-a15ef63511d1)
![image](https://github.com/user-attachments/assets/08855132-93dd-4352-9802-89b6c37fda4f)

4. **Confirm log ingestion**
Trigger a pfsense event, then search: <br>

![image](https://github.com/user-attachments/assets/49aeabe1-7369-4aaa-8ef7-56bb7d2246b7)

> At this point: <br>
> Linux syslogs flow via Filebeat -> Logstash -> Splunk over TCP 9997 into the main index. <br>
> pfSense firewall logs stream directly via UDP 514 into a dedicated pfsense index.
