# rancher Installation Guide

rancher is a free and open-source multi-cluster Kubernetes management platform. Rancher simplifies Kubernetes cluster deployment and management across any infrastructure, serving as an open-source alternative to proprietary Kubernetes platforms

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
  - CPU: 2+ cores recommended
  - RAM: 4GB minimum (8GB+ recommended)
  - Storage: 20GB for management server
  - Network: HTTPS access to clusters
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 443 (default rancher port)
  - Port 80 for HTTP redirect
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

# Install rancher
sudo dnf install -y rancher

# Enable and start service
sudo systemctl enable --now rancher

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload

# Verify installation
rancher --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install rancher
sudo apt install -y rancher

# Enable and start service
sudo systemctl enable --now rancher

# Configure firewall
sudo ufw allow 443

# Verify installation
rancher --version
```

### Arch Linux

```bash
# Install rancher
sudo pacman -S rancher

# Enable and start service
sudo systemctl enable --now rancher

# Verify installation
rancher --version
```

### Alpine Linux

```bash
# Install rancher
apk add --no-cache rancher

# Enable and start service
rc-update add rancher default
rc-service rancher start

# Verify installation
rancher --version
```

### openSUSE/SLES

```bash
# Install rancher
sudo zypper install -y rancher

# Enable and start service
sudo systemctl enable --now rancher

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload

# Verify installation
rancher --version
```

### macOS

```bash
# Using Homebrew
brew install rancher

# Start service
brew services start rancher

# Verify installation
rancher --version
```

### FreeBSD

```bash
# Using pkg
pkg install rancher

# Enable in rc.conf
echo 'rancher_enable="YES"' >> /etc/rc.conf

# Start service
service rancher start

# Verify installation
rancher --version
```

### Windows

```bash
# Using Chocolatey
choco install rancher

# Or using Scoop
scoop install rancher

# Verify installation
rancher --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/rancher

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
rancher --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable rancher

# Start service
sudo systemctl start rancher

# Stop service
sudo systemctl stop rancher

# Restart service
sudo systemctl restart rancher

# Check status
sudo systemctl status rancher

# View logs
sudo journalctl -u rancher -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add rancher default

# Start service
rc-service rancher start

# Stop service
rc-service rancher stop

# Restart service
rc-service rancher restart

# Check status
rc-service rancher status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'rancher_enable="YES"' >> /etc/rc.conf

# Start service
service rancher start

# Stop service
service rancher stop

# Restart service
service rancher restart

# Check status
service rancher status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start rancher
brew services stop rancher
brew services restart rancher

# Check status
brew services list | grep rancher
```

### Windows Service Manager

```powershell
# Start service
net start rancher

# Stop service
net stop rancher

# Using PowerShell
Start-Service rancher
Stop-Service rancher
Restart-Service rancher

# Check status
Get-Service rancher
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream rancher_backend {
    server 127.0.0.1:443;
}

server {
    listen 80;
    server_name rancher.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name rancher.example.com;

    ssl_certificate /etc/ssl/certs/rancher.example.com.crt;
    ssl_certificate_key /etc/ssl/private/rancher.example.com.key;

    location / {
        proxy_pass http://rancher_backend;
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
    ServerName rancher.example.com
    Redirect permanent / https://rancher.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName rancher.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/rancher.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/rancher.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:443/
    ProxyPassReverse / http://127.0.0.1:443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend rancher_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/rancher.pem
    redirect scheme https if !{ ssl_fc }
    default_backend rancher_backend

backend rancher_backend
    balance roundrobin
    server rancher1 127.0.0.1:443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R rancher:rancher /etc/rancher
sudo chmod 750 /etc/rancher

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
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
sudo systemctl status rancher

# View logs
sudo journalctl -u rancher -f

# Monitor resource usage
top -p $(pgrep rancher)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/rancher"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/rancher-backup-$DATE.tar.gz" /etc/rancher /var/lib/rancher

echo "Backup completed: $BACKUP_DIR/rancher-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop rancher

# Restore from backup
tar -xzf /backup/rancher/rancher-backup-*.tar.gz -C /

# Start service
sudo systemctl start rancher
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u rancher -n 100
sudo tail -f /var/log/rancher/rancher.log

# Check configuration
rancher --version

# Check permissions
ls -la /etc/rancher
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 443

# Test connectivity
telnet localhost 443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep rancher)

# Check disk I/O
iotop -p $(pgrep rancher)

# Check connections
ss -an | grep 443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  rancher:
    image: rancher:latest
    ports:
      - "443:443"
    volumes:
      - ./config:/etc/rancher
      - ./data:/var/lib/rancher
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update rancher

# Debian/Ubuntu
sudo apt update && sudo apt upgrade rancher

# Arch Linux
sudo pacman -Syu rancher

# Alpine Linux
apk update && apk upgrade rancher

# openSUSE
sudo zypper update rancher

# FreeBSD
pkg update && pkg upgrade rancher

# Always backup before updates
tar -czf /backup/rancher-pre-update-$(date +%Y%m%d).tar.gz /etc/rancher

# Restart after updates
sudo systemctl restart rancher
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/rancher

# Clean old logs
find /var/log/rancher -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/rancher
```

## Additional Resources

- Official Documentation: https://docs.rancher.org/
- GitHub Repository: https://github.com/rancher/rancher
- Community Forum: https://forum.rancher.org/
- Best Practices Guide: https://docs.rancher.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
