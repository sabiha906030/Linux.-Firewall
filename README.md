# Linux.-Firewall

# Server Firewall Configuration Report

## Rule Analysis

Below is an in-depth analysis of each rule implemented in our firewall configuration:

### 1. Allow SSH Access

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

**Purpose**: This rule allows incoming SSH connections on the standard port 22.

**Justification**: SSH access is necessary for remote administration of the server. Without this rule, remote management would be impossible if the default policy is set to DROP or REJECT.

### 2. Allow Established and Related Connections

```bash
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

**Purpose**: This rule permits packets that are part of established connections or related to existing connections.

**Justification**: For proper network functionality, the server needs to receive responses to outgoing connections it initiated. This rule ensures that ongoing communications are not disrupted by the firewall.

### 3. Set Default Policy for Outgoing Traffic

```bash
sudo iptables -P OUTPUT ACCEPT
```

**Purpose**: Sets the default policy for outbound traffic to ACCEPT.

**Justification**: This allows the server to initiate outgoing connections without restrictions, which is generally safe since the threat model primarily focuses on incoming traffic.

### 4. Allow HTTP Traffic

```bash
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

**Purpose**: Permits incoming HTTP connections on the standard port 80.

**Justification**: Necessary for web server functionality, allowing users to access web content hosted on the server.

### 5. Allow HTTPS Traffic

```bash
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

**Purpose**: Allows incoming HTTPS connections on the standard port 443.

**Justification**: Required for secure web communications through SSL/TLS, enabling encrypted data transfer between clients and the server.

### 6. SYN Flood Protection

```bash
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s -j ACCEPT
```

**Purpose**: Limits the rate of new TCP connections with the SYN flag to 1 per second.

**Justification**: Protects against SYN flood attacks, which attempt to consume server resources by sending a large number of SYN packets without completing the TCP handshake.

### 7. NULL Packet Protection

```bash
sudo iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
```

**Purpose**: Drops packets with no TCP flags set (NULL packets).

**Justification**: NULL packets are often used in network reconnaissance and can indicate a port scan. Legitimate traffic should never have all TCP flags unset, making this an effective rule to block certain types of scanning activity.

### 8. Log Blocked Traffic

```bash
sudo iptables -A INPUT -j LOG --log-prefix "BLOCKED: "
```

**Purpose**: Logs all packets that reach this rule (which would be all packets not matched by previous rules).

**Justification**: Provides visibility into blocked traffic, aiding in threat analysis and firewall rule optimization.

### 9. Log New Allowed Connections

```bash
sudo iptables -I INPUT 1 -m state --state NEW -j LOG --log-prefix "ALLOWED: "
```

**Purpose**: Logs all new connections that are allowed through the firewall.

**Justification**: Helps monitor legitimate traffic and detect anomalies in access patterns, valuable for security auditing.

### 10. Save Firewall Rules

```bash
sudo iptables-save > /etc/iptables.rules
sudo iptables-save | sudo tee /etc/iptables.rules > /dev/null
```

**Purpose**: Saves the current iptables configuration to a file.

**Justification**: Ensures rules can be restored after a system reboot.

### 11. Restore Rules at Boot

```bash
sudo sh -c "echo 'iptables-restore < /etc/iptables.rules' >> /etc/rc.local"
sudo chmod +x /etc/rc.local
```

**Purpose**: Configures the system to restore iptables rules on boot.

**Justification**: Ensures continuous protection even after server restarts.

## Additional Protection Against Common Attacks

### ICMP Flood Protection (Added Rule)

```bash
sudo iptables -A INPUT -p icmp -m limit --limit 1/s --limit-burst 4 -j ACCEPT
sudo iptables -A INPUT -p icmp -j DROP
```

**Purpose**: Limits the rate of ICMP packets to 1 per second with a burst of 4 packets.

**Justification**: Prevents ICMP flood attacks (ping floods) which can consume bandwidth and potentially cause denial of service, while still allowing legitimate ICMP traffic for network diagnostics.