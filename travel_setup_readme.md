# Travel Router & NAS Setup Guide - Raspberry Pi Edition

## Overview
This guide provides step-by-step instructions for building a portable travel router and NAS system using a Raspberry Pi 3 Model B+ and Netgear R6230 AC1200 router. This budget-friendly solution is perfect for families traveling with multiple devices requiring reliable internet access and media streaming capabilities.

## Hardware Requirements

### Core Components
- **Raspberry Pi 3 Model B+**
  - 1.4GHz quad-core BCM2837B0 ARMv8 64-bit CPU
  - 1GB RAM
  - Dual-band WiFi (2.4GHz and 5GHz)
  - Bluetooth 4.2/BLE
  - 4x USB ports
  - Ethernet port
  - GPIO header
- **Netgear R6230 AC1200 Smart WiFi Router**
  - AC1200 dual-band wireless
  - External antennas for better range
  - Gigabit Ethernet ports
  - Guest network capability
- **MicroSD Card** (64GB+ Class 10/U3 recommended)
- **External USB 3.0 Hard Drive** (2TB+ for media storage)
- **Portable USB Hub** (powered, 4+ ports)
- **Micro USB Power Supply** (5V 2.5A minimum)
- **Ethernet Cable** (short, for Pi-to-router connection)
- **Portable Power Bank** (20,000mAh+ for flight usage)

### Optional Components
- **USB WiFi Adapter** (for dual WiFi setup)
- **Case for Raspberry Pi** (with cooling fan)
- **USB-to-SATA Adapter** (for internal drives)
- **32GB USB Flash Drive** (for additional storage)

## Software Stack
- **Raspberry Pi OS Lite** (64-bit, headless)
- **OpenMediaVault** (NAS functionality)
- **Plex Media Server** (Media streaming)
- **Syncthing** (File synchronization)
- **Pi-hole** (DNS filtering and ad blocking)
- **OpenVPN** (VPN client)
- **Netdata** (System monitoring)

## Step-by-Step Setup

### Phase 1: Raspberry Pi Preparation

#### 1.1 Operating System Installation
```bash
# Flash Raspberry Pi OS to microSD card
1. Download Raspberry Pi Imager from official website
2. Flash "Raspberry Pi OS Lite (64-bit)" to microSD card
3. Enable SSH by creating empty file named 'ssh' in boot partition
4. Configure WiFi by creating wpa_supplicant.conf in boot partition:

country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="YourWiFiNetwork"
    psk="YourWiFiPassword"
}
```

#### 1.2 Initial System Setup
```bash
# SSH into Raspberry Pi
ssh pi@raspberrypi.local
# Default password: raspberry

# Update system
sudo apt update && sudo apt upgrade -y

# Change default password
passwd

# Configure system
sudo raspi-config
# - Enable SSH permanently
# - Expand filesystem
# - Set timezone
# - Enable I2C and SPI if needed
# - Set GPU memory split to 64MB

# Install essential packages
sudo apt install -y curl wget git vim htop
```

### Phase 2: Network Router Configuration

#### 2.1 Netgear R6230 Initial Setup
```bash
# Connect to router via web interface
http://192.168.1.1
# Default login: admin/password

# Basic Configuration:
1. Change admin password
2. Set WiFi network name: "FamilyTravel-5G" and "FamilyTravel-2.4G"
3. Set strong WiFi passwords
4. Enable guest network: "FamilyTravel-Guest"
5. Configure port forwarding for Pi services:
   - Port 8096 (Plex) -> Pi IP:8096
   - Port 8384 (Syncthing) -> Pi IP:8384
   - Port 80 (Pi-hole admin) -> Pi IP:80
```

#### 2.2 Advanced Router Settings
```bash
# Quality of Service (QoS) Setup:
1. Enable Dynamic QoS
2. Set media streaming as high priority
3. Limit guest network bandwidth to 50%

# Security Settings:
1. Disable WPS
2. Enable WPA3 (or WPA2 if WPA3 unavailable)
3. Disable remote management unless needed
4. Enable firewall with default deny-all incoming
```

### Phase 3: NAS Setup with OpenMediaVault

#### 3.1 OpenMediaVault Installation
```bash
# Install OMV on Raspberry Pi
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash

# Reboot after installation
sudo reboot

# Access OMV web interface
http://[PI_IP_ADDRESS]
# Default login: admin/openmediavault
```

