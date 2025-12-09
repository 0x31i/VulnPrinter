# AXIS M1025 IoT Camera CTF Lab - Setup Guide

## Overview

Pokemon-themed CTF lab environment with 27 flags distributed across a realistic AXIS camera filesystem. This setup script creates intentionally vulnerable configurations for educational penetration testing.

**⚠️ WARNING: ISOLATED LAB ONLY - NEVER USE IN PRODUCTION**

---

## Lab Specifications

- **Target Device**: AXIS M1025 Network Camera (or compatible Linux system)
- **Firmware Version**: 10.5.0 (simulated)
- **Total Flags**: 27 (5 Easy, 13 Medium, 9 Hard)
- **Flag Format**: `FLAG{POKEMON_NAME#########}`
- **Shell**: POSIX-compliant (BusyBox ash compatible)

---

## Prerequisites

### Network Requirements
- Isolated VLAN or air-gapped network
- Static IP for camera: `192.168.1.132` (recommended)
- No internet access (prevents firmware updates)

### System Requirements
- Linux-based embedded system or VM
- Minimum 512MB storage for flag files
- Root/sudo access required
- SSH enabled

---

## Quick Setup

### Step 1: Transfer Script to Camera

```bash
# From your admin workstation
scp vulnaxis.sh root@192.168.1.132:/tmp/

# Or via physical access/serial console
# Copy script to /tmp/ directory
```

### Step 2: Execute Setup Script

```bash
# SSH into camera
ssh root@192.168.1.132

# Make script executable
chmod +x /tmp/vulnaxis.sh

# Run setup (takes ~10 seconds)
/tmp/vulnaxis.sh
```

**Expected Output:**
```
[*] Starting Axis Camera CTF setup - v5.0
[+] Creating comprehensive Axis camera directory structure...
[+] Distributing EASY FLAGS across writable directories...
[+] Flag #1: VAPIX Configuration...
[+] Flag #4: System Log Entry...
...
[*] CTF Setup Complete - FIXED VERSION v5.0
    EASY flags: 5 (basic enumeration)
    MEDIUM flags: 13 (exploitation required)  
    HARD flags: 9 (advanced techniques)
    TOTAL: 27 flags
```

### Step 3: Verify Installation

```bash
# Check flag distribution
find /mnt -name "*.conf" -o -name "*.txt" 2>/dev/null | wc -l
find /var/lib/persistent -type f 2>/dev/null | wc -l
find /var/cache/recorder -type f 2>/dev/null | wc -l

# Quick flag check (should show flags)
grep -r "FLAG{" /var/lib/axis/conf/ 2>/dev/null | head -3
cat /var/log/messages | grep FLAG
```

### Step 4: Configure Access

**Default Credentials:**
```
Username: root
Password: pass
```

**SSH Access:**
```bash
ssh root@192.168.1.132
# Password: pass
```

**Web Interface:**
```
http://192.168.1.132
https://192.168.1.132
```

---

## Directory Structure Overview

The script creates 8 major writable directory trees:

| Directory | Purpose | Flags |
|-----------|---------|-------|
| `/mnt/flash/` | Firmware, bootloader, factory configs | 3 |
| `/var/lib/persistent/` | Persistent storage and licenses | 4 |
| `/var/cache/recorder/` | Recording streams and sessions | 3 |
| `/var/lib/axis/` | Core AXIS configurations | 5 |
| `/var/www/local/` | Web interface and CGI scripts | 2 |
| `/dev/shm/axis/` | Shared memory and IPC | 1 |
| `/run/axis/` | Runtime services | 1 |
| `/usr/local/axis/` | Custom applications | 3 |

**Total Files Created**: ~40+ configuration files across realistic AXIS camera paths

---

## Flag Distribution by Difficulty

### EASY Flags (5) - Basic Enumeration
| ID | Flag | Location | Technique |
|----|------|----------|-----------|
| #1 | `FLAG{FRODO27189846}` | `/var/lib/axis/conf/vapix.conf` | File reading |
| #4 | `FLAG{GIMLI42137246}` | `/var/log/messages` | Log analysis |
| #7 | `FLAG{MERRY36385024}` | `/var/www/local/admin/index.html` | HTML comment |
| #14 | `FLAG{SARUMAN83479324}` | `/var/cache/recorder/streams/primary/` | Config file |
| #19 | `FLAG{THEODEN40558954}` | `/mnt/flash/config/factory/` | Factory data |

### MEDIUM Flags (13) - Exploitation Required
Require path traversal, service enumeration, or CGI exploitation.

### HARD Flags (9) - Advanced Techniques  
Require firmware analysis, race conditions, SSRF, or crypto weaknesses.

---

## Testing Access Points

### Web Services
```bash
# Test HTTP access
curl http://192.168.1.132/

# VAPIX API endpoint
curl http://root:pass@192.168.1.132/axis-cgi/param.cgi

# Admin interface
curl http://192.168.1.132/admin/
```

### File System Enumeration
```bash
# Student reconnaissance commands
find /mnt -type f 2>/dev/null
find /var/lib/persistent -name "*.lic" 2>/dev/null
ls -laR /var/cache/recorder/ 2>/dev/null
cat /var/log/messages
```

### CGI Scripts
```bash
# Vulnerable command injection endpoint
/var/www/local/axis-cgi/webhook.cgi

# Test SSRF vulnerability
curl "http://192.168.1.132/axis-cgi/webhook.cgi?url=http://127.0.0.1:8888/internal"
```

---

## Reset and Cleanup

