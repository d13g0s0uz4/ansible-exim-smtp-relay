# Detailed Usage Guide

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Configuration Examples](#configuration-examples)
4. [Deployment Scenarios](#deployment-scenarios)
5. [Testing](#testing)
6. [Monitoring](#monitoring)
7. [Common Use Cases](#common-use-cases)

## Prerequisites

### Control Node Requirements

```bash
# Install Ansible on Ubuntu/Debian
sudo apt update
sudo apt install ansible python3-pip -y

# Verify installation
ansible --version
```

### Target Server Requirements

- Ubuntu 24.04 LTS (fresh installation recommended)
- Minimum 1GB RAM
- 10GB disk space
- SSH access enabled
- sudo/root privileges

### SSH Key Setup

```bash
# Generate SSH key if you don't have one
ssh-keygen -t ed25519 -C "ansible@yourdomain.com"

# Copy SSH key to target server
ssh-copy-id ubuntu@YOUR_SERVER_IP

# Test SSH connection
ssh ubuntu@YOUR_SERVER_IP
```

## Installation

### 1. Clone the Repository

```bash
cd ~/projects
git clone https://github.com/d13g0s0uz4/ansible-exim-smtp-relay.git
cd ansible-exim-smtp-relay
```

### 2. Verify Files

```bash
# Check directory structure
tree .

# Expected output:
# .
# ├── ansible.cfg
# ├── inventory.ini
# ├── playbook.yml
# ├── README.md
# └── templates
#     ├── exim4.conf.localmacros.j2
#     └── update-exim4.conf.conf.j2
```

## Configuration Examples

### Example 1: Single Local Network

Simple configuration for a single office network:

**inventory.ini:**
```ini
[mail_relay]
mail-relay ansible_host=192.168.1.50 ansible_user=ubuntu

[mail_relay:vars]
ansible_python_interpreter=/usr/bin/python3
```

**playbook.yml (vars section):**
```yaml
vars:
  local_network: "192.168.1.0/24"
  mail_name: "mail.office.local"
  enable_tls: true
  enable_rate_limiting: true
```

### Example 2: Multiple Networks (Branch Offices)

Configuration for multiple office locations:

**playbook.yml (vars section):**
```yaml
vars:
  # HQ: 192.168.1.0/24, Branch1: 192.168.10.0/24, Branch2: 10.0.0.0/16
  local_network: "192.168.1.0/24 : 192.168.10.0/24 : 10.0.0.0/16"
  mail_name: "relay.company.com"
  other_hostnames: "mail.company.com : smtp.company.com"
  enable_tls: true
```

### Example 3: VPN Network

Configuration for VPN clients:

**playbook.yml (vars section):**
```yaml
vars:
  # Office network + VPN network
  local_network: "192.168.1.0/24 : 10.8.0.0/24"
  mail_name: "vpn-relay.company.com"
  smtp_accept_max_per_host: 10  # Lower limit for VPN
```

### Example 4: Development/Testing Environment

Configuration for development servers:

**playbook.yml (vars section):**
```yaml
vars:
  local_network: "172.16.0.0/12"  # Docker networks
  mail_name: "dev-relay.local"
  enable_tls: false  # Disable TLS for local testing
  enable_rate_limiting: false
```

## Deployment Scenarios

### Scenario 1: First-Time Deployment

```bash
# Step 1: Test connectivity
ansible -i inventory.ini mail_relay -m ping

# Step 2: Check sudo access
ansible -i inventory.ini mail_relay -m shell -a "sudo whoami"

# Step 3: Run playbook with check mode (dry-run)
ansible-playbook -i inventory.ini playbook.yml --check

# Step 4: Deploy
ansible-playbook -i inventory.ini playbook.yml

# Step 5: Verify deployment
ansible -i inventory.ini mail_relay -m shell -a "sudo systemctl status exim4"
```

### Scenario 2: Update Existing Configuration

```bash
# Only update configuration files
ansible-playbook -i inventory.ini playbook.yml --tags configuration

# Skip package installation
ansible-playbook -i inventory.ini playbook.yml --skip-tags packages
```

### Scenario 3: Multiple Servers Deployment

**inventory.ini:**
```ini
[mail_relay]
relay01 ansible_host=192.168.1.50 ansible_user=ubuntu
relay02 ansible_host=192.168.1.51 ansible_user=ubuntu
relay-backup ansible_host=192.168.1.52 ansible_user=ubuntu

[mail_relay:vars]
ansible_python_interpreter=/usr/bin/python3
```

```bash
# Deploy to all servers
ansible-playbook -i inventory.ini playbook.yml

# Deploy to specific server
ansible-playbook -i inventory.ini playbook.yml --limit relay01

# Deploy one at a time (serial execution)
ansible-playbook -i inventory.ini playbook.yml --forks 1
```

### Scenario 4: Different Configurations per Server

Create host-specific variable files:

```bash
mkdir -p host_vars
```

**host_vars/relay01.yml:**
```yaml
local_network: "192.168.1.0/24"
mail_name: "relay01.office.local"
smtp_accept_max_per_host: 30
```

**host_vars/relay02.yml:**
```yaml
local_network: "192.168.10.0/24"
mail_name: "relay02.branch.local"
smtp_accept_max_per_host: 15
```

## Testing

### 1. Basic Connectivity Test

```bash
# From relay server
sudo netstat -tlnp | grep :25

# Expected output:
tcp        0      0 0.0.0.0:25              0.0.0.0:*               LISTEN      1234/exim4
```

### 2. SMTP Relay Test (from client)

```bash
# Install swaks on client machine
sudo apt install swaks -y

# Test relay
swaks --to destination@gmail.com \
      --from sender@company.local \
      --server 192.168.1.50 \
      --body "Test relay message" \
      --header "Subject: Test"
```

### 3. TLS Test

```bash
# Test STARTTLS
openssl s_client -starttls smtp -connect 192.168.1.50:25

# You should see certificate information and:
# 250 STARTTLS
```

### 4. Telnet Test

```bash
telnet 192.168.1.50 25
# Type:
EHLO test.local
MAIL FROM: <test@local.domain>
RCPT TO: <recipient@example.com>
DATA
Subject: Test Email

This is a test message.
.
QUIT
```

### 5. Check Logs

```bash
# Real-time log monitoring
sudo tail -f /var/log/exim4/mainlog

# Check for errors
sudo grep -i error /var/log/exim4/mainlog

# Check rejected messages
sudo tail -20 /var/log/exim4/rejectlog
```

## Monitoring

### Log Analysis

```bash
# Count messages sent today
sudo exiqgrep -z | wc -l

# View mail queue
sudo exim4 -bp

# View specific message
sudo exim4 -Mvh MESSAGE_ID

# Remove message from queue
sudo exim4 -Mrm MESSAGE_ID

# Force delivery attempt
sudo exim4 -M MESSAGE_ID
```

### Performance Monitoring

```bash
# Check active connections
sudo ss -tnp | grep :25

# Monitor system resources
top -p $(pgrep exim4)

# Check disk usage for mail queue
du -sh /var/spool/exim4/
```

### Create Monitoring Script

**monitor-exim.sh:**
```bash
#!/bin/bash
echo "=== Exim4 Status ==="
sudo systemctl status exim4 --no-pager

echo -e "\n=== Mail Queue ==="
sudo exim4 -bpc

echo -e "\n=== Recent Activity ==="
sudo tail -20 /var/log/exim4/mainlog

echo -e "\n=== Listening Ports ==="
sudo netstat -tlnp | grep exim
```

```bash
chmod +x monitor-exim.sh
./monitor-exim.sh
```

## Common Use Cases

### Use Case 1: Application Server Notifications

Configure your application to use the relay:

**PHP (Laravel/Symfony):**
```php
// .env
MAIL_MAILER=smtp
MAIL_HOST=192.168.1.50
MAIL_PORT=25
MAIL_ENCRYPTION=null
```

**Python:**
```python
import smtplib
from email.mime.text import MIMEText

smtp = smtplib.SMTP('192.168.1.50', 25)
msg = MIMEText('Test message')
msg['Subject'] = 'Test'
msg['From'] = 'app@local.domain'
msg['To'] = 'admin@example.com'
smtp.send_message(msg)
smtp.quit()
```

**Node.js (Nodemailer):**
```javascript
const nodemailer = require('nodemailer');

const transporter = nodemailer.createTransport({
  host: '192.168.1.50',
  port: 25,
  secure: false,
  tls: {
    rejectUnauthorized: false
  }
});
```

### Use Case 2: Monitoring System Alerts

Configure monitoring tools:

**Zabbix:**
```
Administration -> Media types -> Email
SMTP server: 192.168.1.50
SMTP server port: 25
SMTP helo: monitoring.local
```

**Nagios/Icinga:**
```cfg
# /etc/nagios/nagios.cfg
define command {
    command_name notify-host-by-email
    command_line /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$" | /usr/bin/mail -s "Host $HOSTSTATE$ Alert: $HOSTNAME$" -S smtp=192.168.1.50:25 $CONTACTEMAIL$
}
```

### Use Case 3: Backup Reports

Send automated backup reports:

```bash
#!/bin/bash
# backup-notify.sh

BACKUP_LOG="/var/log/backup.log"
RELAY="192.168.1.50"
TO="admin@example.com"
FROM="backup@server.local"

if grep -q "SUCCESS" "$BACKUP_LOG"; then
    STATUS="SUCCESS"
else
    STATUS="FAILED"
fi

mail -s "Backup Report: $STATUS" \
     -S smtp=$RELAY:25 \
     -r $FROM \
     $TO < $BACKUP_LOG
```

### Use Case 4: Docker Container Notifications

**docker-compose.yml:**
```yaml
version: '3'
services:
  app:
    image: myapp:latest
    environment:
      SMTP_HOST: 192.168.1.50
      SMTP_PORT: 25
    extra_hosts:
      - "mail-relay:192.168.1.50"
```

## Troubleshooting Commands

```bash
# Verify Exim configuration
sudo exim4 -bV

# Test configuration syntax
sudo exim4 -C /var/lib/exim4/config.autogenerated -bV

# Check if relay is allowed from specific IP
sudo exim4 -bh 192.168.1.100

# View all configuration
sudo exim4 -bP

# View specific configuration option
sudo exim4 -bP relay_from_hosts

# Regenerate configuration
sudo update-exim4.conf
sudo systemctl restart exim4

# Clear frozen messages
sudo exim4 -bpu | grep frozen | awk '{print $3}' | xargs -n1 sudo exim4 -Mrm
```

## Best Practices

1. **Use specific networks**: Don't use `0.0.0.0/0` for relay_nets
2. **Monitor logs regularly**: Set up log rotation and monitoring
3. **Update regularly**: Keep Exim and Ubuntu updated
4. **Use TLS in production**: Replace self-signed certificates
5. **Implement rate limiting**: Prevent abuse
6. **Backup configuration**: Before making changes
7. **Test before deployment**: Always use `--check` mode first
8. **Document changes**: Keep track of configuration modifications
9. **Use version control**: Store configurations in Git
10. **Implement monitoring**: Set up alerts for mail queue issues

## Advanced Configuration

For advanced features like:
- DKIM signing
- SPF validation  
- DMARC reporting
- Spam filtering (SpamAssassin)
- Virus scanning (ClamAV)
- LDAP authentication

Refer to the [Exim documentation](https://www.exim.org/docs.html) and consider extending the playbook.