#### 3.2 Storage Configuration
```bash
# Connect USB hard drive to Pi
# In OMV web interface:

1. Storage > Disks - Verify USB drive detected
2. Storage > File Systems - Create ext4 filesystem on USB drive
3. Storage > Shared Folders - Create folders:
   - "Media" (for Plex content)
   - "Travel-Files" (for daily footage/documents)
   - "Sync" (for Syncthing)
4. Services > SMB/CIFS - Enable and configure shares
```

### Phase 4: Media Server Setup

#### 4.1 Plex Media Server Installation
```bash
# Install Plex via OMV-Extras
# In OMV web interface:
1. System > OMV-Extras
2. Install Docker
3. Portainer > Add Container:

Container Name: plex
Image: plexinc/pms-docker:latest
Ports: 32400:32400/tcp
Volumes:
  - /srv/dev-disk-by-label-USBHDD/Media:/data
  - /srv/dev-disk-by-label-USBHDD/plex-config:/config
Environment Variables:
  - PLEX_CLAIM=[Get from plex.tv/claim]
  - ADVERTISE_IP=http://[PI_IP]:32400
```

#### 4.2 Plex Configuration
```bash
# Access Plex at http://[PI_IP]:32400
1. Sign in with Plex account
2. Add Libraries:
   - Movies: /data/Movies
   - TV Shows: /data/TV
   - Music: /data/Music
3. Configure transcoding:
   - Use hardware acceleration if available
   - Limit simultaneous transcodes to 2
4. Enable offline sync for mobile apps
```

### Phase 5: File Synchronization

#### 5.1 Syncthing Installation
```bash
# Install Syncthing
curl -s https://syncthing.net/release-key.txt | sudo apt-key add -
echo "deb https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list
sudo apt update
sudo apt install syncthing

# Create systemd service for pi user
sudo systemctl enable syncthing@pi.service
sudo systemctl start syncthing@pi.service
```

#### 5.2 Syncthing Configuration
```bash
# Access Syncthing web interface
http://[PI_IP]:8384

# Configuration:
1. Add remote device (home NAS/computer) using device ID
2. Share folder: /srv/dev-disk-by-label-USBHDD/Sync
3. Configure as "Send Only" from travel Pi
4. Set bandwidth limits for mobile connections:
   - Upload: 1 Mbps during day, unlimited at night
   - Download: 5 Mbps
5. Enable folder watching for real-time sync
```

### Phase 6: Network Security & Monitoring

#### 6.1 Pi-hole DNS Filtering
```bash
# Install Pi-hole
curl -sSL https://install.pi-hole.net | bash

# Configuration during setup:
1. Choose eth0 as interface
2. Set upstream DNS to Cloudflare (1.1.1.1)
3. Install web admin interface
4. Log queries for monitoring

# Configure router to use Pi-hole:
1. Set primary DNS to Pi IP address
2. Set secondary DNS to 8.8.8.8 (backup)
```

#### 6.2 VPN Client Setup
```bash
# Install OpenVPN
sudo apt install openvpn

# Configure VPN (example with ExpressVPN)
sudo mkdir /etc/openvpn/client
# Copy VPN provider config files to /etc/openvpn/client/
sudo systemctl enable openvpn-client@[config-name]

# Create routing rules for selective VPN usage
# Only route specific traffic through VPN while keeping local network accessible
```

#### 6.3 System Monitoring
```bash
# Install Netdata for system monitoring
bash <(curl -Ss https://my-netdata.io/kickstart.sh)

# Configure lightweight monitoring
sudo nano /etc/netdata/netdata.conf
# Reduce data retention for Raspberry Pi
# history = 3600  # 1 hour instead of default
```

### Phase 7: Content Management & Optimization

#### 7.1 Media Preparation Scripts
```bash
# Create media organization script
sudo nano /usr/local/bin/organize-media.sh

#!/bin/bash
MEDIA_DIR="/srv/dev-disk-by-label-USBHDD/Media"
USB_MOUNT="/media/usb-import"

# Function to organize movies
organize_movies() {
    find "$USB_MOUNT" -name "*.mp4" -o -name "*.mkv" -o -name "*.avi" | while read file; do
        mv "$file" "$MEDIA_DIR/Movies/"
    done
}

# Function to organize TV shows
organize_tv() {
    find "$USB_MOUNT" -path "*/TV/*" -name "*.mp4" -o -name "*.mkv" | while read file; do
        mv "$file" "$MEDIA_DIR/TV/"
    done
}

organize_movies
organize_tv
sudo plex_scan_library.sh

sudo chmod +x /usr/local/bin/organize-media.sh
```

