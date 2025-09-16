# Linux System Hardening Guide

## Table of Contents

1. [Initial System Setup](#initial-system-setup)
2. [BIOS/UEFI Security](#biosuefi-security)
3. [User Account Management](#user-account-management)
4. [Hardware Security](#hardware-security)
   - [Microphone Deactivation](#microphone-deactivation)
   - [Camera Deactivation](#camera-deactivation)
   - [Bluetooth Deactivation](#bluetooth-deactivation)
5. [Network Security](#network-security)
   - [Firewall Configuration](#firewall-configuration)
   - [Graphical Firewall Management](#graphical-firewall-management)
6. [Security Auditing](#security-auditing)
7. [Advanced Hardening](#advanced-hardening)
8. [Warnings](#warnings)
9. [Contributing](#contributing)
10. [License](#license)

---

## Initial System Setup

Security begins with the initial installation. Nearly every Linux distribution offers the option to choose a minimal setup during installation.

### Service Management

**Disable all unnecessary services** to reduce the attack surface. You can check running services immediately after installation or periodically using:

```bash
sudo netstat -tulpen
```

This command displays all listening ports and associated processes, helping you identify unwanted services.

### Best Practices for Initial Setup
- Choose minimal installation when available
- Regularly audit running services
- Remove unnecessary packages and dependencies
- Keep the system updated

---

## BIOS/UEFI Security

### Coreboot Alternative

Consider replacing the current proprietary BIOS with **Coreboot** for enhanced security:

- **Website**: [https://www.coreboot.org/](https://www.coreboot.org/)
- **Benefits**: Potentially more secure than proprietary BIOS options due to reduced attack surface
- **Limitation**: Not all hardware is compatible with Coreboot

**Note**: Research hardware compatibility thoroughly before attempting Coreboot installation.

---

## User Account Management

### Minimize Shell Access

**Reduce local user accounts with shell access** to the absolute minimum required for system operation.

#### Disable Shell Access for Users

```bash
# Option 1: Set nologin shell
sudo usermod -s /usr/sbin/nologin username

# Option 2: Set false shell
sudo usermod -s /bin/false username
```

#### Alternative Access Control

Configure access restrictions using the **access.conf** file:

```bash
sudo nano /etc/security/access.conf
```

This provides granular control over user access patterns and login restrictions.

---

## Hardware Security

### Microphone Deactivation

#### List Audio Input Sources

```bash
pactl list short sources
```

#### Mute Specific Microphone

Identify the ID or name of the microphone you want to disable, then:

```bash
pactl set-source-mute [ID or Name] 1
```

**Physical Security Note**: In some cases, **physical disconnection** of the microphone may be the only reliable method to ensure complete deactivation.

**Reference**: [NSA Plans to Infect Millions of Computers with Malware](https://theintercept.com/2014/03/12/nsa-plans-infect-millions-computers-malware/)

### Camera Deactivation

The exact module depends on your hardware. For most webcams, the **uvcvideo** module is responsible:

```bash
sudo modprobe -r uvcvideo
```

**Note**: This change is temporary and will revert after reboot. For permanent deactivation, add the module to the blacklist.

### Bluetooth Deactivation

```bash
sudo rfkill block bluetooth
```

---

## Network Security

### Firewall Configuration

A firewall provides an **additional security layer** and serves as an important basic configuration measure.

#### IPTables Firewall Script

```bash
#!/bin/sh
# iptables Firewall Script
echo "Loading Firewall ..."

##################
# iptables 
##################
IPTABLES="/sbin/iptables"

##################
# Purge/Flush 
##################
# Delete all rules
$IPTABLES -F 
$IPTABLES -t nat -F
$IPTABLES -t mangle -F

# Delete all rule chains
$IPTABLES -X 
$IPTABLES -t nat -X
$IPTABLES -t mangle -X

##################
# Rules
##################
# IPv4 Default policies
$IPTABLES -P INPUT DROP
$IPTABLES -P FORWARD DROP
$IPTABLES -P OUTPUT ACCEPT

# Allow loopback interface traffic
$IPTABLES -A INPUT -i lo -j ACCEPT 
$IPTABLES -A OUTPUT -o lo -j ACCEPT

# Allow ICMP response packets
$IPTABLES -A INPUT -p icmp -m icmp --icmp-type echo-reply -j ACCEPT 
$IPTABLES -A INPUT -p icmp -m icmp --icmp-type echo-request -j ACCEPT 
$IPTABLES -A INPUT -p icmp -m icmp --icmp-type destination-unreachable -j ACCEPT

# Accept all packets for existing TCP connections
$IPTABLES -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Properly reject all other packets
$IPTABLES -A INPUT -p tcp -j REJECT --reject-with tcp-reset 
$IPTABLES -A INPUT -j REJECT --reject-with icmp-port-unreachable

echo "................ done!"
```

#### Firewall Configuration Explanation

This script configures the iptables firewall to:

- **Block all incoming and forwarded connections by default**
- **Allow outgoing connections**
- **Permit specific incoming traffic**:
  - Loopback interface traffic
  - Specific ICMP traffic types
  - Responses to existing connections

### Graphical Firewall Management

**GUFW** provides a graphical frontend for easier firewall management.

#### Installation on Debian/Ubuntu

```bash
sudo apt install gufw
```

---

## Security Auditing

### Lynis Security Auditing Tool

**Lynis** is an open-source tool that assists in analyzing and improving system security.

- **Website**: [https://cisofy.com/lynis/](https://cisofy.com/lynis/)
- **Purpose**: System analysis, package inventory, and configuration assessment
- **Benefits**: Identifies security weaknesses and provides recommendations

#### Key Features
- System information gathering
- Installed package analysis
- Configuration file examination
- Security recommendations and scoring

---

## Advanced Hardening

### Kernel Security Enhancements

Additional measures such as **kernel modifications** with grsecurity/PaX can further improve security:

#### Considerations
- **Enhanced Protection**: Provides additional kernel-level security features
- **Implementation Complexity**: Requires more effort and expertise
- **Maintenance Overhead**: May complicate system updates and maintenance

---

## Warnings

⚠️ **Critical Security Warnings**:

- **Backup Systems**: Always create full system backups before implementing hardening measures
- **Test Environment**: Test all configurations in a non-production environment first
- **Service Dependencies**: Disabling services may break application functionality
- **Hardware Compatibility**: Verify hardware compatibility before BIOS/firmware modifications
- **Physical Access**: Physical security measures are essential - software hardening cannot protect against physical attacks
- **Regular Updates**: Keep all security tools and system components updated
- **Professional Consultation**: Consider consulting security professionals for critical systems

---

## Contributing

We welcome contributions to improve this hardening guide:

1. **Fork** the repository
2. **Create** a feature branch
3. **Test** your changes thoroughly
4. **Submit** a pull request with detailed descriptions
5. **Follow** security best practices in your contributions

### Contribution Guidelines
- Verify all commands and configurations
- Include relevant security references
- Test on multiple distributions when possible
- Maintain clear, concise documentation

---

## License

This Linux System Hardening Guide is released under the **MIT License**.

### MIT License Summary
- **Permission**: Free to use, modify, and distribute
- **Condition**: Include original copyright notice
- **Limitation**: No warranty provided

For the complete license text, see the LICENSE file in the repository root.

---
