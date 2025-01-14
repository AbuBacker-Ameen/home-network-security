# Suricata Setup

## **Overview**

This document covers the installation and configuration of **Suricata** on a Raspberry Pi, set up as an Intrusion Detection System (IDS). Suricata inspects network traffic and detects suspicious activity, working alongside Pi-hole and Unbound for a secure and private home network.

---

## **What is Suricata?**

Suricata is a powerful open-source network threat detection engine. It can:

- Detect malicious activity through deep packet inspection.
- Generate alerts for network intrusions.
- Monitor DNS queries and other network traffic.
- Operate in **IDS mode** (passive) or **IPS mode** (active).

In this setup, we use **IDS mode** since the Raspberry Pi is not acting as a gateway.

---

## **Why Choose Suricata?**

Suricata is a powerful open-source network threat detection engine. It was chosen over Snort for its multithreading capabilities, which are essential for efficient use of hardware resources, even on constrained devices like the Raspberry Pi. While Snort is a robust alternative, its lack of native multithreading makes it less suitable for setups handling moderate to high traffic. Suricata also supports modern protocols and integrates seamlessly with various logging systems, aligning with the goal of centralized SIEM integration.

---

## **Packet Capture Method: AF_PACKET**

**AF_PACKET** is used as the capture method in this setup due to its efficiency and compatibility with the Raspberry Pi. It enables kernel-level packet capture and supports:

- **Clustered Processing**: Load balancing across threads for better performance.
- **Ease of Use**: Simplifies configuration compared to more complex options like PF_RING or Netmap.

### Other Options:
- **PF_RING**: High-performance packet capture but requires additional installation steps.
- **Netmap**: Suitable for very high-speed networks but not ideal for low-resource devices.
- **PCAP**: Traditional packet capture method, less efficient than AF_PACKET for this use case.

---

## **Installation**

### **1. Update the System**

Ensure the system is up-to-date:

```bash
sudo apt update && sudo apt upgrade -y
```

### **2. Install Suricata**

Install Suricata using the package manager:

```bash
sudo apt install suricata -y
```

Verify the installation:

```bash
suricata --build-info
```

### **3. Configure Network Interface**

Identify the active network interface:

```bash
ip a
```

For example, if your interface is `eth0`, note this for later configuration.

---

## **Configuration**

### **1. Backup Default Configuration**

Before making changes, back up the default Suricata configuration file:

```bash
sudo cp /etc/suricata/suricata.yaml /etc/suricata/suricata.yaml.bak
```

### **2. Edit Suricata Configuration**

Open the configuration file:

```bash
sudo nano /etc/suricata/suricata.yaml
```

Key changes:

1. **Set the Home Network:**

   ```yaml
   vars:
     address-groups:
       HOME_NET: "[192.168.0.1/24, 10.0.0.0/24]"
   ```

2. **Enable Rule Sources:**

   Ensure `ET Open Rules` is enabled in the `rule-files` section:

   ```yaml
   rule-files:
     - suricata.rules
   ```

3. **Set AF_PACKET Mode:**

   Locate the `af-packet` section and configure:

   ```yaml
   af-packet:
     - interface: eth0
       threads: 1
       cluster-id: 99
       cluster-type: cluster_flow
       defrag: yes
       use-mmap: yes
   ```

4. **Logging Optimization:**

   Reduce log verbosity to save resources temporarily until a SIEM solution is implemented:

   ```yaml
   logging:
     default-log-level: notice
   ```

### **3. Enable Automatic Updates for Rules**

Install Suricata Update:

```bash
sudo apt install suricata-update -y
```

Download and enable the Emerging Threats Open Ruleset:

```bash
sudo suricata-update enable-source oisf/et/open
sudo suricata-update
```

Restart Suricata:

```bash
sudo systemctl restart suricata
```

---

## **Testing Suricata**

### **1. Run Suricata in Test Mode**

Ensure the configuration file has no errors:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```

### **2. Test with Sample Traffic**

Simulate malicious traffic to generate alerts:

```bash
curl http://testmyids.com
```

Check the logs for alerts:

```bash
sudo tail -f /var/log/suricata/fast.log
```

---

## **Performance Optimization**

To ensure smooth performance on a Raspberry Pi:

1. **Adjust Memory Limits:**

   ```yaml
   stream:
     memcap: 64mb
     reassembly:
       memcap: 256mb
   ```

2. **Limit Logs:**

   Reduce logging to save resources temporarily until a centralized SIEM solution is implemented:

   ```yaml
   eve-log:
     - alert:
         metadata: no
         tagged-packets: no
   ```

3. **Set CPU Affinity:**

   ```yaml
   threading:
     set-cpu-affinity: yes
     cpu-affinity:
       - management-cpu-set:
           cpu: [ 0 ]
       - worker-cpu-set:
           cpu: [ 1 ]
   ```

---

## **Troubleshooting**

1. **Check Logs:**

   ```bash
   sudo tail -f /var/log/suricata/suricata.log
   ```

2. **Validate Configuration:**

   ```bash
   sudo suricata -T
   ```

3. **Resource Issues:**

   - Reduce logging verbosity temporarily.
   - Adjust memory limits in the configuration file.

---

## **Next Steps**

- **Firewall Setup:** Implement `iptables` firewall.
- **WireGuard VPN Integration:** Encrypt DNS queries and secure network traffic.
- **Centralized Logging:** Forward Suricata logs to a SIEM for analysis.

