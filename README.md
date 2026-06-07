# Ansible Role: Fail2Ban

[![CI](https://github.com/guidugli/ansible-role-fail2ban/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-fail2ban/actions/workflows/CI.yml)
[![Release](https://github.com/guidugli/ansible-role-fail2ban/actions/workflows/release.yml/badge.svg)](https://github.com/guidugli/ansible-role-fail2ban/actions/workflows/release.yml)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-guidugli.fail2ban-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/fail2ban/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

Install and configure Fail2Ban on Debian, Ubuntu, Fedora, and compatible Red Hat-family systems.

## Features

- Installs the correct Fail2Ban package set per supported platform.
- Manages global Fail2Ban `DEFAULT` jail settings in `jail.local`.
- Manages per-jail settings using structured role variables.
- Supports optional systemd service hardening through a drop-in override.
- Uses automatic Ansible role argument validation through `meta/argument_specs.yml`.
- Keeps semantic validation in `tasks/assert.yml`.
- Avoids hardcoded privilege escalation in role tasks; callers should set `become: true` when needed.

## Supported Platforms

The generated metadata currently targets:

- Ubuntu: `noble`, `jammy`
- Debian: `trixie`, `bookworm`
- Fedora: `43`, `42`

## Requirements

Install the required collections before running the role:

```bash
ansible-galaxy collection install community.general ansible.utils
```

The role manages system packages and services, so the calling play normally needs privilege escalation.

## Role Variables

Defaults are defined in `defaults/main.yml`.

### Service and hardening

```yaml
f2b_manage_service: true
f2b_systemd_secure: false
f2b_systemd_sec_delete_old_log: false
```

`fb_systemd_sec_delete_old_log` is retained as a compatibility alias. Prefer
`f2b_systemd_sec_delete_old_log` in new playbooks.

### Global Fail2Ban defaults

```yaml
f2b_bantime_increment: false
f2b_bantime_rndtime: 120
f2b_bantime_maxtime: 0
f2b_bantime_factor: 1
f2b_bantime_formula: ""
f2b_bantime_multipliers: ""
f2b_bantime_overalljails: false
f2b_ignoreself: true
f2b_ignoreip: "127.0.0.1/8 ::1"
f2b_bantime: "30m"
f2b_findtime: "30m"
f2b_maxretry: 5
f2b_maxmatches: "%(maxretry)s"
f2b_usedns: "warn"
f2b_destemail: "root@localhost"
f2b_sender: "root@localhost"
f2b_mta: "sendmail"
```

Empty optional strings, such as `f2b_bantime_formula`, are treated as unset and are not rendered.
`f2b_bantime_maxtime: 0` is treated as unmanaged.

Only one of `f2b_bantime_formula` or `f2b_bantime_multipliers` may be set.

### Per-jail settings

```yaml
f2b_jails:
  - name: sshd
    settings:
      - key: enabled
        value: "true"
      - key: mode
        value: normal
      - key: port
        value: "22"
      - key: backend
        state: absent
```

Settings are written as provided. Validate option names and values against your target Fail2Ban version.

## Example Playbook

```yaml
---
- name: Configure Fail2Ban
  hosts: all
  become: true
  roles:
    - role: guidugli.fail2ban
      vars:
        f2b_systemd_secure: true
        f2b_bantime_increment: true
        f2b_bantime: "30m"
        f2b_findtime: "30m"
        f2b_maxretry: 5
        f2b_jails:
          - name: sshd
            settings:
              - key: enabled
                value: "true"
              - key: mode
                value: normal
```

## How It Works

1. Ansible automatically validates supported inputs using `meta/argument_specs.yml`.
2. `tasks/assert.yml` performs semantic validation that argument specs cannot fully express.
3. `tasks/fail2ban.yml` installs packages and manages `jail.local`.
4. `tasks/systemd_secure.yml` applies the optional systemd hardening override only when systemd manages `fail2ban.service`.

## Molecule Testing

Run both scenarios when available:

```bash
molecule test -s default
molecule test -s systemd
```

The default scenario should validate package/configuration behavior. The systemd scenario should validate service management and systemd hardening behavior.

## Metadata and Release Workflow

`meta/main.yml` is generated. Update `templates/meta_main.yml.j2` and shared metadata inputs first, then regenerate metadata using the repository generator workflow.

Do not hand-edit generated metadata unless you are intentionally testing the rendered output.

## Repository Structure

```text
defaults/main.yml              Role defaults
handlers/main.yml              Service and systemd handlers
meta/argument_specs.yml        Role input schema
meta/main.yml                  Generated Galaxy metadata
tasks/assert.yml               Semantic validation
tasks/fail2ban.yml             Package and configuration management
tasks/main.yml                 Validation and task dispatch
tasks/systemd_secure.yml       Optional systemd hardening
templates/meta_main.yml.j2     Metadata template
vars/main.yml                  Internal package and option maps
```

## License

MIT

## Author

Carlos Guidugli
