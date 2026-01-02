# Ansible Server Management Scripts

A comprehensive collection of Ansible playbooks for managing and maintaining Linux servers. Designed for use with Ansible Semaphore or direct command-line execution.

## 📁 Directory Structure

```
ansible-scripts/
├── inventory/
│   └── hosts.yml          # Server inventory file
└── playbooks/
    ├── system-updates.yml          # APT updates and upgrades
    ├── container-update.yml        # Docker container management
    ├── server-setup.yml            # Initial server configuration
    ├── security-hardening.yml      # Security best practices
    └── health-check.yml            # System health monitoring
```

## 📋 Prerequisites

- Ansible 2.9 or higher installed on your control machine
- SSH access to target servers
- Sudo privileges on target servers
- Python 3 on target servers

### Install Ansible

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# Using pip
pip3 install ansible
```

## 🚀 Quick Start

### 1. Configure Your Inventory

Edit [inventory/hosts.yml](inventory/hosts.yml) and add your servers:

```yaml
all:
  children:
    production:
      hosts:
        server-01:
          ansible_host: 192.168.1.10
          ansible_user: ubuntu
```

### 2. Test Connectivity

```bash
ansible all -i inventory/hosts.yml -m ping
```

### 3. Run a Playbook

```bash
ansible-playbook -i inventory/hosts.yml playbooks/system-updates.yml
```

## 📚 Available Playbooks

### 1. System Updates (`system-updates.yml`)

Performs APT package updates and upgrades on Debian/Ubuntu systems.

**Features:**
- Updates package cache
- Performs distribution upgrade
- Automatic cleanup of old packages
- Optional automatic reboot if required
- Displays upgrade summary

**Usage:**
```bash
# Basic usage
ansible-playbook -i inventory/hosts.yml playbooks/system-updates.yml

# Enable automatic reboot
ansible-playbook -i inventory/hosts.yml playbooks/system-updates.yml -e "auto_reboot=yes"

# Target specific group
ansible-playbook -i inventory/hosts.yml playbooks/system-updates.yml --limit production
```

### 2. Container Update (`container-update.yml`)

Manages Docker containers: pulls latest images and restarts containers.

**Features:**
- Pulls latest Docker images
- Restarts containers with updated images
- Verifies container health after restart
- Optional image pruning
- Shows disk usage

**Usage:**
```bash
# Update all containers on docker_hosts
ansible-playbook -i inventory/hosts.yml playbooks/container-update.yml

# Update specific containers
ansible-playbook -i inventory/hosts.yml playbooks/container-update.yml -e "target_containers=['nginx','mysql']"

# Disable image pruning
ansible-playbook -i inventory/hosts.yml playbooks/container-update.yml -e "prune_images=no"
```

### 3. Server Setup (`server-setup.yml`)

Initial configuration for new servers.

**Features:**
- Installs essential packages
- Configures timezone
- Creates sudo user (optional)
- Hardens SSH configuration
- Configures firewall (UFW)
- Sets up fail2ban
- Enables automatic security updates
- Sets hostname

**Usage:**
```bash
# Basic setup
ansible-playbook -i inventory/hosts.yml playbooks/server-setup.yml

# Create new user with SSH key
ansible-playbook -i inventory/hosts.yml playbooks/server-setup.yml \
  -e "create_user=admin" \
  -e "user_ssh_key='ssh-rsa AAAAB3...'"

# Set custom timezone
ansible-playbook -i inventory/hosts.yml playbooks/server-setup.yml \
  -e "server_timezone=America/New_York"
```

### 4. Security Hardening (`security-hardening.yml`)

Implements security best practices and hardening measures.

**Features:**
- SSH hardening (disable root login, enforce key auth)
- Fail2ban configuration
- Firewall setup with UFW
- Kernel parameter hardening
- Automatic security updates
- Password policy enforcement
- Security audit with Lynis

**Usage:**
```bash
# Apply security hardening
ansible-playbook -i inventory/hosts.yml playbooks/security-hardening.yml

# Allow additional ports
ansible-playbook -i inventory/hosts.yml playbooks/security-hardening.yml \
  -e "allowed_tcp_ports=[80,443,8080]"

# Custom SSH port
ansible-playbook -i inventory/hosts.yml playbooks/security-hardening.yml \
  -e "ssh_port=2222"
```

### 5. Health Check (`health-check.yml`)

Comprehensive system health monitoring and reporting.

**Features:**
- Disk usage monitoring with thresholds
- Memory and swap usage analysis
- CPU load average checking
- Process monitoring
- Failed services detection
- Docker container status
- Network connectivity check
- Generates detailed report

**Usage:**
```bash
# Run health check
ansible-playbook -i inventory/hosts.yml playbooks/health-check.yml

# Custom thresholds
ansible-playbook -i inventory/hosts.yml playbooks/health-check.yml \
  -e "disk_warning_threshold=70" \
  -e "disk_critical_threshold=85"
