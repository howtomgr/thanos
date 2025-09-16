# thanos Installation Guide

thanos is a free and open-source highly available Prometheus setup with long term storage. Thanos provides unlimited storage capacity for Prometheus metrics with global query view, serving as an open-source alternative to Cortex or commercial monitoring solutions

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
  - CPU: 2+ cores per component
  - RAM: 2GB minimum per component
  - Storage: 10GB+ for local cache
  - Network: gRPC between components
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 10901 (default thanos port)
  - Ports 10902 (gRPC), 10903 (HTTP)
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

# Install thanos
sudo dnf install -y thanos

# Enable and start service
sudo systemctl enable --now thanos

# Configure firewall
sudo firewall-cmd --permanent --add-port=10901/tcp
sudo firewall-cmd --reload

# Verify installation
thanos --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install thanos
sudo apt install -y thanos

# Enable and start service
sudo systemctl enable --now thanos

# Configure firewall
sudo ufw allow 10901

# Verify installation
thanos --version
```

### Arch Linux

```bash
# Install thanos
sudo pacman -S thanos

# Enable and start service
sudo systemctl enable --now thanos

# Verify installation
thanos --version
```

### Alpine Linux

```bash
# Install thanos
apk add --no-cache thanos

# Enable and start service
rc-update add thanos default
rc-service thanos start

# Verify installation
thanos --version
```

### openSUSE/SLES

```bash
# Install thanos
sudo zypper install -y thanos

# Enable and start service
sudo systemctl enable --now thanos

# Configure firewall
sudo firewall-cmd --permanent --add-port=10901/tcp
sudo firewall-cmd --reload

# Verify installation
thanos --version
```

### macOS

```bash
# Using Homebrew
brew install thanos

# Start service
brew services start thanos

# Verify installation
thanos --version
```

### FreeBSD

```bash
# Using pkg
pkg install thanos

# Enable in rc.conf
echo 'thanos_enable="YES"' >> /etc/rc.conf

# Start service
service thanos start

# Verify installation
thanos --version
```

### Windows

```bash
# Using Chocolatey
choco install thanos

# Or using Scoop
scoop install thanos

# Verify installation
thanos --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/thanos

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
thanos --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable thanos

# Start service
sudo systemctl start thanos

# Stop service
sudo systemctl stop thanos

# Restart service
sudo systemctl restart thanos

# Check status
sudo systemctl status thanos

# View logs
sudo journalctl -u thanos -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add thanos default

# Start service
rc-service thanos start

# Stop service
rc-service thanos stop

# Restart service
rc-service thanos restart

# Check status
rc-service thanos status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'thanos_enable="YES"' >> /etc/rc.conf

# Start service
service thanos start

# Stop service
service thanos stop

# Restart service
service thanos restart

# Check status
service thanos status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start thanos
brew services stop thanos
brew services restart thanos

# Check status
brew services list | grep thanos
```

### Windows Service Manager

```powershell
# Start service
net start thanos

# Stop service
net stop thanos

# Using PowerShell
Start-Service thanos
Stop-Service thanos
Restart-Service thanos

# Check status
Get-Service thanos
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream thanos_backend {
    server 127.0.0.1:10901;
}

server {
    listen 80;
    server_name thanos.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name thanos.example.com;

    ssl_certificate /etc/ssl/certs/thanos.example.com.crt;
    ssl_certificate_key /etc/ssl/private/thanos.example.com.key;

    location / {
        proxy_pass http://thanos_backend;
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
    ServerName thanos.example.com
    Redirect permanent / https://thanos.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName thanos.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/thanos.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/thanos.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:10901/
    ProxyPassReverse / http://127.0.0.1:10901/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend thanos_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/thanos.pem
    redirect scheme https if !{ ssl_fc }
    default_backend thanos_backend

backend thanos_backend
    balance roundrobin
    server thanos1 127.0.0.1:10901 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R thanos:thanos /etc/thanos
sudo chmod 750 /etc/thanos

# Configure firewall
sudo firewall-cmd --permanent --add-port=10901/tcp
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
sudo systemctl status thanos

# View logs
sudo journalctl -u thanos -f

# Monitor resource usage
top -p $(pgrep thanos)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/thanos"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/thanos-backup-$DATE.tar.gz" /etc/thanos /var/lib/thanos

echo "Backup completed: $BACKUP_DIR/thanos-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop thanos

# Restore from backup
tar -xzf /backup/thanos/thanos-backup-*.tar.gz -C /

# Start service
sudo systemctl start thanos
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u thanos -n 100
sudo tail -f /var/log/thanos/thanos.log

# Check configuration
thanos --version

# Check permissions
ls -la /etc/thanos
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 10901

# Test connectivity
telnet localhost 10901

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep thanos)

# Check disk I/O
iotop -p $(pgrep thanos)

# Check connections
ss -an | grep 10901
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  thanos:
    image: thanos:latest
    ports:
      - "10901:10901"
    volumes:
      - ./config:/etc/thanos
      - ./data:/var/lib/thanos
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update thanos

# Debian/Ubuntu
sudo apt update && sudo apt upgrade thanos

# Arch Linux
sudo pacman -Syu thanos

# Alpine Linux
apk update && apk upgrade thanos

# openSUSE
sudo zypper update thanos

# FreeBSD
pkg update && pkg upgrade thanos

# Always backup before updates
tar -czf /backup/thanos-pre-update-$(date +%Y%m%d).tar.gz /etc/thanos

# Restart after updates
sudo systemctl restart thanos
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/thanos

# Clean old logs
find /var/log/thanos -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/thanos
```

## Additional Resources

- Official Documentation: https://docs.thanos.org/
- GitHub Repository: https://github.com/thanos/thanos
- Community Forum: https://forum.thanos.org/
- Best Practices Guide: https://docs.thanos.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
