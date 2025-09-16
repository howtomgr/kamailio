# Kamailio Installation Guide

Kamailio is a free and open-source SIP Server. Open source SIP server

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 5060 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5060 (default kamailio port)
- **Dependencies**:
  - kamailio-mysql-modules
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

# Install kamailio
sudo dnf install -y kamailio kamailio-mysql-modules

# Enable and start service
sudo systemctl enable --now kamailio

# Configure firewall
sudo firewall-cmd --permanent --add-service=kamailio
sudo firewall-cmd --reload

# Verify installation
kamailio --version || systemctl status kamailio
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install kamailio
sudo apt install -y kamailio kamailio-mysql-modules

# Enable and start service
sudo systemctl enable --now kamailio

# Configure firewall
sudo ufw allow 5060

# Verify installation
kamailio --version || systemctl status kamailio
```

### Arch Linux

```bash
# Install kamailio
sudo pacman -S kamailio

# Enable and start service
sudo systemctl enable --now kamailio

# Verify installation
kamailio --version || systemctl status kamailio
```

### Alpine Linux

```bash
# Install kamailio
apk add --no-cache kamailio

# Enable and start service
rc-update add kamailio default
rc-service kamailio start

# Verify installation
kamailio --version || rc-service kamailio status
```

### openSUSE/SLES

```bash
# Install kamailio
sudo zypper install -y kamailio kamailio-mysql-modules

# Enable and start service
sudo systemctl enable --now kamailio

# Configure firewall
sudo firewall-cmd --permanent --add-service=kamailio
sudo firewall-cmd --reload

# Verify installation
kamailio --version || systemctl status kamailio
```

### macOS

```bash
# Using Homebrew
brew install kamailio

# Start service
brew services start kamailio

# Verify installation
kamailio --version
```

### FreeBSD

```bash
# Using pkg
pkg install kamailio

# Enable in rc.conf
echo 'kamailio_enable="YES"' >> /etc/rc.conf

# Start service
service kamailio start

# Verify installation
kamailio --version || service kamailio status
```

### Windows

```powershell
# Using Chocolatey
choco install kamailio

# Or using Scoop
scoop install kamailio

# Verify installation
kamailio --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/kamailio

# Set up basic configuration
sudo tee /etc/kamailio/kamailio.conf << 'EOF'
# Kamailio Configuration
children=8
EOF

# Test configuration
sudo kamailio -t || sudo kamailio configtest

# Reload service
sudo systemctl reload kamailio
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R kamailio:kamailio /etc/kamailio
sudo chmod 750 /etc/kamailio

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable kamailio

# Start service
sudo systemctl start kamailio

# Stop service
sudo systemctl stop kamailio

# Restart service
sudo systemctl restart kamailio

# Reload configuration
sudo systemctl reload kamailio

# Check status
sudo systemctl status kamailio

# View logs
sudo journalctl -u kamailio -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add kamailio default

# Start service
rc-service kamailio start

# Stop service
rc-service kamailio stop

# Restart service
rc-service kamailio restart

# Check status
rc-service kamailio status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'kamailio_enable="YES"' >> /etc/rc.conf

# Start service
service kamailio start

# Stop service
service kamailio stop

# Restart service
service kamailio restart

# Check status
service kamailio status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start kamailio
brew services stop kamailio
brew services restart kamailio

# Check status
brew services list | grep kamailio
```

### Windows Service Manager

```powershell
# Start service
net start kamailio

# Stop service
net stop kamailio

# Using PowerShell
Start-Service kamailio
Stop-Service kamailio
Restart-Service kamailio

