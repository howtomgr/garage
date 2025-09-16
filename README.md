# garage Installation Guide

garage is a free and open-source S3-compatible storage. Garage provides self-hosted S3-compatible distributed object storage

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 100GB+ for data
  - Network: S3 API
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3900 (default garage port)
  - Admin on 3902
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install garage
sudo dnf install -y garage

# Enable and start service
sudo systemctl enable --now garage

# Configure firewall
sudo firewall-cmd --permanent --add-port=3900/tcp
sudo firewall-cmd --reload

# Verify installation
garage --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install garage
sudo apt install -y garage

# Enable and start service
sudo systemctl enable --now garage

# Configure firewall
sudo ufw allow 3900

# Verify installation
garage --version
```

### Arch Linux

```bash
# Install garage
sudo pacman -S garage

# Enable and start service
sudo systemctl enable --now garage

# Verify installation
garage --version
```

### Alpine Linux

```bash
# Install garage
apk add --no-cache garage

# Enable and start service
rc-update add garage default
rc-service garage start

# Verify installation
garage --version
```

### openSUSE/SLES

```bash
# Install garage
sudo zypper install -y garage

# Enable and start service
sudo systemctl enable --now garage

# Configure firewall
sudo firewall-cmd --permanent --add-port=3900/tcp
sudo firewall-cmd --reload

# Verify installation
garage --version
```

### macOS

```bash
# Using Homebrew
brew install garage

# Start service
brew services start garage

# Verify installation
garage --version
```

### FreeBSD

```bash
# Using pkg
pkg install garage

# Enable in rc.conf
echo 'garage_enable="YES"' >> /etc/rc.conf

# Start service
service garage start

# Verify installation
garage --version
```

### Windows

```bash
# Using Chocolatey
choco install garage

# Or using Scoop
scoop install garage

# Verify installation
garage --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/garage

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
garage --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable garage

# Start service
sudo systemctl start garage

# Stop service
sudo systemctl stop garage

# Restart service
sudo systemctl restart garage

# Check status
sudo systemctl status garage

# View logs
sudo journalctl -u garage -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add garage default

# Start service
rc-service garage start

# Stop service
rc-service garage stop

# Restart service
rc-service garage restart

# Check status
rc-service garage status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'garage_enable="YES"' >> /etc/rc.conf

# Start service
service garage start

# Stop service
service garage stop

# Restart service
service garage restart

# Check status
service garage status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start garage
brew services stop garage
brew services restart garage

# Check status
brew services list | grep garage
```

### Windows Service Manager

```powershell
# Start service
net start garage

# Stop service
net stop garage

# Using PowerShell
Start-Service garage
Stop-Service garage
Restart-Service garage

# Check status
Get-Service garage
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream garage_backend {
    server 127.0.0.1:3900;
}

server {
    listen 80;
    server_name garage.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name garage.example.com;

    ssl_certificate /etc/ssl/certs/garage.example.com.crt;
    ssl_certificate_key /etc/ssl/private/garage.example.com.key;

    location / {
        proxy_pass http://garage_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName garage.example.com
    Redirect permanent / https://garage.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName garage.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/garage.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/garage.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3900/
    ProxyPassReverse / http://127.0.0.1:3900/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend garage_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/garage.pem
    redirect scheme https if !{ ssl_fc }
    default_backend garage_backend

backend garage_backend
    balance roundrobin
    server garage1 127.0.0.1:3900 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R garage:garage /etc/garage
sudo chmod 750 /etc/garage

# Configure firewall
sudo firewall-cmd --permanent --add-port=3900/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status garage

# View logs
sudo journalctl -u garage -f

# Monitor resource usage
top -p $(pgrep garage)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/garage"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/garage-backup-$DATE.tar.gz" /etc/garage /var/lib/garage

echo "Backup completed: $BACKUP_DIR/garage-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop garage

# Restore from backup
tar -xzf /backup/garage/garage-backup-*.tar.gz -C /

# Start service
sudo systemctl start garage
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u garage -n 100
sudo tail -f /var/log/garage/garage.log

# Check configuration
garage --version

# Check permissions
ls -la /etc/garage
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3900

# Test connectivity
telnet localhost 3900

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep garage)

# Check disk I/O
iotop -p $(pgrep garage)

# Check connections
ss -an | grep 3900
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  garage:
    image: garage:latest
    ports:
      - "3900:3900"
    volumes:
      - ./config:/etc/garage
      - ./data:/var/lib/garage
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update garage

# Debian/Ubuntu
sudo apt update && sudo apt upgrade garage

# Arch Linux
sudo pacman -Syu garage

# Alpine Linux
apk update && apk upgrade garage

# openSUSE
sudo zypper update garage

# FreeBSD
pkg update && pkg upgrade garage

# Always backup before updates
tar -czf /backup/garage-pre-update-$(date +%Y%m%d).tar.gz /etc/garage

# Restart after updates
sudo systemctl restart garage
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/garage

# Clean old logs
find /var/log/garage -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/garage
```

## Additional Resources

- Official Documentation: https://docs.garage.org/
- GitHub Repository: https://github.com/garage/garage
- Community Forum: https://forum.garage.org/
- Best Practices Guide: https://docs.garage.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
