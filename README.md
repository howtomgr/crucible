# crucible Installation Guide

crucible is a free and open-source code review tool. Crucible provides code review tool integrated with various VCS

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
  - RAM: 4GB minimum
  - Storage: 10GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8060 (default crucible port)
  - None
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

# Install crucible
sudo dnf install -y crucible

# Enable and start service
sudo systemctl enable --now crucible

# Configure firewall
sudo firewall-cmd --permanent --add-port=8060/tcp
sudo firewall-cmd --reload

# Verify installation
crucible --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install crucible
sudo apt install -y crucible

# Enable and start service
sudo systemctl enable --now crucible

# Configure firewall
sudo ufw allow 8060

# Verify installation
crucible --version
```

### Arch Linux

```bash
# Install crucible
sudo pacman -S crucible

# Enable and start service
sudo systemctl enable --now crucible

# Verify installation
crucible --version
```

### Alpine Linux

```bash
# Install crucible
apk add --no-cache crucible

# Enable and start service
rc-update add crucible default
rc-service crucible start

# Verify installation
crucible --version
```

### openSUSE/SLES

```bash
# Install crucible
sudo zypper install -y crucible

# Enable and start service
sudo systemctl enable --now crucible

# Configure firewall
sudo firewall-cmd --permanent --add-port=8060/tcp
sudo firewall-cmd --reload

# Verify installation
crucible --version
```

### macOS

```bash
# Using Homebrew
brew install crucible

# Start service
brew services start crucible

# Verify installation
crucible --version
```

### FreeBSD

```bash
# Using pkg
pkg install crucible

# Enable in rc.conf
echo 'crucible_enable="YES"' >> /etc/rc.conf

# Start service
service crucible start

# Verify installation
crucible --version
```

### Windows

```bash
# Using Chocolatey
choco install crucible

# Or using Scoop
scoop install crucible

# Verify installation
crucible --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/crucible

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
crucible --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable crucible

# Start service
sudo systemctl start crucible

# Stop service
sudo systemctl stop crucible

# Restart service
sudo systemctl restart crucible

# Check status
sudo systemctl status crucible

# View logs
sudo journalctl -u crucible -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add crucible default

# Start service
rc-service crucible start

# Stop service
rc-service crucible stop

# Restart service
rc-service crucible restart

# Check status
rc-service crucible status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'crucible_enable="YES"' >> /etc/rc.conf

# Start service
service crucible start

# Stop service
service crucible stop

# Restart service
service crucible restart

# Check status
service crucible status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start crucible
brew services stop crucible
brew services restart crucible

# Check status
brew services list | grep crucible
```

### Windows Service Manager

```powershell
# Start service
net start crucible

# Stop service
net stop crucible

# Using PowerShell
Start-Service crucible
Stop-Service crucible
Restart-Service crucible

# Check status
Get-Service crucible
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream crucible_backend {
    server 127.0.0.1:8060;
}

server {
    listen 80;
    server_name crucible.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name crucible.example.com;

    ssl_certificate /etc/ssl/certs/crucible.example.com.crt;
    ssl_certificate_key /etc/ssl/private/crucible.example.com.key;

    location / {
        proxy_pass http://crucible_backend;
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
    ServerName crucible.example.com
    Redirect permanent / https://crucible.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName crucible.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/crucible.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/crucible.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8060/
    ProxyPassReverse / http://127.0.0.1:8060/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend crucible_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/crucible.pem
    redirect scheme https if !{ ssl_fc }
    default_backend crucible_backend

backend crucible_backend
    balance roundrobin
    server crucible1 127.0.0.1:8060 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R crucible:crucible /etc/crucible
sudo chmod 750 /etc/crucible

# Configure firewall
sudo firewall-cmd --permanent --add-port=8060/tcp
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
sudo systemctl status crucible

# View logs
sudo journalctl -u crucible -f

# Monitor resource usage
top -p $(pgrep crucible)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/crucible"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/crucible-backup-$DATE.tar.gz" /etc/crucible /var/lib/crucible

echo "Backup completed: $BACKUP_DIR/crucible-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop crucible

# Restore from backup
tar -xzf /backup/crucible/crucible-backup-*.tar.gz -C /

# Start service
sudo systemctl start crucible
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u crucible -n 100
sudo tail -f /var/log/crucible/crucible.log

# Check configuration
crucible --version

# Check permissions
ls -la /etc/crucible
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8060

# Test connectivity
telnet localhost 8060

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep crucible)

# Check disk I/O
iotop -p $(pgrep crucible)

# Check connections
ss -an | grep 8060
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  crucible:
    image: crucible:latest
    ports:
      - "8060:8060"
    volumes:
      - ./config:/etc/crucible
      - ./data:/var/lib/crucible
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update crucible

# Debian/Ubuntu
sudo apt update && sudo apt upgrade crucible

# Arch Linux
sudo pacman -Syu crucible

# Alpine Linux
apk update && apk upgrade crucible

# openSUSE
sudo zypper update crucible

# FreeBSD
pkg update && pkg upgrade crucible

# Always backup before updates
tar -czf /backup/crucible-pre-update-$(date +%Y%m%d).tar.gz /etc/crucible

# Restart after updates
sudo systemctl restart crucible
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/crucible

# Clean old logs
find /var/log/crucible -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/crucible
```

## Additional Resources

- Official Documentation: https://docs.crucible.org/
- GitHub Repository: https://github.com/crucible/crucible
- Community Forum: https://forum.crucible.org/
- Best Practices Guide: https://docs.crucible.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
