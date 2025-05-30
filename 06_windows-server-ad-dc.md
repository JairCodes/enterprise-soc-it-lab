# 06 - Windows Server (Domain Controller)

## Purpose

The Windows Server VM serves as the primary domain controller (DC) for the internal lab domain `lab.local`. It enables centralized authentication, DNS, and Group Policy management for all systems in the internal network (`int-net`).

---

## Network Adapter Configuration (VirtualBox)

The Windows Server VM uses **one network adapter** in VirtualBox:

| Adapter | Network Type     | Name    | Purpose                                     |
|---------|------------------|---------|---------------------------------------------|
| 1       | Internal Network | int-net | Connects to pfSense LAN and domain clients  |

---

## IP Addressing & Access

The static IP configuration for the domain controller is as follows:

- **IP Address:** `10.0.10.10/24`
- **Gateway:** `10.0.10.1` (pfSense LAN)
- **DNS Server (initial):** `10.0.10.1`
- **DNS Server (final):** `127.0.0.1` (after AD DS role promotion) <br>

Here's how to do it via Control Panel:
1. Open Control Panel
2. Go to:
    - `Network and Interact` -> `Network and Sharing Center`
3. In the left pane, click:
    - `Change adapter settings`
4. Right-click on the network connection -> click **Properties**
5. Scroll down and select:
    - `Internet Protocol Version 4 (TCP/IPv4)` -> click **Properties**
6. Then set the following and click **OK**: <br>
![image](https://github.com/user-attachments/assets/9873220e-6e59-4e03-a3e4-a0062deaae04)

---

## Verifying Network Connectivity

To ensure the server can reach the pfSense gateway, we ran:

```cmd
ping 10.0.10.1
```
Sucessful replies confirmed correct network setup:
![image](https://github.com/user-attachments/assets/b6e075f0-93bd-4206-80e3-b10e1cdfa746)

---

## Installing Active Directory Domain Services (AD DS)

1. Open **Server Manager**
    - Click `Manage` -> `Add Roles and Features` <br>

![image](https://github.com/user-attachments/assets/484ca8eb-8192-4dcd-b7d4-13c5f6be785c)

2. Click **Next** through the first few screens until you reach:
    - **"Server Roles"** <br>
4. Select `Active Directory Domain Services`
5. When prompted, click **Add Features** to include required AD management tools

![image](https://github.com/user-attachments/assets/e7f0feaa-d114-4f5e-8808-a6b8e5eb0397)

---

## Promoting Server to a Domain Controller

After the installation completes:
1. Click the **yellow exclamation icon** in Server Manager
2. Select **"Promote this server to a domain controller"**

![image](https://github.com/user-attachments/assets/37b4228b-947b-4941-a068-aac2315a6ad0)

4. Choose Add a **new forest**, and enter:
```
Root domain name: lab.local
```

![image](https://github.com/user-attachments/assets/f19b7a4b-106f-454c-9dd5-64ca8833d20c)

5. Complete the wizard and reboot when prompted
   
---

## Verifying Active Directory & DNS

After reboot:
Verify Active Directory,
- In Server Manager, click on:
  -  `Tools` -> `Active Directory Users and Computers`
  - You should see the domain tree for `Lab.local`
 
![image](https://github.com/user-attachments/assets/a50254cf-d63d-41a4-a3f3-e84070354fac)

  
Verify DNS,
- In Server Manager, click on:
    -  `Tools` -> `DNS`
    -  Confirm zones for `lab.local` and `_msdcs.lab.local`

![image](https://github.com/user-attachments/assets/bf8d904f-5a1d-46e3-b59b-b7eb68f1ffce)
