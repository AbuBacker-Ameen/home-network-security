# Pi-hole Setup Documentation

This document provides a step-by-step guide to setting up Pi-hole.

---

## **Overview**

Pi-hole is a network-wide ad blocker that also serves as a DNS sinkhole. In this setup, Pi-hole will:
- Block advertisements and malicious domains.
- Act as the primary DNS server for your network.
- Work in tandem with Unbound (a recursive DNS resolver) and WireGuard VPN to enhance privacy and security.

---

## **Prerequisites**

1. **Hardware and OS Requirements:**
   - Raspberry Pi 3B+ running Ubuntu Server 24.04.1 LTS.
   - Static IP address assigned to the Raspberry Pi.
   - A minimum of 4GB swap space configured for optimal performance.

2. **Network Requirements:**
   - DHCP disabled on the router.
   - Router configured to forward DNS queries to the Raspberry Pi.

3. **Tools and Access:**
   - SSH access to the Raspberry Pi.
   - Basic familiarity with Linux commands.

---

## **Installation Steps**

### **1. Update System Packages**

Before installing Pi-hole, update the Raspberry Pi’s packages:
```bash
sudo apt update && sudo apt upgrade -y
```

### **2. Install Pi-hole**

Use the official installation script to set up Pi-hole:
```bash
curl -sSL https://install.pi-hole.net | bash
```

- Follow the on-screen prompts to configure Pi-hole:
  - **Interface Selection:** Select the interface (e.g., `eth0`) connected to the network.
  - **Upstream DNS Provider:** Choose a placeholder (e.g., Google DNS). This will later be replaced by Unbound.
  - **Blocklists:** Use the default blocklists provided during setup.

### **3. Configure Static IP**

Ensure that the Raspberry Pi maintains a static IP address:

Edit the Netplan configuration file:
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Example configuration:
```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.0.2/24
      gateway4: 192.168.0.1
      nameservers:
        addresses:
          - 127.0.0.1
```
Apply the configuration:
```bash
sudo netplan apply
```

### **4. Configure Router Settings**

1. Disable DHCP on the router.
2. Set the Raspberry Pi’s IP as the primary DNS server in the router’s configuration.

---

## **Testing the Setup**

### **1. Verify Pi-hole Functionality**

Access the Pi-hole web interface at `http://<RaspberryPi_IP>/admin`.

- **Login:** Use the password provided during installation.
- **Query Logs:** Check the "Query Logs" to verify that Pi-hole is receiving and blocking DNS queries.

### **2. Test Ad-Blocking**

Visit websites known for ads (e.g., `cnn.com`) and confirm that ads are not being displayed.

---

## **Next Steps**

1. **Integrate Unbound**:
   - Configure Unbound as a recursive DNS resolver.
2. **Secure DNS Queries**:
   - Encrypt DNS traffic using WireGuard VPN.
3. **Monitor Traffic**:
   - Deploy Suricata for intrusion detection.
4. **Harden Security**:
   - Implement iptables firewall.

---

## **Troubleshooting**

### **Common Issues**

1. **Cannot Access Pi-hole Web Interface:**
   - Verify that the Raspberry Pi’s IP is correct.
   - Check if the Pi-hole service is running:
     ```bash
     sudo systemctl status pihole-FTL
     ```

2. **DNS Queries Not Forwarded:**
   - Ensure the router is configured to use the Raspberry Pi as the DNS server.
   - Check Pi-hole logs for errors:
     ```bash
     tail -f /var/log/pihole.log
     ```

---

## **Resources**

- [Pi-hole Official Documentation](https://docs.pi-hole.net/)
- [Ubuntu Server Documentation](https://ubuntu.com/server/docs)
- [Netplan Configuration Guide](https://netplan.io/)

