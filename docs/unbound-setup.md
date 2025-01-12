# README: Unbound Recursive DNS Resolver Setup

## **Overview**

This document outlines the steps for setting up **Unbound** as a secure and private recursive DNS resolver for use with **Pi-hole**. This integration enhances network security by combining Pi-hole's ad-blocking and filtering capabilities with Unbound's ability to perform recursive DNS queries, bypassing third-party DNS providers and ensuring privacy and DNSSEC validation. Unbound adds DNSSEC validation to ensure authenticity and integrates seamlessly with Pi-hole for enhanced privacy and security in your home network.

---

## **Features**

1. **Recursive DNS Resolution**:

   - Resolves DNS queries directly from root servers, bypassing third-party DNS providers.
   - Ensures minimal latency and maximum privacy.

2. **DNSSEC Validation**:

   - Validates DNS responses to prevent spoofing and ensure authenticity.

3. **Integration with Pi-hole**:

   - Serves as the upstream DNS resolver for Pi-hole, ensuring that all DNS queries are resolved privately and securely.

   - Enhances ad-blocking efficiency by leveraging Pi-hole's filtering capabilities while directly resolving DNS queries using Unbound.

   - Provides an additional layer of privacy by bypassing third-party DNS services and validating responses through DNSSEC.:

   - Functions as an upstream DNS server for Pi-hole.

   - Handles all DNS queries forwarded by Pi-hole.

4. **Optimized for Raspberry Pi**:

   - Lightweight configuration tailored for limited resources.

---

## **Network Architecture**

Below is a simplified text-based diagram of the network setup, highlighting Unbound's role as a recursive DNS resolver that queries root servers directly for enhanced privacy and security:

```
   [Client Devices]
        |     
        v
     [Pi-hole]
        |     
        v
     [Unbound]
        |     
        v
   [Root DNS Servers]
```

- **Client Devices**: These include computers, smartphones, and IoT devices on your network.
- **Pi-hole**: Handles DNS queries from client devices and forwards them to Unbound.
- **Unbound**: Acts as a recursive resolver, directly querying root DNS servers.
- **Root DNS Servers**: The ultimate authority for DNS resolution.

---

## **Installation and Configuration**

### **1. Install Unbound**

1. Update the system:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. Install Unbound:

   ```bash
   sudo apt install unbound
   ```

3. Verify the installation:

   ```bash
   unbound -V
   ```

### **2. Configure Unbound**

1. Create a new configuration file for Pi-hole:

   ```bash
   sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
   ```

2. Add the following configuration:

   ```yaml
   server:
     # Listen on localhost
     interface: 127.0.0.1
     port: 5335

     # Hide DNS resolver identity
     hide-identity: yes
     hide-version: yes

     # Privacy options
     use-caps-for-id: yes
     cache-min-ttl: 3600
     cache-max-ttl: 86400

     # Prevent DNS leaks
     harden-glue: yes
     harden-dnssec-stripped: yes

     # DNSSEC validation
     auto-trust-anchor-file: "/var/lib/unbound/root.key"

     # Performance tuning
     prefetch: yes
     prefetch-key: yes
     num-threads: 2
     msg-cache-size: 4m
     rrset-cache-size: 8m

     # Root server hints
     root-hints: "/etc/unbound/root.hints"

     # Access control
     access-control: 127.0.0.0/8 allow
     access-control: ::1 allow
   ```

3. Download the latest root hints file:

   ```bash
   sudo curl -o /etc/unbound/root.hints https://www.internic.net/domain/named.root
   ```

4. Update the DNSSEC root key:

   ```bash
   sudo unbound-anchor -a /var/lib/unbound/root.key
   ```

5. Test the configuration:

   ```bash
   sudo unbound-checkconf
   ```

6. Restart the Unbound service:

   ```bash
   sudo systemctl restart unbound
   ```

### **3. Configure Pi-hole to Use Unbound**

1. Log in to the Pi-hole admin interface.

2. Navigate to **Settings > DNS**.

3. Under "Upstream DNS Servers", add:

   - `127.0.0.1#5335`

4. Save the settings and restart Pi-hole:

   ```bash
   pihole restartdns
   ```

---

## **Testing Unbound**

1. Verify DNS resolution:

   ```bash
   dig example.com @127.0.0.1 -p 5335
   ```

2. Test DNSSEC validation:

   - Successful query:
     ```bash
     dig sigok.verteiltesysteme.net @127.0.0.1 -p 5335
     ```
   - Failed query (DNSSEC properly enforced):
     ```bash
     dig fail01.dnssec.works @127.0.0.1 -p 5335
     ```

---

## **Best Practices**

1. **Regular Updates**:

   - Keep Unbound and the root hints file updated for optimal performance and security.

2. **Monitor Logs**:

   - Check Unbound logs for errors or unusual activity:
     ```bash
     sudo tail -f /var/log/unbound/unbound.log
     ```

3. **Minimal Logging**:

   - To reduce resource usage, configure Unbound for minimal logging by adding this to the configuration file:
     ```yaml
     log-queries: no
     log-replies: no
     verbosity: 1
     ```

---

## **Troubleshooting**

1. **Unbound Not Starting**:

   - Check the configuration for errors:
     ```bash
     sudo unbound-checkconf
     ```
   - Review the logs:
     ```bash
     sudo journalctl -u unbound
     ```

2. **Slow DNS Resolution**:

   - Increase `msg-cache-size` and `rrset-cache-size` in the configuration.
   - Enable prefetching: `prefetch: yes` and `prefetch-key: yes`.

3. **Failed DNSSEC Validation**:

   - Ensure the root key file is updated:
     ```bash
     sudo unbound-anchor -a /var/lib/unbound/root.key
     ```

---

## **Next Steps**

- **Integrate WireGuard VPN**:

  - Redirect DNS queries through an encrypted tunnel.

- **SIEM Integration**:

  - Gather centralized logs from Pi-hole, Unbound, router and other network components for monitoring and analysis.

- **Advanced Tuning**:

  - Optimize Unbound for higher traffic or specific use cases.