#### 7.2 Automated Backup & Sync
```bash
# Create daily sync script
sudo nano /usr/local/bin/daily-sync.sh

#!/bin/bash
LOG_FILE="/var/log/travel-sync.log"
TRAVEL_DIR="/srv/dev-disk-by-label-USBHDD/Travel-Files"

# Create daily folder
TODAY=$(date +%Y-%m-%d)
mkdir -p "$TRAVEL_DIR/$TODAY"

# Log sync status
echo "$(date): Starting daily sync" >> $LOG_FILE

# Force Syncthing to sync immediately
curl -X POST -H "X-API-Key: $(cat ~/.config/syncthing/config.xml | grep '<apikey>' | sed 's/.*<apikey>\(.*\)<\/apikey>.*/\1/')" \
    http://localhost:8384/rest/db/scan?folder=travel-sync

echo "$(date): Daily sync completed" >> $LOG_FILE

# Add to crontab
echo "0 2 * * * /usr/local/bin/daily-sync.sh" | sudo crontab -
```

### Phase 8: Travel Deployment

#### 8.1 Pre-Travel Checklist
```bash
# System health check script
sudo nano /usr/local/bin/travel-check.sh

#!/bin/bash
echo "=== Travel System Health Check ==="

# Check disk space
df -h | grep -E "(Filesystem|/dev/sd|/srv)"

# Check services
systemctl is-active plex syncthing@pi pihole-FTL openvpn-client@*

# Check network connectivity
ping -c 3 8.8.8.8
ping -c 3 google.com

# Check Plex library
curl -s http://localhost:32400/status/sessions | grep -o '"size":[0-9]*'

# Check available storage for new content
AVAILABLE=$(df /srv/dev-disk-by-label-USBHDD | tail -1 | awk '{print $4}')
echo "Available storage: $(($AVAILABLE / 1024 / 1024)) GB"

echo "=== Health Check Complete ==="
```

#### 8.2 Travel Network Setup Procedures

##### Hotel/Airbnb Deployment
```bash
# Quick setup procedure:
1. Connect Netgear router to venue internet (ethernet preferred)
2. Power on Raspberry Pi and wait 3 minutes for full boot
3. Connect laptop to FamilyTravel WiFi network
4. Verify Pi accessibility: ping [PI_IP]
5. Test Plex access: http://[PI_IP]:32400
6. Check Pi-hole status: http://[PI_IP]/admin
7. Connect family devices to WiFi network

# Troubleshooting commands:
ssh pi@[PI_IP]
sudo systemctl status plex syncthing@pi pihole-FTL
tail -f /var/log/syslog
```

##### Airplane Mode Setup
```bash
# For flight usage:
1. Connect Pi directly to router via ethernet
2. Disable router WiFi broadcasting (airplane compliance)
3. Connect devices to router via ethernet hub if needed
4. Access Plex and local content without internet
5. Use portable power bank rated for 8+ hours operation
```

## Advanced Configurations

### Performance Optimization for Raspberry Pi 3B+

#### 8.3 System Tuning
```bash
# Optimize for media serving
sudo nano /boot/config.txt

# Add these lines for better performance:
gpu_mem=128
dtoverlay=pi3-disable-wifi  # If using ethernet only
arm_freq=1400
core_freq=400
sdram_freq=500
over_voltage=2

# Configure swap file
sudo dphys-swapfile swapoff
sudo nano /etc/dphys-swapfile
# CONF_SWAPSIZE=1024
sudo dphys-swapfile setup
sudo dphys-swapfile swapon
```

#### 8.4 USB Storage Optimization
```bash
# Mount USB drive with optimizations
sudo nano /etc/fstab

# Add line for USB drive:
UUID=[USB_DRIVE_UUID] /srv/dev-disk-by-label-USBHDD ext4 defaults,noatime,nofail 0 0

# Enable USB 3.0 performance
echo 'dwc_otg.speed=1' | sudo tee -a /boot/cmdline.txt
```

## Troubleshooting Guide

### Common Issues & Solutions

#### Network Connectivity Problems
```bash
# Router not providing internet
1. Check physical ethernet connection
2. Restart router (unplug 30 seconds)
3. Check router status lights
4. Verify ISP/venue internet is working
5. Reset router to factory defaults if needed

# Pi not accessible on network
1. Check Pi power LED (steady red = good)
2. Check Pi activity LED (flashing green = booting)
3. Verify ethernet connection to router
4. Check router DHCP client list for Pi
5. SSH with direct IP: ssh pi@192.168.1.XXX
```

