# MinIO Installation Guide

MinIO is a free and open-source high-performance object storage server compatible with Amazon S3. MinIO provides enterprise-grade object storage that is 100% open source and S3 compatible, serving as a powerful alternative to Amazon S3, Azure Blob Storage, or Google Cloud Storage without vendor lock-in

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
  - RAM: 2GB minimum (8GB+ for production)
  - Storage: 10GB minimum plus data storage
  - Network: Stable network for distributed deployments
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 12.0+
- **Network Requirements**:
  - Port 9000 (default MinIO port)
  - Port 9001 for MinIO Console
- **Dependencies**:
  - None (standalone binary)
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

# Install MinIO
sudo dnf install -y minio

# Enable and start service
sudo systemctl enable --now minio

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
minio --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install MinIO
sudo apt install -y minio

# Enable and start service
sudo systemctl enable --now minio

# Configure firewall
sudo ufw allow 9000

# Verify installation
minio --version
```

### Arch Linux

```bash
# Install MinIO
sudo pacman -S minio

# Enable and start service
sudo systemctl enable --now minio

# Verify installation
minio --version
```

### Alpine Linux

```bash
# Install MinIO
apk add --no-cache minio

# Enable and start service
rc-update add minio default
rc-service minio start

# Verify installation
minio --version
```

### openSUSE/SLES

```bash
# Install MinIO
sudo zypper install -y minio

# Enable and start service
sudo systemctl enable --now minio

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
minio --version
```

### macOS

```bash
# Using Homebrew
brew install minio

# Start service
brew services start minio

# Verify installation
minio --version
```

### FreeBSD

```bash
# Using pkg
pkg install minio

# Enable in rc.conf
echo 'minio_enable="YES"' >> /etc/rc.conf

# Start service
service minio start

# Verify installation
minio --version
```

### Windows

```bash
# Using Chocolatey
choco install minio

# Or using Scoop
scoop install minio

# Verify installation
minio --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/minio

# Set up basic configuration
# Configuration details will vary based on your specific needs
# See official documentation for detailed configuration options

# Test configuration
minio server --help
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable minio

# Start service
sudo systemctl start minio

# Stop service
sudo systemctl stop minio

# Restart service
sudo systemctl restart minio

# Check status
sudo systemctl status minio

# View logs
sudo journalctl -u minio -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add minio default

# Start service
rc-service minio start

# Stop service
rc-service minio stop

# Restart service
rc-service minio restart

# Check status
rc-service minio status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'minio_enable="YES"' >> /etc/rc.conf

# Start service
service minio start

# Stop service
service minio stop

# Restart service
service minio restart

# Check status
service minio status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start minio
brew services stop minio
brew services restart minio

# Check status
brew services list | grep minio
```

### Windows Service Manager

```powershell
# Start service
net start minio

# Stop service
net stop minio

# Using PowerShell
Start-Service minio
Stop-Service minio
Restart-Service minio

# Check status
Get-Service minio
```

## Advanced Configuration

### Advanced MinIO Configuration

See the official documentation for advanced configuration options including:
- High availability setup
- Performance tuning
- Security hardening
- Integration with other services


## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream minio_backend {
    server 127.0.0.1:9000;
}

server {
    listen 80;
    server_name minio.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name minio.example.com;

    ssl_certificate /etc/ssl/certs/minio.example.com.crt;
    ssl_certificate_key /etc/ssl/private/minio.example.com.key;

    location / {
        proxy_pass http://minio_backend;
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
    ServerName minio.example.com
    Redirect permanent / https://minio.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName minio.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/minio.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/minio.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9000/
    ProxyPassReverse / http://127.0.0.1:9000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend minio_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/minio.pem
    redirect scheme https if !{ ssl_fc }
    default_backend minio_backend

backend minio_backend
    balance roundrobin
    server minio1 127.0.0.1:9000 check
```

## Security Configuration

### Security Best Practices

```bash
# Set appropriate permissions
sudo chown -R minio:minio /etc/minio
sudo chmod 750 /etc/minio

# Configure firewall rules
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

Not applicable

## Performance Optimization

### 8. Performance Tuning

```bash
# System tuning for MinIO
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Monitor performance
mc admin info local
```

## Monitoring

### Monitoring Setup

```bash
# Basic monitoring
sudo systemctl status minio
sudo journalctl -u minio -f

# Set up health checks
curl -f http://localhost:9000/health || exit 1
```

## 9. Backup and Restore

### Backup Procedures

```bash
#!/bin/bash
# Backup script
BACKUP_DIR="/backup/minio"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
mc mirror --overwrite /var/lib/minio /backup/minio

# Restore procedure
# Stop service, restore files, restart service
sudo systemctl stop minio
# Restore backed up files
sudo systemctl start minio
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u minio -f
sudo tail -f /var/log/minio/minio.log

# Check configuration
minio server --help

# Check permissions
ls -la /etc/minio
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9000

# Test connectivity
telnet localhost 9000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep minio)

# Check disk I/O
iotop -p $(pgrep minio)

# Check network connections
ss -an | grep 9000
```



## Integration Examples

### Example Integration

```yaml
# Docker Compose example
version: '3.8'
services:
  minio:
    image: minio:latest
    ports:
      - "9000:9000"
    volumes:
      - ./config:/etc/minio
      - ./data:/var/lib/minio
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update minio

# Debian/Ubuntu
sudo apt update && sudo apt upgrade minio

# Arch Linux
sudo pacman -Syu minio

# Alpine Linux
apk update && apk upgrade minio

# openSUSE
sudo zypper update minio

# FreeBSD
pkg update && pkg upgrade minio

# Always backup before updates
mc mirror --overwrite /var/lib/minio /backup/minio

# Restart after updates
sudo systemctl restart minio
```

### Regular Maintenance Tasks

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/minio

# Clean old logs
find /var/log/minio -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/minio

# Verify configuration
minio server --help

# Test functionality
mc admin info local
```



## Additional Resources

- [Official Documentation](https://docs.min.io/)
- [GitHub Repository](https://github.com/minio/minio)
- [Community Forum](https://slack.min.io/)
- [Best Practices Guide](https://docs.min.io/docs/minio-best-practices.html)


---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