### Full Reset
```bash
# Remove all CTF files
rm -rf /var/lib/axis/conf/vapix.conf
rm -rf /var/lib/persistent/
rm -rf /var/cache/recorder/
rm -rf /mnt/flash/config/
rm -rf /usr/local/axis/
rm -rf /var/www/local/
rm -f /var/log/messages

# Or re-run script (overwrites all)
/tmp/vulnaxis.sh
```

### Between Teams
```bash
# Reset file permissions (if modified)
find /var/lib/axis -type f -exec chmod 644 {} \;
find /var/www/local/axis-cgi -type f -name "*.cgi" -exec chmod 755 {} \;

# Clear temporary files
rm -rf /dev/shm/axis/*
rm -rf /run/axis/*
rm -f /tmp/race_result.txt
```

---

## Troubleshooting

### Script Fails to Execute

**Symptom:** `Permission denied` or `not found`

**Solution:**
```bash
# Ensure script has execute permissions
chmod +x /tmp/vulnaxis.sh

# Verify POSIX shell compatibility
sh /tmp/vulnaxis.sh  # Use sh instead of bash
```

### Directories Not Writable

**Symptom:** `mkdir: cannot create directory`

**Solution:**
```bash
# Check available space
df -h

# Ensure running as root
whoami  # Should return 'root'

# Try mounting writable
mount -o remount,rw /
```

### Flags Not Visible

**Symptom:** `grep "FLAG{" returns no results`

**Solution:**
```bash
# Check if script completed
ls -la /var/lib/axis/conf/vapix.conf

# Re-run specific flag creation
grep -A 10 "Flag #1" /tmp/vulnaxis.sh | sh

# Verify file contents
cat /var/lib/axis/conf/vapix.conf
cat /var/log/messages
```

### Cannot SSH to Camera

**Symptom:** `Connection refused` or `Connection timed out`

**Solution:**
```bash
# Check SSH service status
ps | grep sshd

# Start SSH if not running
/usr/sbin/sshd

# Check firewall (if present)
iptables -L -n
```

### Students Find Flags Too Quickly

**Solution:**
```bash
# Increase difficulty by restricting permissions
chmod 600 /mnt/flash/config/factory/*
chmod 600 /var/lib/persistent/system/licenses/*

# Hide files more deeply
mv /var/cache/recorder/.temp/.recording_session_20240101 \
   /var/cache/recorder/.temp/...recording_session_20240101
```

---

## Network Isolation Checklist

- [ ] Camera on dedicated VLAN (192.168.1.0/24)
- [ ] No default gateway configured
- [ ] Firewall rules block internet access  
- [ ] DNS servers removed or pointed to internal
- [ ] Students connect via VPN or isolated subnet
- [ ] No routing between CTF network and production
- [ ] Physical network cable separation (if possible)

---

## Student Documentation

Provide students with:
- Target IP: `192.168.1.132`
- Challenge scope: Full system access allowed
- Tools allowed: nmap, curl, ssh, any pentesting tools
- Off-limits: Physical access (unless UART/JTAG challenges)
- Flag submission: Automated platform or manual verification

**Do NOT provide:**
- Root credentials (`root:pass`)
- Script source code (`vulnaxis.sh`)
- Flag locations or hints
- Directory structure map

---

## OWASP IoT Top 10 Coverage

This lab covers:

| Category | Examples in Lab |
|----------|-----------------|
| IoT-01 | Hardcoded passwords in configs |
| IoT-02 | Unauthenticated CGI endpoints |
| IoT-03 | Command injection in webhook.cgi |
| IoT-04 | Insecure firmware storage |
| IoT-05 | Outdated crypto algorithms (DES, MD5) |
| IoT-06 | Unencrypted credential storage |
| IoT-07 | Plaintext API tokens and keys |
| IoT-08 | UART/JTAG debug interfaces enabled |
| IoT-09 | Default admin credentials |
| IoT-10 | Race condition vulnerabilities |

---

## Support and Updates

**Script Version:** v5.0 (Fixed cgroup paths, comprehensive coverage)

**Known Issues:**
- None currently

**Future Enhancements:**
- Additional firmware vulnerabilities
- RTSP stream exploitation
- ONVIF protocol challenges

**Questions?** 
- Review the instructor writeup: `AXIS_IoT_CTF_Instructor_Writeup_v2.md`
- Check implementation guide: `IoT_Camera_Security_Training__Comprehensive_Vulnerability_Implementation_Guide_for_AXIS_Systems.md`

---

## Quick Reference

### Essential Commands
```bash
# Setup
chmod +x vulnaxis.sh && ./vulnaxis.sh

# Verify flags
grep -r "FLAG{" /var/lib/axis/ /var/log/ /mnt/flash/ 2>/dev/null | wc -l

# Reset permissions
find /var/lib/axis -type f -exec chmod 644 {} \;

# Full cleanup
rm -rf /var/lib/axis/ /var/lib/persistent/ /var/cache/recorder/ /mnt/flash/config/ /usr/local/axis/
```

### Attack Surface Summary
- **Ports**: 22 (SSH), 80 (HTTP), 443 (HTTPS), 554 (RTSP)
- **Protocols**: HTTP, SSH, RTSP, VAPIX API
- **CGI Scripts**: webhook.cgi (SSRF), param.cgi (injection)
- **Weak Crypto**: DES, 3DES, MD5, SHA1, RC4
- **Debug Interfaces**: UART (115200 baud), JTAG (enabled)

---

**Lab setup complete. Happy hunting! 🎯**
