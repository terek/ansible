# Yimple Ansible Configuration

This Ansible configuration manages the deployment and configuration of the Yimple application infrastructure.

## Inventory

- **Target**: `91.99.94.164` (web server)
- **User**: `root`

## Available Roles

### nodejs
- Installs Node.js 20.x from NodeSource repository
- Sets up proper GPG keys and repositories
- Verifies installation

### cloudflared
- Downloads and installs cloudflared tunnel client
- Creates dedicated user and configuration directories
- Sets up systemd service for tunnel management
- **Note**: Requires manual tunnel configuration in Cloudflare dashboard

### scraper
- Deploys the Yimple scraper service
- Creates dedicated `scraper` user and directories
- Copies compiled application from `../scraper/dist/`
- Sets up systemd service with security hardening
- Automatically starts on boot
- **Dependencies**: Requires `nodejs` role

### firewall (commented out)
- UFW firewall configuration
- Allows SSH access and Cloudflare IPs
- Blocks all other incoming traffic

## Usage

```bash
# Deploy all active roles
ansible-playbook -i inventory.ini playbook.yml

# Deploy specific roles
ansible-playbook -i inventory.ini playbook.yml --tags "nodejs,scraper"
```

## Service Management

After deployment, services can be managed via systemd:

```bash
# Check scraper status
systemctl status yimple-scraper

# View scraper logs
journalctl -u yimple-scraper -f

# Restart scraper
systemctl restart yimple-scraper
``` 
