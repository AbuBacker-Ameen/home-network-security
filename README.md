# Home Network Security Project

A comprehensive project designed to enhance the security and privacy of a home network using a Raspberry Pi server. This project integrates multiple tools and configurations to provide advanced network monitoring, DNS privacy, and secure access.

---

## **Overview**

This project transforms a Raspberry Pi 3B+ into a robust network security server. The primary objectives are:

- To block ads and malicious domains using **Pi-hole**.
- To encrypt DNS queries via **Unbound** with **WireGuard VPN**.
- To monitor network traffic and detect intrusions with **Suricata** (IDS).
- To secure the Raspberry Pi with **iptables firewall**.
- To centralize logs for effective monitoring and incident analysis using a **SIEM solution**.

---

## **Features**

1. **Ad-Blocking and DNS Privacy**:

   - Pi-hole as a DNS sinkhole.
   - Unbound for DNS resolution with DNSSEC validation.
   - WireGuard VPN configured with Proton servers to encrypt DNS traffic.

2. **Intrusion Detection**:

   - Suricata IDS to monitor and analyze network traffic.

3. **Firewall**:

   - Advanced iptables configuration to secure the server for both LAN and WAN access.

4. **Centralized Logging**:

   - Logs from Pi-hole, Suricata, and the router integrated into a SIEM system for analysis.

---

## **Setup Guide**

### **1. Prerequisites**

- Raspberry Pi 3B+ with Ubuntu Server 24.04.1 LTS.
- Static IP address assigned to the Raspberry Pi.
- SSH access configured.
- A minimum of 4GB swap space was created to prevent RAM failures, as the Raspberry Pi has only 2GB of RAM.
- Recommended SD card: Kingstone Canvas Go Plus microSD 64GB, or equivalent with high read/write speeds for optimal performance.

### **2. Installation and Configuration**

#### **2.1 Pi-hole and Unbound**

- Install Pi-hole:
  ```bash
  curl -sSL https://install.pi-hole.net | bash
  ```
- Configure Unbound as a recursive DNS resolver:
  - Create a configuration file at `/etc/unbound/unbound.conf.d/pi-hole.conf`.
  - Enable DNSSEC and set Pi-hole to forward DNS queries to Unbound.

#### **2.2 WireGuard VPN**

- Install WireGuard:
  ```bash
  sudo apt install wireguard
  ```
- Configure WireGuard to connect to Proton VPN servers.
- Redirect all DNS traffic through the VPN tunnel.

#### **2.3 Suricata IDS**

- Install Suricata:
  ```bash
  sudo apt install suricata
  ```
- Configure Suricata for **AF\_PACKET mode** with optimized settings for limited resources.
- Enable and update rules:
  ```bash
  sudo suricata-update enable-source oisf/et/open
  sudo suricata-update
  ```

#### **2.4 iptables Firewall**

- Define rules for LAN and WAN access:
  - Allow SSH, DNS, and HTTP(S) traffic.
  - Drop all other traffic by default.
- Save and apply rules persistently:
  ```bash
  sudo iptables-save > /etc/iptables/rules.v4
  ```

#### **2.5 Centralized Logging (SIEM)**

- Install and configure a SIEM solution (e.g., ELK Stack, Graylog) ( I did not decide witch software to use yet):
  - Forward logs from Pi-hole, Suricata, and the router to the SIEM server.
  - Visualize logs using dashboards.

---

## **Network Diagram**

```
+--------------------+          +-------------------------+         +--------------------+
|    Internet       +---------->        Router          +---------> Raspberry Pi      |
|                    |          |   (Gateway/Firewall)   |         | (Pi-hole, Unbound, |
|                    |          |                        |         |   Suricata IDS,    |
|                    |          | DHCP Disabled          |         |   WireGuard VPN)   |
+--------------------+          +-------------------------+         +---------+----------+
                                                                         |
                                                                         |
                                                    +--------------------v-------------------+
                                                    |                 Local Devices          |
                                                    | (Computers, IoT, Phones, etc.)         |
                                                    +----------------------------------------+
```

- **Traffic Flow**:
  - DNS Requests: Devices -> Raspberry Pi (Pi-hole -> Unbound -> WireGuard -> Proton VPN -> Internet).
  - IDS Monitoring: Suricata passively monitors all network traffic.
  - Firewall: iptables restricts and secures access.



---

## **Project Goals**

- Enhance privacy and security for all connected devices.
- Provide visibility into network activities.
- Implement an easily replicable and cost-effective solution.

---