# Check status
Get-Service kamailio
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/kamailio/kamailio.conf << 'EOF'
children=8
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart kamailio
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream kamailio_backend {
    server 127.0.0.1:5060;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name kamailio.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name kamailio.example.com;

    ssl_certificate /etc/ssl/certs/kamailio.example.com.crt;
    ssl_certificate_key /etc/ssl/private/kamailio.example.com.key;

    location / {
        proxy_pass http://kamailio_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName kamailio.example.com
    Redirect permanent / https://kamailio.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName kamailio.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/kamailio.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/kamailio.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5060/
    ProxyPassReverse / http://127.0.0.1:5060/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:5060/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend kamailio_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/kamailio.pem
    redirect scheme https if !{ ssl_fc }
    default_backend kamailio_backend

backend kamailio_backend
    balance roundrobin
    option httpchk GET /health
    server kamailio1 127.0.0.1:5060 check
    server kamailio2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R kamailio:kamailio /etc/kamailio
sudo chmod 750 /etc/kamailio

# Configure firewall
sudo firewall-cmd --permanent --add-service=kamailio
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/kamailio.conf << 'EOF'
[kamailio]
enabled = true
port = 5060
filter = kamailio
logpath = /var/log/kamailio/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/kamailio.key \
    -out /etc/ssl/certs/kamailio.crt

# Configure SSL in kamailio
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE kamailio_db;
CREATE USER kamailio_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE kamailio_db TO kamailio_user;
EOF

# Configure kamailio to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE kamailio_db;
CREATE USER 'kamailio_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON kamailio_db.* TO 'kamailio_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Kamailio specific tuning
children=8
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
kamailio soft nofile 65535
kamailio hard nofile 65535
kamailio soft nproc 32768
kamailio hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'kamailio'
    static_configs:
      - targets: ['localhost:5060']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet kamailio; then
    echo "Kamailio is running"
    exit 0
else
    echo "Kamailio is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/kamailio << 'EOF'
/var/log/kamailio/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 kamailio kamailio
    postrotate
        systemctl reload kamailio > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Kamailio backup script
BACKUP_DIR="/backup/kamailio"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop kamailio

# Backup configuration
tar -czf "$BACKUP_DIR/kamailio-config-$DATE.tar.gz" /etc/kamailio

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/kamailio-data-$DATE.tar.gz" /var/lib/kamailio

# Start service
systemctl start kamailio

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop kamailio

# Restore configuration
sudo tar -xzf /backup/kamailio/kamailio-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/kamailio/kamailio-data-*.tar.gz -C /

# Set permissions
sudo chown -R kamailio:kamailio /etc/kamailio
sudo chown -R kamailio:kamailio /var/lib/kamailio

# Start service
sudo systemctl start kamailio
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u kamailio -n 100
sudo tail -f /var/log/kamailio/*.log

# Check configuration
sudo kamailio -t || sudo kamailio configtest

# Check permissions
ls -la /etc/kamailio
ls -la /var/lib/kamailio
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5060
sudo netstat -tlnp | grep 5060

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 5060
nc -zv localhost 5060
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep kamailio)
htop -p $(pgrep kamailio)

# Check connections
ss -ant | grep :5060 | wc -l

# Monitor I/O
iotop -p $(pgrep kamailio)
```

### Debug Mode

```bash
# Run in debug mode
sudo kamailio -d
# or
sudo kamailio debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  kamailio:
    image: kamailio:latest
    container_name: kamailio
    ports:
      - "5060:5060"
    volumes:
      - ./config:/etc/kamailio
      - ./data:/var/lib/kamailio
    environment:
      - kamailio_CONFIG=/etc/kamailio/kamailio.conf
    restart: unless-stopped
    networks:
      - kamailio_net

networks:
  kamailio_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kamailio
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kamailio
  template:
    metadata:
      labels:
        app: kamailio
    spec:
      containers:
      - name: kamailio
        image: kamailio:latest
        ports:
        - containerPort: 5060
        volumeMounts:
        - name: config
          mountPath: /etc/kamailio
      volumes:
      - name: config
        configMap:
          name: kamailio-config
---
apiVersion: v1
kind: Service
metadata:
  name: kamailio
spec:
  selector:
    app: kamailio
  ports:
  - port: 5060
    targetPort: 5060
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Kamailio
  hosts: all
  become: yes
  tasks:
    - name: Install kamailio
      package:
        name: kamailio
        state: present
    
    - name: Configure kamailio
      template:
        src: kamailio.conf.j2
        dest: /etc/kamailio/kamailio.conf
        owner: kamailio
        group: kamailio
        mode: '0640'
      notify: restart kamailio
    
    - name: Start and enable kamailio
      systemd:
        name: kamailio
        state: started
        enabled: yes
  
  handlers:
    - name: restart kamailio
      systemd:
        name: kamailio
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update kamailio

# Debian/Ubuntu
sudo apt update && sudo apt upgrade kamailio

# Arch Linux
sudo pacman -Syu kamailio

# Alpine Linux
apk update && apk upgrade kamailio

# openSUSE
sudo zypper update kamailio

# FreeBSD
pkg update && pkg upgrade kamailio

# Always backup before updates
tar -czf /backup/kamailio-pre-update-$(date +%Y%m%d).tar.gz /etc/kamailio

# Restart after updates
sudo systemctl restart kamailio
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/kamailio -name "*.log" -mtime +30 -delete

# Verify integrity
sudo kamailio --verify || sudo kamailio check

# Update databases (if applicable)
sudo kamailio-update-db

# Optimize performance
sudo kamailio-optimize

# Check for security updates
sudo kamailio --security-check
```

## Additional Resources

- Official Documentation: https://docs.kamailio.org/
- GitHub Repository: https://github.com/kamailio/kamailio
- Community Forum: https://forum.kamailio.org/
- Wiki: https://wiki.kamailio.org/
- Comparison vs OpenSIPS, Asterisk, FreeSWITCH, repro: https://docs.kamailio.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
