# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Base role — SSH hardening, UFW firewall, user creation, kernel and filesystem hardening
- Syslog role — rsyslog forwarding to remote server via TCP/UDP
- Fail2ban role — SSH brute-force protection with UFW ban action
- Unattended-upgrades role — automatic security patching
- NTP role — chrony time synchronisation and timezone configuration
- Docker role — Docker CE and Compose plugin from official repository
- Swap role — configurable swap file with swappiness tuning
- Logrotate role — custom log rotation rules with compression
- MOTD role — informative login banner with system stats
- Users role — additional accounts with SSH keys and optional sudo
- Auditd role — CIS-inspired audit rules for security monitoring
- Node Exporter role — Prometheus metrics exporter as systemd service
- DNS role — systemd-resolved with DNS-over-TLS and DNSSEC support
- ClamAV role — antivirus with scheduled filesystem scans
- Lynis role — automated weekly security auditing
- Tuning role — sysctl performance parameters and system limits
- Caddy role — reverse proxy with automatic HTTPS and gzip
- WireGuard role — VPN server with multi-peer support and NAT
- Journald role — persistent logging with size and retention limits
- Timezone role — locale and timezone configuration
- Comprehensive README with full variable reference
- Security policy (SECURITY.md)
- Contributing guidelines (CONTRIBUTING.md)
- Code of Conduct (CODE_OF_CONDUCT.md)
