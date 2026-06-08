# Linux Setup and Configuration

Ansible playbook for provisioning and hardening Linux hosts. Deploys a secure baseline configuration with SSH key authentication, firewall rules, and kernel hardening — plus a set of optional extras that can be toggled on or off via a simple configuration file.

## Table of Contents

- [Requirements](#requirements)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [Roles](#roles)
  - [Base (always applied)](#base-always-applied)
  - [Syslog Forwarding (optional)](#syslog-forwarding-optional)
  - [Fail2ban (optional)](#fail2ban-optional)
  - [Unattended Upgrades (optional)](#unattended-upgrades-optional)
  - [NTP (optional)](#ntp-optional)
  - [Docker (optional)](#docker-optional)
- [Usage](#usage)
- [Security Considerations](#security-considerations)
- [Contributing](#contributing)
- [Licence](#licence)

## Requirements

- Ansible >= 2.14
- Python 3.8+
- Target hosts running Debian/Ubuntu (apt-based)
- SSH access to target hosts (root or a user with sudo)

Install required Ansible collections:

```bash
ansible-galaxy collection install ansible.posix community.general
```

## Quick Start

1. Clone the repository:

   ```bash
   git clone https://github.com/<your-username>/Linux-Setup-and-Configuration.git
   cd Linux-Setup-and-Configuration
   ```

2. Copy the example environment file and populate it with your values:

   ```bash
   cp env.example.yml env.yml
   ```

3. Edit `inventory.ini` with your target host(s):

   ```ini
   [servers]
   myserver ansible_host=192.168.1.100 ansible_user=root
   ```

4. Run the playbook:

   ```bash
   ansible-playbook site.yml
   ```

## Configuration

All configuration lives in a single file: **`env.yml`** (your local copy, gitignored).

The committed **`env.example.yml`** serves as the reference template. It contains sensible defaults and documents every available option. Copy it to `env.yml` and adjust to taste.

### Key Principles

- `env.example.yml` — committed to the repository. Contains safe defaults and full documentation. **Never put secrets here.**
- `env.yml` — your local copy. Gitignored. Contains your actual SSH keys, IP addresses, and any secrets.
- The playbook reads only from `env.yml` at runtime.

### Variable Reference

| Variable | Default | Description |
|----------|---------|-------------|
| `target_user` | `deploy` | Non-root user to create for SSH access |
| `ssh_port` | `22` | SSH daemon listen port |
| `ssh_permit_root_login` | `no` | Whether root may log in via SSH |
| `ssh_password_authentication` | `no` | Whether password auth is permitted |
| `ssh_authorised_keys` | `[]` | List of public keys to deploy |
| `firewall_enabled` | `true` | Whether to configure UFW |
| `firewall_allowed_tcp_ports` | `[22]` | TCP ports to allow inbound |
| `firewall_allowed_udp_ports` | `[]` | UDP ports to allow inbound |
| `enable_syslog` | `false` | Enable remote syslog forwarding |
| `enable_fail2ban` | `true` | Enable fail2ban intrusion prevention |
| `enable_unattended_upgrades` | `true` | Enable automatic security updates |
| `enable_ntp` | `true` | Enable NTP time synchronisation |
| `enable_docker` | `false` | Install Docker CE |
| `syslog_server` | `logs.example.com` | Remote syslog destination |
| `syslog_port` | `514` | Remote syslog port |
| `syslog_protocol` | `tcp` | Transport protocol (tcp/udp) |
| `fail2ban_bantime` | `3600` | Ban duration in seconds |
| `fail2ban_findtime` | `600` | Window for counting failures |
| `fail2ban_maxretry` | `5` | Failures before ban |
| `ntp_servers` | UK pool | List of NTP server addresses |
| `ntp_timezone` | `Europe/London` | System timezone |
| `docker_users` | `[deploy]` | Users to add to the docker group |

## Project Structure

```
.
├── ansible.cfg              # Ansible configuration
├── env.example.yml          # Example config (committed)
├── env.yml                  # Your local config (gitignored)
├── inventory.ini            # Host inventory
├── site.yml                 # Main playbook
├── .gitignore
└── roles/
    ├── base/                # SSH hardening, firewall, packages
    │   ├── handlers/
    │   ├── tasks/
    │   └── templates/
    ├── syslog/              # Remote syslog forwarding
    │   ├── handlers/
    │   ├── tasks/
    │   └── templates/
    ├── fail2ban/            # Intrusion prevention
    │   ├── handlers/
    │   ├── tasks/
    │   └── templates/
    ├── unattended_upgrades/ # Automatic security patches
    │   ├── tasks/
    │   └── templates/
    ├── ntp/                 # Time synchronisation
    │   ├── handlers/
    │   ├── tasks/
    │   └── templates/
    └── docker/              # Docker CE installation
        └── tasks/
```

## Roles

### Base (always applied)

The base role is **mandatory** and runs on every deployment. It handles:

- **Package installation** — essential utilities (curl, wget, vim, htop, git, etc.)
- **User creation** — creates the deploy user with sudo privileges
- **SSH key deployment** — installs your public keys for the deploy user
- **SSH hardening** — deploys a locked-down `sshd_config`:
  - Disables password authentication
  - Disables root login
  - Limits authentication attempts
  - Disables X11 forwarding, TCP forwarding, and agent forwarding
  - Sets aggressive timeouts
- **Firewall (UFW)** — default deny inbound, allow only specified ports
- **Kernel hardening** — sysctl parameters to mitigate common network attacks
- **Filesystem hardening** — disables unused/legacy filesystem modules

### Syslog Forwarding (optional)

Installs rsyslog and configures it to forward all logs to a remote syslog server. Useful for centralised logging with Graylog, ELK, or a managed SIEM.

Enable with:
```yaml
enable_syslog: true
syslog_server: "logs.yourcompany.com"
syslog_port: 514
syslog_protocol: "tcp"
```

The template uses `@@` for TCP or `@` for UDP (handled by the protocol setting).

### Fail2ban (optional)

Installs and configures fail2ban to monitor SSH authentication logs and temporarily ban IP addresses that exhibit brute-force behaviour.

Enable with:
```yaml
enable_fail2ban: true
fail2ban_bantime: 3600
fail2ban_maxretry: 5
```

Uses UFW as the ban action for seamless integration with the firewall.

### Unattended Upgrades (optional)

Configures the system to automatically install security patches from the distribution's security repositories. Does **not** auto-reboot by default.

Enable with:
```yaml
enable_unattended_upgrades: true
```

### NTP (optional)

Installs chrony and configures it to synchronise time against specified NTP servers. Defaults to the UK NTP pool. Also sets the system timezone.

Enable with:
```yaml
enable_ntp: true
ntp_timezone: "Europe/London"
ntp_servers:
  - "0.uk.pool.ntp.org"
  - "1.uk.pool.ntp.org"
```

### Docker (optional)

Installs Docker CE from the official Docker repository, along with the Docker Compose plugin. Adds specified users to the `docker` group.

Enable with:
```yaml
enable_docker: true
docker_users:
  - deploy
```

## Usage

### Full deployment

```bash
ansible-playbook site.yml
```

### Target specific hosts

```bash
ansible-playbook site.yml --limit myserver
```

### Dry run (check mode)

```bash
ansible-playbook site.yml --check --diff
```

### Run only specific roles using tags

You may also target individual roles:

```bash
ansible-playbook site.yml --tags base
ansible-playbook site.yml --tags syslog
```

### Override variables at runtime

```bash
ansible-playbook site.yml -e "ssh_port=2222"
```

## Security Considerations

- **Never commit `env.yml`** — it contains your SSH keys and potentially sensitive configuration. The `.gitignore` ensures this cannot happen accidentally.
- **Use Ansible Vault** for additional secrets if needed:
  ```bash
  ansible-vault encrypt env.yml
  ansible-playbook site.yml --ask-vault-pass
  ```
- **Test in a staging environment** before applying to production hosts.
- **Review the sshd_config** template carefully — the defaults are strict and will lock out password-based access immediately.
- **Ensure you have console access** to target hosts in case SSH configuration locks you out.

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-new-role`)
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

Please ensure all YAML passes `ansible-lint` before submitting.

## Licence

MIT
