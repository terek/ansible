# Ansible Vault Setup

## Initial Setup

1. **Encrypt the vault file:**
   ```bash
   cd ansible
   ansible-vault encrypt vault.yml
   ```
   You'll be prompted to enter a vault password. Remember this password!

2. **Create a vault password file (optional but recommended):**
   ```bash
   echo "your-vault-password" > .vault_pass
   chmod 600 .vault_pass
   ```
   Add `.vault_pass` to your `.gitignore` file.

## Usage

### Running the playbook with vault:

**Option 1: Prompt for password**
```bash
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
```

**Option 2: Using password file**
```bash
ansible-playbook -i inventory.ini playbook.yml --vault-password-file .vault_pass
```

### Managing vault contents:

**Edit the vault:**
```bash
ansible-vault edit vault.yml --vault-password-file .vault_pass
```

**View vault contents:**
```bash
ansible-vault view vault.yml
```

**Change vault password:**
```bash
ansible-vault rekey vault.yml
```

## Security Notes

- Never commit the vault password file to git
- Use strong passwords for the vault
- Regularly rotate secrets stored in the vault
- Consider using different vault files for different environments (dev, staging, prod)

## Environment Variables in the Scraper

The scraper now reads the following environment variables:
- `SCRAPER_AUTH_TOKEN`: The bearer token for API authentication
- `NODE_ENV`: Set to "production" in the systemd service

To add more secrets:
1. Add them to `vault.yml`
2. Update `scraper.env.j2` template to include the new variables
3. Modify the scraper code to read the new environment variables

## Development Setup

For local development, create a `.env` file in the `scraper/` directory:

```bash
# Copy this content to scraper/.env for development
SCRAPER_AUTH_TOKEN=your-secret-token-here
NODE_ENV=development
# PORT=3000  # optional, defaults to 3000
```

The `.env` file should not be committed to git (it's in .gitignore).
