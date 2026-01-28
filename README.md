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

**Expected Output:**
```
╔════════════════════════════════════════════════════════════════╗
║     HP Printer CTF Flag Deployment                            ║
╚════════════════════════════════════════════════════════════════╝

[1/3] Deploying SNMP flags...
  → Deploying FLAG{LUKE47239581} via SNMP sysLocation...
  → Deploying FLAG{LEIA83920174} via SNMP sysContact...

[2/3] Creating realistic print jobs with flags...
  → Sending print job 1 (with PADME & MACE flags)...
    ✓ Job 1 sent via netcat
  → Sending additional print jobs...
    ✓ Job 2 sent
    ✓ Job 3 sent

[3/3] Manual web configuration required for HAN flag:
  URL: https://192.168.148.105
  Press ENTER when HAN flag is configured...
```

### 3. Manual Web Configuration (HAN Flag)

**When script prompts, complete this step:**

1. Open web browser: `https://192.168.148.105`
2. Login with admin credentials
3. Navigate to: **General → About The Printer → Configure Information**
4. **Printer Nickname** field: Enter `HP-MFP-FLAG{HAN62947103}`
5. Click **Apply**
6. Return to terminal and press **ENTER** to continue

## Verification

The script automatically verifies deployment. You should see:

```
╔════════════════════════════════════════════════════════════════╗
║                    VERIFICATION                                ║
╚════════════════════════════════════════════════════════════════╝

[SNMP] Checking sysLocation and sysContact...
FLAG{LUKE47239581}
FLAG{LEIA83920174}

[IPP] Checking print jobs in queue...
  ✓ Flags found in upcoming jobs!
job-originating-user-name (nameWithoutLanguage) = FLAG{PADME91562837}
job-name (nameWithoutLanguage) = OVERCLOCK-Job-FLAG{MACE41927365}

[IPP] Checking printer attributes...
printer-location (nameWithoutLanguage) = OC-Server-Room-B | FLAG{LUKE47239581}
printer-info (nameWithoutLanguage) = HP-MFP-FLAG{HAN62947103}
```

### Manual Verification (Optional)

```bash
# SNMP enumeration
snmpwalk -v2c -c public 192.168.148.105 1.3.6.1.2.1.1

# IPP printer attributes
ipptool -tv ipp://192.168.148.105:631/ipp/print - << EOF
{
    NAME "Get Printer Attributes"
    OPERATION Get-Printer-Attributes
    GROUP operation-attributes-tag
    ATTR charset attributes-charset utf-8
    ATTR language attributes-natural-language en
    ATTR uri printer-uri \$uri
    ATTR keyword requested-attributes all
    STATUS successful-ok
}
EOF

# IPP print jobs
ipptool -tv ipp://192.168.148.105:631/ipp/print - << EOF
{
    NAME "Get Jobs"
    OPERATION Get-Jobs
    GROUP operation-attributes-tag
    ATTR charset attributes-charset utf-8
    ATTR language attributes-natural-language en
    ATTR uri printer-uri \$uri
    ATTR keyword which-jobs all
    ATTR keyword requested-attributes all
    STATUS successful-ok
}
EOF
```

## Troubleshooting

### Script Fails at SNMP Step

**Error:** `Timeout: No Response from 192.168.148.105`

**Solution:**
1. Verify SNMP is enabled: EWS → Network → Other Settings → SNMPv1/v2c
2. Check write community string is `public` or `private`
3. Test manually:
   ```bash
   snmpget -v2c -c public 192.168.148.105 1.3.6.1.2.1.1.6.0
   ```

### Print Jobs Not Appearing

**Error:** Jobs sent but not visible in queue

**Solution:**
1. Check web interface Job Queue tab for "Guest: Print" jobs
2. Jobs may have already printed - check completed jobs:
   ```bash
   ipptool -tv ipp://192.168.148.105:631/ipp/print - << EOF
   {
       NAME "Get Completed Jobs"
       OPERATION Get-Jobs
       GROUP operation-attributes-tag
       ATTR charset attributes-charset utf-8
       ATTR language attributes-natural-language en
       ATTR uri printer-uri \$uri
       ATTR keyword which-jobs completed
       ATTR keyword requested-attributes all
       STATUS successful-ok
   }
   EOF
   ```
3. Re-run deployment script to submit jobs again

### Cannot Access Web Interface

**Error:** Browser shows "This site can't be reached"

**Solution:**
1. Verify IP: `ping 192.168.148.105`
2. Check port 80/443: `nmap -p 80,443 192.168.148.105`
3. Try HTTP instead: `http://192.168.148.105`
4. Factory reset via control panel if unresponsive

### Wrong Admin PIN

**Error:** PIN `68076694` doesn't work

**Solution:**
1. Open front cover of printer
2. Look for white sticker near toner cartridge area
3. Find 8-digit PIN on sticker
4. Update script with correct PIN, or
5. Print Network Configuration Report (Reports menu) to display PIN

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
