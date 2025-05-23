# Travel Router & NAS Setup Guide

## Overview
This guide provides step-by-step instructions for building a portable travel router and NAS system using a ZimaBoard (or similar x86 mini PC) and GL.iNet travel router. Perfect for families traveling with multiple devices requiring reliable internet access and media streaming capabilities.

## Hardware Requirements

### Core Components
- **ZimaBoard 832** (or similar x86 mini PC)
  - Intel Celeron quad-core processor
  - 8GB RAM minimum
  - 2x SATA ports
  - 2x Ethernet ports
  - PCIe slot
  - USB ports
- **GL.iNet Travel Router** (Beryl AX or similar)
- **2x 2TB SSDs** (for storage redundancy)
- **SATA Y-connector cable** (power splitter)
- **Ethernet cable** (short, for router-to-NAS connection)
- **External USB drive** (for data transfer)
- **Portable power bank** (for flight usage)

### Optional Components
- **WiFi 6 PCIe card** (if enhanced wireless needed)
- **Additional USB hub** (for more ports)
- **3D printed case** (for integrated housing)

## Software Stack
- **CasaOS** (Primary NAS OS)
- **Plex Media Server** (Media streaming)
- **Syncthing** (File synchronization)
- **Pi-hole** (DNS filtering and ad blocking)
- **Uptime Kuma** (Network monitoring)

## Step-by-Step Setup

### Phase 1: Hardware Assembly

#### 1.1 ZimaBoard Preparation
```bash
# Connect SSDs via SATA Y-connector
1. Insert SATA Y-connector into ZimaBoard SATA ports
2. Connect both 2TB SSDs to Y-connector
3. Ensure secure connections and proper seating
```

#### 1.2 Initial Boot and OS Installation
```bash
# Flash CasaOS to USB drive
1. Download CasaOS image from official website
2. Use Rufus/Etcher to flash image to USB drive
3. Boot ZimaBoard from USB
4. Follow installation wizard
5. Set admin credentials
6. Configure network settings
```

### Phase 2: Network Configuration

#### 2.1 GL.iNet Router Setup
```bash
# Router initial configuration
1. Connect GL.iNet router to power
2. Access admin panel at 192.168.8.1
3. Set admin password
4. Configure WiFi SSID: "Let's Just Go Fam"
5. Set WiFi password
6. Enable client mode for upstream connections
```

#### 2.2 Network Topology Setup
```bash
# Physical connections
1. Connect Ethernet cable from GL.iNet LAN to ZimaBoard
2. Set DHCP reservation for ZimaBoard: 10.19.7.19
3. Configure DNS to point to ZimaBoard (Pi-hole)
```

### Phase 3: CasaOS Configuration

#### 3.1 Storage Setup
```bash
# Access CasaOS web interface
http://10.19.7.19:80

# Storage configuration
1. Navigate to Files app
2. Identify connected SSDs
3. Label drives:
   - Drive 1: "Plex_Media"
   - Drive 2: "Travel_NAS"
4. Create folder structure:
   - /Plex_Media/Movies
   - /Plex_Media/TV_Shows
   - /Travel_NAS/Daily_Footage/[DATE]
```

#### 3.2 App Installation
```bash
# Install core applications via CasaOS App Store
1. Plex Media Server
2. Syncthing
3. Pi-hole
4. Uptime Kuma
5. File Browser (if not default)
```

### Phase 4: Service Configuration

#### 4.1 Plex Media Server Setup
```bash
# Plex configuration
1. Access Plex at http://10.19.7.19:32400
2. Sign in with Plex account
3. Add libraries:
   - Movies: /Plex_Media/Movies
   - TV Shows: /Plex_Media/TV_Shows
4. Configure remote access (optional)
5. Set transcoding quality for mobile devices
```

#### 4.2 Syncthing Configuration
```bash
# Syncthing setup for remote sync
1. Access Syncthing web UI
2. Add remote device (home studio NAS)
   - Actions > Show ID (copy device ID)
   - Add to home NAS Syncthing instance
3. Configure shared folders:
   - /Travel_NAS/Daily_Footage (send only)
4. Set sync conditions:
   - Bandwidth limits for mobile connections
   - Schedule for overnight syncing
```

#### 4.3 Pi-hole DNS Configuration
```bash
# Pi-hole setup
1. Access Pi-hole admin at http://10.19.7.19/admin
2. Configure upstream DNS servers:
   - Primary: 1.1.1.1 (Cloudflare)
   - Secondary: 8.8.8.8 (Google)
3. Add blocklists for ad filtering
4. Configure GL.iNet router to use Pi-hole as DNS
```

#### 4.4 Monitoring Setup
```bash
# Uptime Kuma configuration
1. Access monitoring dashboard
2. Add monitors for:
   - Internet connectivity
   - Plex service health
   - Syncthing sync status
   - Storage usage alerts
```

### Phase 5: Content Preparation

#### 5.1 Media Library Setup
```bash
# Pre-travel content loading
1. Connect external USB drive with content
2. Copy movies to /Plex_Media/Movies
3. Copy TV shows to /Plex_Media/TV_Shows
4. Ensure proper file naming for Plex recognition
5. Refresh Plex libraries
```

