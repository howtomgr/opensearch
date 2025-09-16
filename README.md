# opensearch Installation Guide

opensearch is a free and open-source search and analytics suite. OpenSearch provides open source search and analytics suite forked from Elasticsearch

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
  - RAM: 4GB minimum (8GB+ recommended)
  - Storage: 50GB+ for indices
  - Network: HTTP/REST API
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9200 (default opensearch port)
  - Transport on 9300
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

# Install opensearch
sudo dnf install -y opensearch

# Enable and start service
sudo systemctl enable --now opensearch

# Configure firewall
sudo firewall-cmd --permanent --add-port=9200/tcp
sudo firewall-cmd --reload

# Verify installation
opensearch --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install opensearch
sudo apt install -y opensearch

# Enable and start service
sudo systemctl enable --now opensearch

# Configure firewall
sudo ufw allow 9200

# Verify installation
opensearch --version
```

### Arch Linux

```bash
# Install opensearch
sudo pacman -S opensearch

# Enable and start service
sudo systemctl enable --now opensearch

# Verify installation
opensearch --version
```

### Alpine Linux

```bash
# Install opensearch
apk add --no-cache opensearch

# Enable and start service
rc-update add opensearch default
rc-service opensearch start

# Verify installation
opensearch --version
```

### openSUSE/SLES

```bash
# Install opensearch
sudo zypper install -y opensearch

# Enable and start service
sudo systemctl enable --now opensearch

# Configure firewall
sudo firewall-cmd --permanent --add-port=9200/tcp
sudo firewall-cmd --reload

# Verify installation
opensearch --version
```

### macOS

```bash
# Using Homebrew
brew install opensearch

# Start service
brew services start opensearch

# Verify installation
opensearch --version
```

### FreeBSD

```bash
# Using pkg
pkg install opensearch

# Enable in rc.conf
echo 'opensearch_enable="YES"' >> /etc/rc.conf

# Start service
service opensearch start

# Verify installation
opensearch --version
```

### Windows

```bash
# Using Chocolatey
choco install opensearch

# Or using Scoop
scoop install opensearch

# Verify installation
opensearch --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/opensearch

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
opensearch --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable opensearch

# Start service
sudo systemctl start opensearch

# Stop service
sudo systemctl stop opensearch

# Restart service
sudo systemctl restart opensearch

# Check status
sudo systemctl status opensearch

# View logs
sudo journalctl -u opensearch -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add opensearch default

# Start service
rc-service opensearch start

# Stop service
rc-service opensearch stop

# Restart service
rc-service opensearch restart

# Check status
rc-service opensearch status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'opensearch_enable="YES"' >> /etc/rc.conf

# Start service
service opensearch start

# Stop service
service opensearch stop

# Restart service
service opensearch restart

# Check status
service opensearch status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start opensearch
brew services stop opensearch
brew services restart opensearch

# Check status
brew services list | grep opensearch
```

### Windows Service Manager

```powershell
# Start service
net start opensearch

# Stop service
net stop opensearch

# Using PowerShell
Start-Service opensearch
Stop-Service opensearch
Restart-Service opensearch

# Check status
Get-Service opensearch
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream opensearch_backend {
    server 127.0.0.1:9200;
}

server {
    listen 80;
    server_name opensearch.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name opensearch.example.com;

    ssl_certificate /etc/ssl/certs/opensearch.example.com.crt;
    ssl_certificate_key /etc/ssl/private/opensearch.example.com.key;

    location / {
        proxy_pass http://opensearch_backend;
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
    ServerName opensearch.example.com
    Redirect permanent / https://opensearch.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName opensearch.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/opensearch.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/opensearch.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9200/
    ProxyPassReverse / http://127.0.0.1:9200/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend opensearch_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/opensearch.pem
    redirect scheme https if !{ ssl_fc }
    default_backend opensearch_backend

backend opensearch_backend
    balance roundrobin
    server opensearch1 127.0.0.1:9200 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R opensearch:opensearch /etc/opensearch
sudo chmod 750 /etc/opensearch

# Configure firewall
sudo firewall-cmd --permanent --add-port=9200/tcp
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
sudo systemctl status opensearch

# View logs
sudo journalctl -u opensearch -f

# Monitor resource usage
top -p $(pgrep opensearch)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/opensearch"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/opensearch-backup-$DATE.tar.gz" /etc/opensearch /var/lib/opensearch

echo "Backup completed: $BACKUP_DIR/opensearch-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop opensearch

# Restore from backup
tar -xzf /backup/opensearch/opensearch-backup-*.tar.gz -C /

# Start service
sudo systemctl start opensearch
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u opensearch -n 100
sudo tail -f /var/log/opensearch/opensearch.log

# Check configuration
opensearch --version

# Check permissions
ls -la /etc/opensearch
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9200

# Test connectivity
telnet localhost 9200

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep opensearch)

# Check disk I/O
iotop -p $(pgrep opensearch)

# Check connections
ss -an | grep 9200
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  opensearch:
    image: opensearch:latest
    ports:
      - "9200:9200"
    volumes:
      - ./config:/etc/opensearch
      - ./data:/var/lib/opensearch
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update opensearch

# Debian/Ubuntu
sudo apt update && sudo apt upgrade opensearch

# Arch Linux
sudo pacman -Syu opensearch

# Alpine Linux
apk update && apk upgrade opensearch

# openSUSE
sudo zypper update opensearch

# FreeBSD
pkg update && pkg upgrade opensearch

# Always backup before updates
tar -czf /backup/opensearch-pre-update-$(date +%Y%m%d).tar.gz /etc/opensearch

# Restart after updates
sudo systemctl restart opensearch
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/opensearch

# Clean old logs
find /var/log/opensearch -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/opensearch
```

## Additional Resources

- Official Documentation: https://docs.opensearch.org/
- GitHub Repository: https://github.com/opensearch/opensearch
- Community Forum: https://forum.opensearch.org/
- Best Practices Guide: https://docs.opensearch.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
