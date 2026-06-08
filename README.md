# Linux Setup and Configuration

Ansible playbook for provisioning and hardening Linux hosts. Deploys a secure baseline configuration with SSH key authentication, firewall rules, and kernel hardening — plus a set of optional extras that can be toggled on or off via a simple configuration file.

## Table of Contents

- [Requirements](#requirements)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [Roles](#roles)
  - [Base (always applied)](#base-always-applied)
  - [Additional Users (optional)](#additional-users-optional)
  - [Timezone & Locale (optional)](#timezone--locale-optional)
  - [Syslog Forwarding (optional)](#syslog-forwarding-optional)
  - [Journald (optional)](#journald-optional)
  - [Fail2ban (optional)](#fail2ban-optional)
  - [Unattended Upgrades (optional)](#unattended-upgrades-optional)
  - [NTP (optional)](#ntp-optional)
  - [Swap (optional)](#swap-optional)
  - [Logrotate (optional)](#logrotate-optional)
  - [MOTD (optional)](#motd-optional)
  - [Auditd (optional)](#auditd-optional)
  - [Node Exporter (optional)](#node-exporter-optional)
  - [DNS (optional)](#dns-optional)
  - [ClamAV (optional)](#clamav-optional)
  - [Lynis (optional)](#lynis-optional)
  - [System Tuning (optional)](#system-tuning-optional)
  - [Caddy (optional)](#caddy-optional)
  - [WireGuard (optional)](#wireguard-optional)
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
| `additional_users` | `[]` | Extra user accounts to create |
| `enable_timezone` | `true` | Set system locale and timezone |
| `enable_syslog` | `false` | Enable remote syslog forwarding |
| `enable_journald` | `true` | Configure journald log persistence and limits |
| `enable_fail2ban` | `true` | Enable fail2ban intrusion prevention |
| `enable_unattended_upgrades` | `true` | Enable automatic security updates |
| `enable_ntp` | `true` | Enable NTP time synchronisation |
| `enable_swap` | `false` | Create and enable a swap file |
| `enable_logrotate` | `true` | Configure log rotation rules |
| `enable_motd` | `true` | Deploy a custom login banner |
| `enable_auditd` | `false` | Enable Linux audit framework |
| `enable_node_exporter` | `false` | Install Prometheus node exporter |
| `enable_dns` | `false` | Configure systemd-resolved DNS |
| `enable_clamav` | `false` | Install ClamAV antivirus |
| `enable_lynis` | `false` | Install Lynis security auditing |
| `enable_tuning` | `false` | Apply system performance tuning |
| `enable_caddy` | `false` | Install Caddy reverse proxy |
| `enable_wireguard` | `false` | Install and configure WireGuard VPN |
| `enable_docker` | `false` | Install Docker CE |
| `syslog_server` | `logs.example.com` | Remote syslog destination |
| `syslog_port` | `514` | Remote syslog port |
| `syslog_protocol` | `tcp` | Transport protocol (tcp/udp) |
| `journald_max_use` | `500M` | Maximum disk usage for journal |
| `journald_max_retention` | `1month` | Maximum retention period |
| `fail2ban_bantime` | `3600` | Ban duration in seconds |
| `fail2ban_findtime` | `600` | Window for counting failures |
| `fail2ban_maxretry` | `5` | Failures before ban |
| `ntp_servers` | UK pool | List of NTP server addresses |
| `ntp_timezone` | `Europe/London` | System timezone |
| `swap_size` | `2G` | Swap file size |
| `swap_swappiness` | `10` | Kernel swappiness value |
| `node_exporter_version` | `1.7.0` | Node exporter release version |
| `node_exporter_port` | `9100` | Metrics listen port |
| `dns_nameservers` | `[1.1.1.1, 9.9.9.9]` | Primary DNS resolvers |
| `dns_over_tls` | `false` | Enable DNS-over-TLS |
| `clamav_scan_schedule` | `0 2 * * 0` | Cron schedule for virus scans |
| `caddy_sites` | `[]` | List of site definitions |
| `wireguard_interface` | `wg0` | WireGuard interface name |
| `wireguard_port` | `51820` | WireGuard listen port |
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
    ├── users/               # Additional user accounts
    │   └── tasks/
    ├── timezone/            # Locale and timezone
    │   └── tasks/
    ├── syslog/              # Remote syslog forwarding
    │   ├── handlers/
    │   ├── tasks/
    │   └── templates/
    ├── journald/            # Journal log management
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
    ├── swap/                # Swap file management
    │   └── tasks/
    ├── logrotate/           # Log rotation
    │   ├── tasks/
    │   └── templates/
    ├── motd/                # Login banner
    │   ├── tasks/
    │   └── templates/
    ├── auditd/              # Linux audit framework
    │   ├── handlers/
    │   ├── tasks/
    │   └── templates/
    ├── node_exporter/       # Prometheus metrics
    │   ├── handlers/
    │   ├── tasks/
    │   └── templates/
    ├── dns/                 # DNS resolver configuration
    │   ├── tasks/
    │   └── templates/
    ├── clamav/              # Antivirus scanning
    │   ├── tasks/
    │   └── templates/
    ├── lynis/               # Security auditing
    │   └── tasks/
    ├── tuning/              # System performance tuning
    │   └── tasks/
    ├── caddy/               # Reverse proxy with auto-TLS
    │   ├── handlers/
    │   ├── tasks/
    │   └── templates/
    ├── wireguard/           # VPN
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

### Additional Users (optional)

Creates extra user accounts beyond the primary deploy user. Supports per-user SSH key deployment, group membership, and passwordless sudo configuration.

Enable by defining the `additional_users` list:
```yaml
additional_users:
  - name: alice
    groups: "sudo"
    sudo: true
    ssh_keys:
      - "ssh-ed25519 AAAA... alice-key"
  - name: bob
    ssh_keys:
      - "ssh-ed25519 AAAA... bob-key"
```

### Timezone & Locale (optional)

Sets the system timezone and generates the desired locale. Useful for ensuring consistent timestamps in logs and cron jobs.

Enable with:
```yaml
enable_timezone: true
system_timezone: "Europe/London"
system_locale: "en_GB.UTF-8"
```

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

### Journald (optional)

Configures systemd-journald for persistent storage with size limits and retention policies. Prevents journal logs from consuming all available disk space.

Enable with:
```yaml
enable_journald: true
journald_max_use: "500M"
journald_max_file_size: "50M"
journald_max_retention: "1month"
```

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

### Swap (optional)

Creates a swap file, enables it, adds it to fstab for persistence across reboots, and sets a conservative swappiness value.

Enable with:
```yaml
enable_swap: true
swap_size: "2G"
swap_swappiness: "10"
```

### Logrotate (optional)

Configures custom log rotation rules. Define multiple rules, each with its own path, frequency, retention count, and optional post-rotation commands.

Enable with:
```yaml
enable_logrotate: true
logrotate_rules:
  - path: "/var/log/syslog"
    frequency: daily
    rotate: 14
    compress: true
    postrotate: "systemctl reload rsyslog 2>/dev/null || true"
```

### MOTD (optional)

Deploys a clean, informative login banner displaying hostname, OS, kernel version, uptime, memory usage, disk usage, and IP address. Removes the default Ubuntu clutter.

Enable with:
```yaml
enable_motd: true
motd_warning_message: "Authorised users only. Activity is monitored."
```

### Auditd (optional)

Installs the Linux audit daemon with a comprehensive ruleset inspired by CIS benchmarks. Monitors:

- Authentication and PAM configuration changes
- User/group file modifications
- Sudoers changes
- SSH configuration changes
- Network configuration changes
- Cron modifications
- All commands executed by root
- Kernel module loading

The ruleset is made immutable — a reboot is required to alter it.

Enable with:
```yaml
enable_auditd: true
```

### Node Exporter (optional)

Installs Prometheus Node Exporter as a systemd service, running under a dedicated system user. Exposes hardware and OS metrics for scraping by a Prometheus instance.

Enable with:
```yaml
enable_node_exporter: true
node_exporter_version: "1.7.0"
node_exporter_port: 9100
```

Remember to allow the port through the firewall if scraping remotely:
```yaml
firewall_allowed_tcp_ports:
  - 22
  - 9100
```

### DNS (optional)

Configures systemd-resolved with custom nameservers. Supports DNS-over-TLS and DNSSEC for enhanced privacy and security.

Enable with:
```yaml
enable_dns: true
dns_nameservers:
  - "1.1.1.1"
  - "9.9.9.9"
dns_over_tls: true
dns_dnssec: true
```

### ClamAV (optional)

Installs ClamAV antivirus with automatic signature updates via freshclam. Deploys a cron job for scheduled filesystem scans.

Enable with:
```yaml
enable_clamav: true
clamav_scan_schedule: "0 2 * * 0"
clamav_scan_paths: "/home /var /tmp"
```

### Lynis (optional)

Installs Lynis, an open-source security auditing tool. Configures a weekly cron job that performs a full system audit and logs the results.

Enable with:
```yaml
enable_lynis: true
lynis_cron_hour: "3"
lynis_cron_weekday: "1"
```

### System Tuning (optional)

Applies performance-oriented sysctl parameters and raises system limits (file descriptors, process counts, network backlog). Suitable for servers running high-traffic applications or databases.

Enable with:
```yaml
enable_tuning: true
tuning_file_max: "2097152"
tuning_somaxconn: "65535"
tuning_nofile_soft: "65535"
tuning_nofile_hard: "65535"
```

### Caddy (optional)

Installs Caddy as a reverse proxy with automatic HTTPS via Let's Encrypt. Supports multiple site definitions with per-site reverse proxy targets, static file serving, custom headers, and gzip compression.

Enable with:
```yaml
enable_caddy: true
caddy_sites:
  - domain: "app.example.com"
    reverse_proxy: "localhost:8080"
    tls_email: "you@example.com"
    headers:
      X-Frame-Options: "DENY"
      X-Content-Type-Options: "nosniff"
```

Remember to allow ports 80 and 443:
```yaml
firewall_allowed_tcp_ports:
  - 22
  - 80
  - 443
```

### WireGuard (optional)

Installs and configures WireGuard as a VPN server or peer. Enables IP forwarding and deploys the interface configuration with support for multiple peers, NAT masquerading, and persistent keepalives.

Enable with:
```yaml
enable_wireguard: true
wireguard_interface: wg0
wireguard_address: "10.0.0.1/24"
wireguard_port: 51820
wireguard_private_key: "<your-private-key>"
wireguard_peers:
  - public_key: "<peer-public-key>"
    allowed_ips: "10.0.0.2/32"
    persistent_keepalive: 25
```

Remember to allow the WireGuard port:
```yaml
firewall_allowed_udp_ports:
  - 51820
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

```bash
ansible-playbook site.yml --tags base
ansible-playbook site.yml --tags fail2ban
ansible-playbook site.yml --tags "ntp,swap,motd"
```

### Override variables at runtime

```bash
ansible-playbook site.yml -e "ssh_port=2222"
ansible-playbook site.yml -e "enable_docker=true"
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
- **WireGuard private keys** should only ever exist in your local `env.yml` — never in `env.example.yml`.

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-new-role`)
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

Please ensure all YAML passes `ansible-lint` before submitting.

## Licence

MIT