#### 5.2 Backup Strategy
```bash
# Configure automated backups
1. Set up Syncthing rules for critical configs
2. Export CasaOS settings
3. Document network configurations
4. Create recovery USB with essential tools
```

### Phase 6: Field Testing

#### 6.1 Home Network Testing
```bash
# Pre-departure testing
1. Test all services accessibility
2. Verify media streaming on multiple devices
3. Check file upload/sync functionality
4. Test network performance under load
5. Validate backup and recovery procedures
```

#### 6.2 Travel Deployment Scenarios

##### Hotel/Airbnb Setup
```bash
# Wired connection (preferred)
1. Locate router/modem ethernet port
2. Connect GL.iNet router via ethernet
3. Power on ZimaBoard
4. Wait for services to initialize (2-3 minutes)
5. Connect devices to "Let's Just Go Fam" WiFi

# Wireless connection (fallback)
1. Configure GL.iNet to connect to venue WiFi
2. Enable repeater mode
3. Follow same power-on sequence
```

##### Airplane Setup
```bash
# In-flight configuration
1. Store ZimaBoard under seat
2. Connect to portable power bank
3. Ensure airplane mode compliance (no WiFi broadcast)
4. Access media via direct ethernet connection to router
5. Distribute content via router's offline mode
```

## Troubleshooting Guide

### Common Issues

#### Network Connectivity Problems
```bash
# Router not connecting to internet
1. Check physical connections
2. Verify venue WiFi credentials
3. Restart router (hold reset 10 seconds)
4. Check DHCP conflicts
5. Update router firmware if needed
```

#### Plex Media Access Issues
```bash
# Plex server not accessible
1. Verify ZimaBoard is powered and booted
2. Check network connectivity to 10.19.7.19
3. Restart Plex service in CasaOS
4. Clear Plex app cache on client devices
5. Check storage drive mounting
```

#### Syncthing Sync Problems
```bash
# Files not syncing to home
1. Verify internet connectivity
2. Check Syncthing service status
3. Confirm remote device connectivity
4. Review bandwidth limitations
5. Check available storage space
```

### Performance Optimization

#### Network Performance
```bash
# Optimize for travel scenarios
1. Adjust Plex transcoding settings for mobile
2. Configure Syncthing bandwidth limits
3. Set Pi-hole cache settings for frequent queries
4. Enable QoS on router for media priority
```

#### Storage Management
```bash
# Manage limited storage effectively
1. Regular cleanup of processed footage
2. Compress media files for mobile viewing
3. Use symbolic links for large libraries
4. Monitor storage usage via dashboard
```

## Security Considerations

### Network Security
```bash
# Essential security measures
1. Change default passwords on all services
2. Enable WPA3 on WiFi networks
3. Configure firewall rules on router
4. Use VPN for sensitive traffic
5. Regular security updates
```

### Data Protection
```bash
# Protect valuable content
1. Enable filesystem encryption on SSDs
2. Secure physical access to devices
3. Regular backup verification
4. Implement access logging
5. Use secure sync protocols (HTTPS/TLS)
```

## Maintenance Schedule

### Weekly Tasks
- Monitor storage usage
- Check sync status and resolve conflicts
- Update media libraries
- Review network performance logs

### Monthly Tasks
- Update CasaOS and applications
- Clean temporary files and logs
- Verify backup integrity
- Review and update blocklists

### Pre-Travel Tasks
- Full system backup
- Update all software components
- Test all functionality end-to-end
- Prepare troubleshooting documentation
- Pack necessary cables and adapters

## Advanced Configurations

### Custom Docker Containers
```bash
# Add specialized services
1. Nextcloud for file sharing
2. Jellyfin as Plex alternative
3. Transmission for downloads
4. Nginx proxy for SSL termination
```

### Network Monitoring
```bash
# Enhanced monitoring setup
1. SNMP monitoring for router
2. Bandwidth usage tracking
3. Connection quality metrics
4. Automated alerting setup
```

## Bill of Materials

### Core Setup (~$400-500)
- ZimaBoard 832: $200
- GL.iNet Beryl AX: $80
- 2x 2TB SSDs: $160
- Cables and accessories: $40

### Optional Upgrades (~$100-200)
- WiFi 6 PCIe card: $50
- Portable power bank: $60
- 3D printed case: $30
- Additional storage: $100

## Resources and References

### Official Documentation
- [CasaOS Documentation](https://casaos.io/docs)
- [GL.iNet Router Setup](https://docs.gl-inet.com)
- [Plex Media Server Guide](https://support.plex.tv)
- [Syncthing Documentation](https://docs.syncthing.net)

### Community Resources
- ZimaBoard Community Forums
- r/homelab subreddit
- Plex Community Forums
- GL.iNet Community

### Video Tutorials
- Original setup walkthrough
- Troubleshooting common issues
- Advanced configuration examples

---

## Support and Contributing

For issues, suggestions, or contributions to this guide, please:
1. Check existing documentation
2. Search community forums
3. Create detailed issue reports
4. Share successful configurations
5. Contribute improvements via pull requests

**Happy travels and streaming!** 🌏📱