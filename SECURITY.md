# Security Policy

## Supported Versions

| Version | Supported          |
|---------|--------------------|
| main    | :white_check_mark: |

## Reporting a Vulnerability

If you discover a security vulnerability within this project, please report it responsibly.

**Do not open a public issue.**

Instead, please email: **<your-email@example.com>**

Include as much detail as possible:

- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

### Response Timeline

- **Acknowledgement** — within 48 hours
- **Initial assessment** — within 5 working days
- **Fix or mitigation** — as soon as reasonably practicable

### Disclosure

We follow coordinated disclosure. Once a fix is released, we will credit the reporter (unless they prefer to remain anonymous) and publish an advisory if appropriate.

## Scope

This policy covers the Ansible playbook code, templates, and configuration in this repository. It does **not** cover:

- Third-party packages installed by the roles (e.g. Docker, ClamAV, Caddy)
- Upstream vulnerabilities in those packages
- Misconfiguration resulting from incorrect values in your local `env.yml`

## Best Practices

If you are using this project, please:

- Keep your `env.yml` out of version control (the `.gitignore` handles this)
- Use Ansible Vault for any secrets
- Regularly update the roles and pinned versions
- Test changes in a staging environment before applying to production
