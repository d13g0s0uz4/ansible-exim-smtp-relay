# Quick Start Guide

Get your Exim SMTP relay running in under 5 minutes!

## Prerequisites Checklist

- [ ] Ubuntu 24.04 LTS server with SSH access
- [ ] Ansible installed on your control machine
- [ ] SSH key copied to target server
- [ ] Sudo access on target server

## 5-Minute Setup

### Step 1: Clone and Configure (1 minute)

```bash
# Clone the repository
git clone https://github.com/d13g0s0uz4/ansible-exim-smtp-relay.git
cd ansible-exim-smtp-relay

# Edit inventory - add your server IP
nano inventory.ini
```

**Add your server:**
```ini
[mail_relay]
my-relay ansible_host=192.168.1.50 ansible_user=ubuntu
```

### Step 2: Customize Variables (1 minute)

```bash
# Edit playbook variables
nano playbook.yml
```

**Update these lines:**
```yaml
vars:
  local_network: "192.168.1.0/24"  # Your network
  mail_name: "relay.yourdomain.local"  # Your hostname
```

### Step 3: Test Connection (30 seconds)

```bash
# Verify connectivity
ansible -i inventory.ini mail_relay -m ping
```

**Expected output:**
```
my-relay | SUCCESS => {
    "ping": "pong"
}
```

### Step 4: Deploy! (2 minutes)

```bash
# Run the playbook
ansible-playbook -i inventory.ini playbook.yml
```

**Wait for completion** - You'll see green "ok" and "changed" messages.

### Step 5: Verify (30 seconds)

```bash
# Check Exim is running
ansible -i inventory.ini mail_relay -m shell -a "sudo systemctl status exim4" 

# Check it's listening
ansible -i inventory.ini mail_relay -m shell -a "sudo netstat -tlnp | grep :25"
```

## Test Your Relay

### From Another Machine on Your Network

```bash
# Install testing tool
sudo apt install swaks -y

# Send test email
swaks --to your-email@gmail.com \
      --from test@local.domain \
      --server 192.168.1.50 \
      --body "Test from Exim relay"
```

Check your email inbox - you should receive the test message!

## Common First-Time Issues

### Issue: "Permission denied" during deployment

**Solution:**
```bash
# Ensure your user has sudo rights
ssh ubuntu@192.168.1.50
sudo whoami  # Should return "root"
```

### Issue: "Relay not permitted"

**Solution:** Check your client IP is in the allowed network range:
```bash
ssh ubuntu@192.168.1.50
sudo grep dc_relay_nets /etc/exim4/update-exim4.conf.conf
```

### Issue: Can't connect to port 25

**Solution:** Check firewall:
```bash
ssh ubuntu@192.168.1.50
sudo ufw status
sudo ufw allow 25/tcp
```

## What Just Happened?

The playbook:

1. ✅ Installed Exim4 and dependencies
2. ✅ Configured Exim to relay for your network
3. ✅ Generated self-signed TLS certificates
4. ✅ Enabled rate limiting (20 connections per host)
5. ✅ Configured firewall rules
6. ✅ Started and enabled Exim4 service

## Next Steps

### Configure Your Applications

Use these settings in your apps:

```
SMTP Host: 192.168.1.50
SMTP Port: 25
Encryption: None (or STARTTLS if enabled)
Authentication: None
```

### Monitor Your Relay

```bash
# Watch logs in real-time
ssh ubuntu@192.168.1.50
sudo tail -f /var/log/exim4/mainlog
```

### Add More Networks

Edit `playbook.yml` and add more networks:

```yaml
local_network: "192.168.1.0/24 : 192.168.10.0/24 : 10.0.0.0/8"
```

Then re-run:
```bash
ansible-playbook -i inventory.ini playbook.yml --tags configuration
```

## Production Checklist

Before going to production:

- [ ] Use a proper hostname with DNS
- [ ] Replace self-signed certificate with CA-signed
- [ ] Set up log monitoring
- [ ] Configure backup
- [ ] Document your setup
- [ ] Test failover (if using multiple relays)
- [ ] Set up SPF/DKIM records

## Get Help

- **Detailed docs**: See [USAGE.md](USAGE.md)
- **Configuration examples**: See [README.md](README.md)
- **Issues**: [GitHub Issues](https://github.com/d13g0s0uz4/ansible-exim-smtp-relay/issues)

## One-Liner for the Impatient

```bash
git clone https://github.com/d13g0s0uz4/ansible-exim-smtp-relay.git && \
cd ansible-exim-smtp-relay && \
echo "[mail_relay]\nrelay ansible_host=YOUR_IP ansible_user=ubuntu" > inventory.ini && \
ansible-playbook -i inventory.ini playbook.yml
```

**Remember to replace `YOUR_IP` with your actual server IP!**

Congratulations! Your SMTP relay is ready! 🎉