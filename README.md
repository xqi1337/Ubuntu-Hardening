# Ubuntu System Hardening Guide

## Table of Contents

- [[#Initial System Setup|Initial System Setup]]
- [[#BIOS/Firmware Security|BIOS/Firmware Security]]
- [[#User Account Management|User Account Management]]
- [[#Hardware Device Control|Hardware Device Control]]
- [[#Network Security|Network Security]]
- [[#Security Analysis Tools|Security Analysis Tools]]
- [[#Advanced Kernel Hardening|Advanced Kernel Hardening]]

## Initial System Setup

### Minimal Installation
Security begins at installation time. Nearly every distribution offers the option to choose a minimal setup during installation. You should disable all unnecessary services.

You can verify this using netstat, either occasionally or directly after installation:
```bash](<## Table of Contents

- [Initial System Setup](https://claude.ai/chat/660fab57-f945-45c4-baf8-2bd789940c1f#initial-system-setup)
- [BIOS/Firmware Security](https://claude.ai/chat/660fab57-f945-45c4-baf8-2bd789940c1f#biosfirmware-security)
- [User Account Management](https://claude.ai/chat/660fab57-f945-45c4-baf8-2bd789940c1f#user-account-management)
- [Hardware Device Control](https://claude.ai/chat/660fab57-f945-45c4-baf8-2bd789940c1f#hardware-device-control)
- [Network Security](https://claude.ai/chat/660fab57-f945-45c4-baf8-2bd789940c1f#network-security)
- [Security Analysis Tools](https://claude.ai/chat/660fab57-f945-45c4-baf8-2bd789940c1f#security-analysis-tools)
- [Advanced Kernel Hardening](https://claude.ai/chat/660fab57-f945-45c4-baf8-2bd789940c1f#advanced-kernel-hardening)>)

## Initial System Setup

### Minimal Installation

Security begins at installation time. Nearly every distribution offers the option to choose a minimal setup during installation. You should disable all unnecessary services.

You can verify this using netstat, either occasionally or directly after installation:

```bash
sudo netstat -tulpen
```

### Service Verification

Check which services are running and listening on your system to identify potential attack vectors.

## BIOS/Firmware Security

### Coreboot Replacement

Ideally, you can replace the current BIOS with Coreboot:

- Website: https://www.coreboot.org/
- Coreboot can potentially be more secure than proprietary BIOS options since it offers less attack surface
- However, not all hardware is compatible with Coreboot

## User Account Management

### Minimize Shell Access

You should reduce local user accounts with shell access to a minimum.

Disable shell access for specific users:

```bash
sudo usermod -s /usr/sbin/nologin username
```

Or alternatively:

```bash
sudo usermod -s /bin/false username
```

### Access Control Configuration

Another option is to control access through the `/etc/security/access.conf` file.

## Hardware Device Control

### Microphone Deactivation

List all available audio input sources:

```bash
pactl list short sources
```

Identify the ID or name of the microphone you want to disable, then mute it:

```bash
pactl set-source-mute [ID or Name] 1
```

**Note**: In some cases, physically disconnecting the microphone may be the only reliable method to ensure it is completely disabled. Reference: https://theintercept.com/2014/03/12/nsa-plans-infect-millions-computers-malware/

### Camera Deactivation

The exact module depends on your hardware. For many webcams, the uvcvideo module is responsible:

```bash
sudo modprobe -r uvcvideo
```

### Bluetooth Deactivation

```bash
sudo rfkill block bluetooth
```

## Network Security

### Firewall Configuration

A firewall represents an additional security layer and serves as an important basic configuration measure.

Basic iptables firewall script:

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
# IPv4 Default
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

# Accept all packets to an existing TCP connection
$IPTABLES -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Properly reject all packets
$IPTABLES -A INPUT -p tcp -j REJECT --reject-with tcp-reset 
$IPTABLES -A INPUT -j REJECT --reject-with icmp-port-unreachable

echo "................ done!"
```

This script configures the iptables firewall so that all incoming and forwarded connections are blocked by default, but allows outgoing connections as well as specific incoming traffic like loopback and certain ICMP traffic, plus responses to existing connections.

### GUI Firewall Management

There is a graphical frontend called GUFW that can be installed with your preferred package manager if not already present. On Debian/Ubuntu, install with:

```bash
sudo apt install gufw
```

## Security Analysis Tools

### Lynis Security Auditing

The open-source tool Lynis (https://cisofy.com/lynis/) is useful because it supports system security analysis and improvement. It gathers information about the system, installed packages, and existing configurations.

## Advanced Kernel Hardening

### Additional Security Measures

With additional measures like kernel modifications grsecurity/PaX, security can be further improved, but this also requires more effort and expertise.

### Performance vs Security Trade-offs

Consider that enhanced security measures may impact system performance and compatibility with certain applications.

## Best Practices Summary

1. **Minimal Installation**: Only install necessary components
2. **Regular Monitoring**: Check running services and open ports
3. **Hardware Control**: Disable unused hardware components
4. **Network Security**: Implement proper firewall rules
5. **User Management**: Limit shell access and user privileges
6. **Security Auditing**: Use tools like Lynis for regular assessments
7. **Stay Updated**: Keep system and security tools current

## Warning

These configurations may impact system functionality. Always test in a non-production environment first and ensure you understand the implications of each security measure before implementation.
