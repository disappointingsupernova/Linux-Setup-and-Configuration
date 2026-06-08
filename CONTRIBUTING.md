# Contributing

Contributions are welcome! Whether it's a bug fix, new role, documentation improvement, or feature request — all input is appreciated.

## How to Contribute

1. **Fork** the repository
2. **Create a feature branch** from `main`:
   ```bash
   git checkout -b feature/my-new-role
   ```
3. **Make your changes** — keep commits small and logical
4. **Test your changes** against a target host (or use `--check --diff` mode)
5. **Ensure YAML linting passes**:
   ```bash
   ansible-lint site.yml
   yamllint .
   ```
6. **Push** to your fork and open a **Pull Request**

## Guidelines

### Code Style

- Follow existing patterns in the repository
- Use fully qualified collection names (e.g. `ansible.builtin.apt`, not `apt`)
- Use `ansible.builtin` and `ansible.posix` modules where possible
- Templates must include the `# {{ ansible_managed }}` header
- Use British English in comments and documentation

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add new role for X
fix: correct template variable in sshd_config
docs: update README with new variable reference
chore: update .gitignore
```

### New Roles

When adding a new role:

1. Create the role under `roles/<role_name>/`
2. Add an `enable_<role_name>` toggle variable to `env.example.yml`
3. Wire the role into `site.yml` with a `when` conditional and a tag
4. Document the role in `README.md` (following the existing format)
5. Keep defaults sensible and safe

### Pull Requests

- Keep PRs focused — one role or one fix per PR
- Provide a clear description of what the PR does and why
- Reference any related issues
- Ensure the PR does not introduce secrets, credentials, or PII

## Reporting Bugs

Open a [GitHub Issue](../../issues) with:

- A clear title
- Steps to reproduce
- Expected vs actual behaviour
- Ansible version and target OS

## Security Vulnerabilities

Please do **not** open a public issue for security vulnerabilities. See [SECURITY.md](SECURITY.md) for responsible disclosure instructions.

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold a welcoming and inclusive environment.
