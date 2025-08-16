# Cloudflared Tunnel Setup

This Ansible role sets up a Cloudflare tunnel (cloudflared) to securely expose your local services to the internet without opening ports on your firewall.

## Prerequisites

1. **Cloudflare Account**: You need a Cloudflare account with a domain configured
2. **Domain in Cloudflare**: Your domain must be using Cloudflare nameservers
3. **Cloudflared CLI**: Install cloudflared on your local machine for initial setup

## Setup Instructions

### 1. Install Cloudflared Locally (if not already installed)

```bash
# On macOS
brew install cloudflared

# On Linux
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
sudo mv cloudflared-linux-amd64 /usr/local/bin/cloudflared
sudo chmod +x /usr/local/bin/cloudflared
```

### 2. Authenticate with Cloudflare

```bash
cloudflared tunnel login
```

This opens a browser window where you select your domain and authorize cloudflared.

### 3. Create a Tunnel

```bash
cloudflared tunnel create yimple-tunnel
```

**Important**: Note the UUID that's generated - you'll need this for the configuration!

Example output:
```
Tunnel credentials written to /Users/you/.cloudflared/12345678-1234-1234-1234-123456789abc.json
```

The UUID in this example is `12345678-1234-1234-1234-123456789abc`.

### 4. Configure Variables

Edit `ansible/cloudflared/vars/main.yml` and set:

```yaml
tunnel_name: "yimple-tunnel"  # The name you used in step 3
tunnel_uuid: "12345678-1234-1234-1234-123456789abc"  # The UUID from step 3
domain_name: "yourdomain.com"  # Your actual domain
```

Update the `services` section to match your actual services and ports.

### 5. Path Understanding

**Important**: Understand where files are located:

| Location | Machine | Path |
|----------|---------|------|
| **Credentials (created)** | Dev machine | `~/.cloudflared/UUID.json` |
| **Credentials (deployed)** | Target VM | `/etc/cloudflared/UUID.json` |
| **Config file** | Target VM | `/etc/cloudflared/config.yml` |
| **Binary** | Target VM | `/usr/local/bin/cloudflared` |

The Ansible playbook will **automatically copy** the credentials from your dev machine to the target VM, so you don't need to manually scp them!

### 6. Run the Ansible Playbook

```bash
ansible-playbook -i inventory.ini playbook.yml --tags cloudflared
```

The playbook will automatically:
- Copy credentials from your dev machine to the target VM
- Deploy the configuration 
- Start the cloudflared service

### 7. Configure DNS Records

For each hostname in your services configuration, create a CNAME record pointing to your tunnel:

```bash
# For each service hostname, run:
cloudflared tunnel route dns yimple-tunnel yourdomain.com
cloudflared tunnel route dns yimple-tunnel api.yourdomain.com
cloudflared tunnel route dns yimple-tunnel scraper.yourdomain.com
```

Or set up the DNS records manually in the Cloudflare dashboard:
- Type: CNAME
- Name: @ (for root domain) or subdomain name
- Content: `12345678-1234-1234-1234-123456789abc.cfargotunnel.com`

### 8. Verify Setup

Check that the tunnel is running:

```bash
sudo systemctl status cloudflared
sudo journalctl -u cloudflared -f
```

Test your services:
```bash
curl https://yourdomain.com
curl https://api.yourdomain.com
```

## Configuration Reference

### Service Types

The `services` configuration supports various service types:

```yaml
services:
  # HTTP service
  - hostname: "app.yourdomain.com"
    service: "http://localhost:3000"
    
  # HTTPS service
  - hostname: "secure.yourdomain.com"
    service: "https://localhost:3001"
    
  # SSH access
  - hostname: "ssh.yourdomain.com"
    service: "ssh://localhost:22"
    
  # Service with path matching
  - hostname: "api.yourdomain.com"
    path: "/v1/*"
    service: "http://localhost:3001"
    
  # Hello world test
  - hostname: "test.yourdomain.com"
    service: "hello_world"
```

### Common Issues

1. **"tunnel credentials not found"**: Make sure you copied the UUID.json file to `/etc/cloudflared/`
2. **"502 Bad Gateway"**: Check that your local service is running and accessible
3. **"DNS resolution failed"**: Verify your CNAME records are properly configured
4. **Permission denied**: Ensure the cloudflared user owns the credentials file

### Security Notes

- The credentials file contains sensitive information - keep it secure
- Consider using Cloudflare Access for additional authentication
- Regularly rotate tunnel credentials
- Monitor tunnel usage in the Cloudflare dashboard

## Troubleshooting

Check tunnel status:
```bash
cloudflared tunnel info yimple-tunnel
cloudflared tunnel list
```

Test tunnel connectivity:
```bash
cloudflared tunnel run --url http://localhost:3000 yimple-tunnel
```

View detailed logs:
```bash
sudo journalctl -u cloudflared -f --no-pager
```
