# 05 - Kali Attacker

## Purpose

The Kali Linux VM serves as the attacker machine within the SOC & IT environment. It is used to simulate threat activity such as scanning, enumeration, and exploitation attempts against other systems on the isolated **attack-net**. This allows for safe red-team-style testing and validation of log collection, alerting, and response mechanisms.

---

## Network Adapter Configuration (VirtualBox)

The Kali VM uses **one network adapter** in VirtualBox:

| Adapter | Network Type     | Name       | Purpose                                   |
|---------|------------------|------------|-------------------------------------------|
| 1       | Internal Network | attack-net | Isolated attacker subnet for test traffic |