```

## 🎯 Using with Ansible Semaphore

### Adding to Semaphore

1. **Create a Repository** in Semaphore pointing to your git repo
2. **Add Inventory**: Upload or link your [inventory/hosts.yml](inventory/hosts.yml)
3. **Create Environment**: Add any required variables
4. **Create Tasks**: Add each playbook as a task template

### Recommended Semaphore Templates

| Template Name | Playbook | Schedule | Description |
|--------------|----------|----------|-------------|
| Daily Updates | [system-updates.yml](playbooks/system-updates.yml) | Daily 2 AM | Automatic system updates |
| Container Updates | [container-update.yml](playbooks/container-update.yml) | Weekly | Update Docker containers |
| Health Check | [health-check.yml](playbooks/health-check.yml) | Hourly | Monitor system health |
| Security Audit | [security-hardening.yml](playbooks/security-hardening.yml) | Monthly | Security hardening check |
| New Server Setup | [server-setup.yml](playbooks/server-setup.yml) | On-demand | Initialize new servers |

## 🔧 Customization

### Common Variables

You can override variables using `-e` flag or by editing playbooks:

```bash
# Example: Run updates with auto-reboot
ansible-playbook -i inventory/hosts.yml playbooks/system-updates.yml \
  -e "auto_reboot=yes" \
  -e "reboot_timeout=300"
```

### SSH Configuration

If using custom SSH keys or ports:

```yaml
# In inventory/hosts.yml
all:
  vars:
    ansible_ssh_private_key_file: ~/.ssh/custom_key
    ansible_port: 2222
```

## 🛡️ Security Considerations

1. **Never commit passwords or keys** to version control
2. Use **Ansible Vault** for sensitive data:
   ```bash
   ansible-vault encrypt inventory/hosts.yml
   ansible-playbook --ask-vault-pass -i inventory/hosts.yml playbooks/system-updates.yml
   ```
3. Always test playbooks in **development environment** first
4. Use `--check` mode for dry runs:
   ```bash
   ansible-playbook -i inventory/hosts.yml playbooks/security-hardening.yml --check
   ```

## 📊 Best Practices

1. **Limit Scope**: Use `--limit` to target specific servers
   ```bash
   ansible-playbook -i inventory/hosts.yml playbooks/system-updates.yml --limit production
   ```

2. **Check Mode**: Always test with `--check` first
   ```bash
   ansible-playbook -i inventory/hosts.yml playbooks/server-setup.yml --check
   ```

3. **Tag Usage**: Run specific sections using tags (if implemented)
   ```bash
   ansible-playbook -i inventory/hosts.yml playbooks/security-hardening.yml --tags ssh
   ```

4. **Verbose Output**: Use `-v`, `-vv`, or `-vvv` for debugging
   ```bash
   ansible-playbook -i inventory/hosts.yml playbooks/health-check.yml -v
   ```

## 🔍 Troubleshooting

### Connection Issues

```bash
# Test connectivity
ansible all -i inventory/hosts.yml -m ping

# Check SSH connection
ssh -i ~/.ssh/id_rsa ubuntu@192.168.1.10

# Verbose ansible connection
ansible all -i inventory/hosts.yml -m ping -vvv
```

### Permission Issues

```bash
# Test sudo access
ansible all -i inventory/hosts.yml -m shell -a "sudo whoami" --become

# Specify become password
ansible-playbook -i inventory/hosts.yml playbooks/system-updates.yml --ask-become-pass
```

### Python Not Found

```bash
# Specify Python interpreter
ansible-playbook -i inventory/hosts.yml playbooks/system-updates.yml \
  -e "ansible_python_interpreter=/usr/bin/python3"
```

## 📝 Maintenance Schedule Recommendation

| Task | Frequency | Playbook |
|------|-----------|----------|
| System Updates | Daily or Weekly | [system-updates.yml](playbooks/system-updates.yml) |
| Security Patches | Weekly | [system-updates.yml](playbooks/system-updates.yml) |
| Container Updates | Weekly | [container-update.yml](playbooks/container-update.yml) |
| Health Checks | Hourly or Daily | [health-check.yml](playbooks/health-check.yml) |
| Security Audit | Monthly | [security-hardening.yml](playbooks/security-hardening.yml) |
| Full Server Backup | Weekly | (Create custom backup playbook) |

## 🤝 Contributing

Feel free to customize these playbooks for your specific needs. Common additions:
- Backup automation playbooks
- Application deployment scripts
- Database maintenance tasks
- Log rotation and cleanup
- Certificate renewal automation

## 📄 License

These playbooks are provided as-is for your server management needs. Customize and use freely.

## ⚠️ Disclaimer

Always test in a development environment before running on production servers. These playbooks perform system-level changes that could affect server availability.

---

**Happy Automating! 🚀**