#### Media Streaming Issues
```bash
# Plex not working
1. Check Plex service: sudo systemctl status plexmediaserver
2. Restart Plex: sudo systemctl restart plexmediaserver
3. Check available RAM: free -h
4. Reduce simultaneous streams (Pi 3B+ limit: 2 transcodes)
5. Use direct play instead of transcoding when possible

# Storage full or slow
1. Check disk space: df -h
2. Clean Plex transcoding cache: rm -rf /var/lib/plexmediaserver/Library/Application\ Support/Plex\ Media\ Server/Cache/
3. Move old content to backup storage
4. Use lower quality video files for mobile viewing
```

#### Synchronization Problems
```bash
# Syncthing not syncing
1. Check Syncthing status: systemctl status syncthing@pi
2. Verify internet connectivity: ping 8.8.8.8
3. Check Syncthing web UI for errors
4. Verify remote device connectivity
5. Check available storage space
6. Restart Syncthing: systemctl restart syncthing@pi
```

## Security Considerations

### Travel Security Best Practices
```bash
# Change all default passwords
passwd  # Pi user password
# Router admin password via web interface
# Pi-hole admin password: pihole -a -p

# Disable unnecessary services
sudo systemctl disable bluetooth
sudo systemctl disable wifi-country
sudo systemctl disable triggerhappy

# Configure firewall
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 32400  # Plex
sudo ufw allow 8384   # Syncthing
sudo ufw allow 80     # Pi-hole
sudo ufw enable
```

### Data Protection
```bash
# Enable automatic security updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Backup critical configurations
mkdir ~/config-backup
cp /etc/fstab ~/config-backup/
cp /boot/config.txt ~/config-backup/
cp ~/.config/syncthing/config.xml ~/config-backup/
tar -czf travel-config-backup.tar.gz ~/config-backup/
```

## Bill of Materials & Cost Analysis

### Core Setup (~$200-250)
- Raspberry Pi 3 Model B+: $35
- Netgear R6230 Router: $60
- 64GB MicroSD Card (U3): $15
- 2TB USB 3.0 Hard Drive: $60
- 5V 2.5A Power Supply: $10
- Ethernet Cable: $5
- Case with Fan: $15
- Powered USB Hub: $20
- Portable Power Bank (20000mAh): $30

### Optional Upgrades (~$50-100)
- Additional USB storage: $30
- USB WiFi adapter: $15
- Cooling accessories: $10
- Carrying case: $25
- Extra cables and adapters: $20

### Cost Comparison
- **Pi 3B+ Setup**: ~$250 total
- **ZimaBoard Setup**: ~$500 total
- **Commercial Travel Router**: $300-500
- **Separate NAS Device**: $200-400

## Performance Expectations

### Raspberry Pi 3B+ Limitations
- **Concurrent Plex Streams**: 2-3 direct play, 1-2 with transcoding
- **Network Throughput**: ~300Mbps wired, ~150Mbps wireless
- **Storage Speed**: Limited by USB 2.0 (~35MB/s)
- **RAM**: 1GB shared with GPU (consider lightweight services)
- **CPU**: Adequate for basic NAS tasks, limited transcoding

### Optimization Tips
- Use H.264/H.265 video files for better compatibility
- Enable hardware acceleration where possible
- Prioritize direct play over transcoding
- Use wired connections when possible
- Monitor system resources regularly

## Resources and References

### Official Documentation
- [Raspberry Pi Documentation](https://www.raspberrypi.org/documentation/)
- [OpenMediaVault Docs](https://openmediavault.readthedocs.io/)
- [Plex Media Server Setup](https://support.plex.tv/articles/200288586-installation/)
- [Syncthing Documentation](https://docs.syncthing.net/)
- [Pi-hole Documentation](https://docs.pi-hole.net/)

### Community Resources
- r/raspberry_pi subreddit
- Plex Community Forums
- OpenMediaVault Community Forum
- Pi-hole Community Discourse

### Recommended YouTube Channels
- ExplainingComputers (Pi tutorials)
- Techno Tim (Self-hosted services)
- Jeff Geerling (Pi performance optimization)

---

## Support and Troubleshooting

For issues or questions:
1. Check the troubleshooting section above
2. Search community forums
3. Check system logs: `sudo journalctl -f`
4. Monitor system resources: `htop`
5. Test network connectivity: `ping`, `nslookup`

**Remember**: The Raspberry Pi 3B+ is a capable but resource-limited device. Manage expectations and optimize for your specific use case!

**Happy travels and streaming!** 🌏📱🥧
