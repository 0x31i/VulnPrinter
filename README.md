# HP Printer CTF Lab - Setup Guide

Quick deployment guide for HP Color LaserJet Pro MFP 4301 vulnerable printer lab with flags.

## Prerequisites

- HP Color LaserJet Pro MFP 4301
- Isolated network/VLAN
- Kali Linux or Ubuntu with SNMP tools
- Network access to printer on port 631 (IPP), 9100 (JetDirect), 161 (SNMP)

## Network Configuration

### 1. Assign Static IP to Printer

**Option A: Via Control Panel (Touchscreen)**
1. Navigate to: Setup → Network → Wired (Ethernet) → IPv4 Configuration
2. Set Static IP: `192.168.148.105`
3. Subnet: `255.255.0.0`
4. Gateway: `192.168.148.1`

**Option B: Via Web Interface**
1. Access printer at current DHCP address
2. Navigate to: Networking → Wired → IPv4 Configuration
3. Set Static IP: `192.168.148.105`
### 2. Verify Connectivity

```bash
ping 192.168.148.105
nmap -p 631,9100,161 192.168.148.105
```

Expected: All three ports should be open.

## Initial Printer Setup

### Access Embedded Web Server (EWS)

1. Navigate to: `https://192.168.148.105`
2. Accept security certificate warning
3. **Default Credentials:**
   - Username: `admin` (lowercase)
   - PIN: `68076694` (admin PIN from internal sticker)
   - If password creation required: `Password123`

### Enable Required Services

**Navigate to:** Network → Network Settings → Services
- Enable SNMP v1/v2 read/write
- Set ALL community names (SET/GET) to public.
- Toggle "Enable SNMPV1/V2 default Get... "
- Then click "APPLY"

**Verify IPP is enabled** (usually enabled by default):
- Network → Advanced Settings → IPP Printing
- Enable IPP
- Then click "APPLY"

- **Navigate to:** Network → Network Security → Firewall
- Disable the Firewall
- Ensure the Default Rule is set to "Allow"
- Then click "APPLY"

- **Navigate to:** Network → Network Security → Secure Communication
- Move all Ciphers to the "Active" table (table on the right)
- Make sure both tables are disabled (unchecked)
- Then, under the SSL/TLS Protocal section
- Ensure the Minimum Protocol Version is set to "TLS 1.0"
- Ensure the Maximum Protocol Version is set to "TLS 1.1"
- Ensure the EC Curve Version is set to "Curve P-256"
- Then click "APPLY"

- Finally, **Navigate to:** Network → Network Security → SMB
- Enable all versions of SMB

## Flag Deployment

### 1. Download and Prepare Script

```bash
# Download the deployment script from this github project and make it executable
chmod +x vulnprinter.sh

# Verify printer IP matches (edit if different)
nano vulnprinter.sh
# Check: PRINTER_IP="192.168.148.105"
```

### 2. Run Deployment Script

```bash
sh ./vulnprinter.sh
```

**What the script does:**
1. **SNMP Flags** - Sets sysLocation and sysContact via SNMP
2. **Print Jobs** - Submits PostScript documents with embedded flags
3. **Prompts for manual web configuration** for the HAN flag


## Reset Procedure

**Quick Reset:**
```bash
# Re-run deployment script
./vulnprinter.sh
```

**Full Reset:**
1. Web interface → System → Service → Restore Factory Defaults
2. Reconfigure static IP
3. Re-run deployment script
4. Redo manual web-based setup
