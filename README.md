# airflow Installation Guide

airflow is a free and open-source workflow platform. Airflow provides platform to programmatically author and monitor workflows

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
  - Storage: 10GB for metadata
  - Network: HTTP/Celery
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default airflow port)
  - Flower on 5555
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

# Install airflow
sudo dnf install -y airflow

# Enable and start service
sudo systemctl enable --now airflow

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
airflow --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install airflow
sudo apt install -y airflow

# Enable and start service
sudo systemctl enable --now airflow

# Configure firewall
sudo ufw allow 8080

# Verify installation
airflow --version
```

### Arch Linux

```bash
# Install airflow
sudo pacman -S airflow

# Enable and start service
sudo systemctl enable --now airflow

# Verify installation
airflow --version
```

### Alpine Linux

```bash
# Install airflow
apk add --no-cache airflow

# Enable and start service
rc-update add airflow default
rc-service airflow start

# Verify installation
airflow --version
```

### openSUSE/SLES

```bash
# Install airflow
sudo zypper install -y airflow

# Enable and start service
sudo systemctl enable --now airflow

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
airflow --version
```

### macOS

```bash
# Using Homebrew
brew install airflow

# Start service
brew services start airflow

# Verify installation
airflow --version
```

### FreeBSD

```bash
# Using pkg
pkg install airflow

# Enable in rc.conf
echo 'airflow_enable="YES"' >> /etc/rc.conf

# Start service
service airflow start

# Verify installation
airflow --version
```

### Windows

```bash
# Using Chocolatey
choco install airflow

# Or using Scoop
scoop install airflow

# Verify installation
airflow --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/airflow

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
airflow --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable airflow

# Start service
sudo systemctl start airflow

# Stop service
sudo systemctl stop airflow

# Restart service
sudo systemctl restart airflow

# Check status
sudo systemctl status airflow

# View logs
sudo journalctl -u airflow -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add airflow default

# Start service
rc-service airflow start

# Stop service
rc-service airflow stop

# Restart service
rc-service airflow restart

# Check status
rc-service airflow status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'airflow_enable="YES"' >> /etc/rc.conf

# Start service
service airflow start

# Stop service
service airflow stop

# Restart service
service airflow restart

# Check status
service airflow status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start airflow
brew services stop airflow
brew services restart airflow

# Check status
brew services list | grep airflow
```

### Windows Service Manager

```powershell
# Start service
net start airflow

# Stop service
net stop airflow

# Using PowerShell
Start-Service airflow
Stop-Service airflow
Restart-Service airflow

# Check status
Get-Service airflow
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream airflow_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name airflow.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name airflow.example.com;

    ssl_certificate /etc/ssl/certs/airflow.example.com.crt;
    ssl_certificate_key /etc/ssl/private/airflow.example.com.key;

    location / {
        proxy_pass http://airflow_backend;
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
    ServerName airflow.example.com
    Redirect permanent / https://airflow.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName airflow.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/airflow.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/airflow.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend airflow_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/airflow.pem
    redirect scheme https if !{ ssl_fc }
    default_backend airflow_backend

backend airflow_backend
    balance roundrobin
    server airflow1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R airflow:airflow /etc/airflow
sudo chmod 750 /etc/airflow

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
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
sudo systemctl status airflow

# View logs
sudo journalctl -u airflow -f

# Monitor resource usage
top -p $(pgrep airflow)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/airflow"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/airflow-backup-$DATE.tar.gz" /etc/airflow /var/lib/airflow

echo "Backup completed: $BACKUP_DIR/airflow-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop airflow

# Restore from backup
tar -xzf /backup/airflow/airflow-backup-*.tar.gz -C /

# Start service
sudo systemctl start airflow
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u airflow -n 100
sudo tail -f /var/log/airflow/airflow.log

# Check configuration
airflow --version

# Check permissions
ls -la /etc/airflow
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep airflow)

# Check disk I/O
iotop -p $(pgrep airflow)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  airflow:
    image: airflow:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/airflow
      - ./data:/var/lib/airflow
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update airflow

# Debian/Ubuntu
sudo apt update && sudo apt upgrade airflow

# Arch Linux
sudo pacman -Syu airflow

# Alpine Linux
apk update && apk upgrade airflow

# openSUSE
sudo zypper update airflow

# FreeBSD
pkg update && pkg upgrade airflow

# Always backup before updates
tar -czf /backup/airflow-pre-update-$(date +%Y%m%d).tar.gz /etc/airflow

# Restart after updates
sudo systemctl restart airflow
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/airflow

# Clean old logs
find /var/log/airflow -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/airflow
```

## Additional Resources

- Official Documentation: https://docs.airflow.org/
- GitHub Repository: https://github.com/airflow/airflow
- Community Forum: https://forum.airflow.org/
- Best Practices Guide: https://docs.airflow.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
