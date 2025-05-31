# 03 - Debian Log Collector

## Purpose

The Debian “log-collector” VM serves as a centralized logging server within the SOC & IT lab’s **mgmt-net** (10.0.30.0/24). Its primary role is to receive logs from other lab systems (e.g., via Filebeat → Logstash) and forward or store them for analysis in a SIEM platform such as Splunk.

---

## Network Adapter Configuration (VirtualBox)

The Debian "log-collector" VM uses **one network adapter** in VirtualBox:

| Adapter | Network Type     | Name    | Purpose                                     |
|---------|------------------|---------|---------------------------------------------|
| 1       | Internal Network | mgmt-net| Connects to pfSense LAN and other clients   |

## Install Prerequisites

Before installing Filebeat and Logstash, update the package lists and ensure that HTTPS transport and certificate support are available.

1. **Become root**
   ```cmd
   su –
   ```
   Enter the root password you set during isntallation.
2. **Update package lists**
   ```cmd
   apt update
   ```
3. **Install prerequisite packages**
   ```cmd
   apt install -y apt-transport-https ca-certificates curl gnupg
   ```

## Add Elastic GPG Key and Repository

To install Filebeat and Logstash from Elastic’s APT repository, add the Elastic GPG key and create the repository entry.

1. **Download and install the Elastic GPG key**  
   ```cmd
   curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
2. **Create the Elastic APT source list**
   ```cmd
   echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] \
   https://artifacts.elastic.co/packages/8.x/apt stable main" \
   > /etc/apt/sources.list.d/elastic-8.x.list
   ```
3. **Update package lists again**
   ```cmd
   apt update
   ```

## Install and Configure Filebeat

With the Elastic repository in place, install Filebeat and enable the “system” module to collect local syslog/auth logs. Then verify the configuration.

1. **Install Filebeat**  
   ```cmd
   apt install -y filebeat
   ```
2. **Enable the “system” module**
   ```cmd
   filebeat modules enable system
   ```
3. **Test Filebeat configuration**
   ```cmd
   filebeat test config
   # Expected output: “Config OK”
   ```
4. **Test Filebeat output (Logstash not yet running)**
   ```cmd
   filebeat test output
   # Expected output: “dial tcp 127.0.0.1:5044: connection refused” 
   # (indicates Filebeat is installed but Logstash is not yet listening on 5044)
   ```

## Install and Configure Logstash

After Filebeat is in place, install Logstash, create a simple Beats-to-stdout pipeline, and ensure it listens on port 5044.

1. **Install Logstash**  
   ```cmd
   apt install -y logstash
   ```
2. **Create the Beats input configuration**
    - File: /etc/logstash/conf.d/01-beats-input.conf
    - Enter:
    ```cmd
    input {
      beats {
        port => 5044
      }
    }
    ```
3. **Create the stdout output configuration**
    - File: /etc/logstash/conf.d/30-stdout-output.conf
    - Enter:
    ```cmd
    output {
      stdout { codec => rubydebug }
    }
    ```
4. **Verify pipeline syntax**
    ```cmd
    /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
    # Expected output: “Configuration OK”
    ```
5. **Configure pipelines.yml to load all 'conf.d' files**
    -  Edit: /etc/logstash/pipelines.yml
    -  Add:
   ```cmd
   - pipeline.id: main
     path.config: "/etc/logstash/conf.d/*.conf"
   ```
6. **Fix permissions on Logstash queue directory** <br>
   When starting Logstash, you may see an error about /var/lib/logstash/queue not being writable. <br>
   Resolve it by:
     ```cmd
    mkdir -p /var/lib/logstash/queue
    chown -R logstash:logstash /var/lib/logstash
    ```
7. **Start and enable Logstash service**
    ```cmd
    systemctl enable logstash
    systemctl restart logstash
    systemctl status logstash
    # Expected status: “Active: active (running)”
    ```
    ![image](https://github.com/user-attachments/assets/3651b0f2-805c-4652-ac0f-b17dd686acac)

8. **Verify Logstash is listening on port 5044**
    ```cmd
    ss -tlnp | grep 5044
    ```
    ![image](https://github.com/user-attachments/assets/e909c921-5588-49d1-b9fe-76c96e4ee322)

> At this point, Logstash is running and accepting Beats connections on port 5044. <br>
> Filebeat can now forward events into this pipeline.
