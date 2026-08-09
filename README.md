[![CI](https://github.com/guidugli/ansible-role-fail2ban/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-fail2ban/actions/workflows/CI.yml)
![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-fail2ban?label=release)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-guidugli.fail2ban-blue)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/fail2ban/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

## Ansible Role: fail2ban

Install and configure Fail2Ban on Debian, Ubuntu, Fedora, and compatible Red Hat-family systems. The role manages global jail defaults, optional per-jail settings, and optional systemd hardening without declaring privilege escalation inside role tasks.

### Requirements

- Ansible Core 2.16 or newer according to role metadata.
- Linux hosts with a supported package manager and Fail2Ban packages available through the configured repositories.
- The `community.general` collection is required for `community.general.ini_file` and is pinned in `requirements.yml` with a minimum version.
- The `containers.podman` collection is required for Molecule container scenarios and is pinned in `requirements.yml` with a minimum version.
- Root-level permissions are normally required because the role installs packages, writes under `/etc/fail2ban`, optionally writes under `/etc/systemd/system`, and manages the Fail2Ban service. Supply privilege externally in your playbook or automation platform.

### Features

- Installs Fail2Ban packages using platform-specific package mappings.
- Creates and manages `/etc/fail2ban/jail.local` through idempotent INI operations.
- Manages global Fail2Ban `DEFAULT` jail values from role variables.
- Manages optional per-jail settings from structured list input.
- Removes optional DEFAULT settings when their variables are intentionally unset.
- Optionally installs a systemd hardening drop-in for `fail2ban.service` on systemd hosts.
- Uses argument specs for input schema validation and task assertions for semantic validation.
- Keeps privilege escalation under caller control and keeps Molecule scenarios container-aware.

### Supported platforms

The generated metadata targets Fedora, Ubuntu, and Debian. The bundled Molecule matrix includes Ubuntu 26.04 and 24.04, Debian 13 and 12, and Fedora 44 and 43.

### Variables

All public role inputs are defined in `defaults/main.yml` and represented in `meta/argument_specs.yml`.

| Variable | Type | Default | Description |
| --- | --- | --- | --- |
| `f2b_manage_service` | bool | `true` | Enable and start `fail2ban` through the target service manager. Set to `false` in containers or images that do not run a service manager. |
| `f2b_systemd_secure` | bool | `false` | Apply the optional systemd hardening override for `fail2ban.service`. |
| `f2b_systemd_sec_delete_old_log` | bool | `false` | Delete `/var/log/fail2ban.log` after moving the log target to `/var/log/fail2ban/fail2ban.log`. |
| `fb_systemd_sec_delete_old_log` | bool | `false` | Deprecated compatibility alias for `f2b_systemd_sec_delete_old_log`. Prefer the `f2b_` variable in new playbooks. |
| `f2b_bantime_increment` | bool | `false` | Manage Fail2Ban `bantime.increment`. |
| `f2b_bantime_rndtime` | int | `120` | Manage Fail2Ban `bantime.rndtime`, in seconds. |
| `f2b_bantime_maxtime` | int | `0` | Manage Fail2Ban `bantime.maxtime`. A value of `0` leaves this directive absent. |
| `f2b_bantime_factor` | int | `1` | Manage Fail2Ban `bantime.factor`. |
| `f2b_bantime_formula` | string | `""` | Optional Fail2Ban `bantime.formula`. Empty string leaves this directive absent. |
| `f2b_bantime_multipliers` | string | `""` | Optional Fail2Ban `bantime.multipliers`. Empty string leaves this directive absent. |
| `f2b_bantime_overalljails` | bool | `false` | Manage Fail2Ban `bantime.overalljails`. |
| `f2b_ignoreself` | bool | `true` | Manage Fail2Ban `ignoreself`. |
| `f2b_ignoreip` | string | `127.0.0.1/8 ::1` | Space-separated IP addresses, CIDRs, or hostnames ignored by Fail2Ban. |
| `f2b_bantime` | string | `30m` | Manage Fail2Ban `bantime`. Accepted examples include `600`, `30m`, `1h`, `1d`, and `1w`. |
| `f2b_findtime` | string | `30m` | Manage Fail2Ban `findtime`. Accepted examples include `600`, `30m`, `1h`, `1d`, and `1w`. |
| `f2b_maxretry` | int | `5` | Manage Fail2Ban `maxretry`. |
| `f2b_maxmatches` | string | `%(maxretry)s` | Manage Fail2Ban `maxmatches`. |
| `f2b_usedns` | string | `warn` | Manage Fail2Ban `usedns`. Allowed values are `yes`, `warn`, `no`, and `raw`. |
| `f2b_destemail` | string | `root@localhost` | Manage Fail2Ban `destemail`. |
| `f2b_sender` | string | `root@localhost` | Manage Fail2Ban `sender`. |
| `f2b_mta` | string | `sendmail` | Manage Fail2Ban `mta`. |
| `f2b_jails` | list(dict) | `[]` | Per-jail settings to manage in `jail.local`. Each item requires `name` and `settings`. Each setting requires `key`, optional `value`, and optional `state` of `present` or `absent`. |

`f2b_bantime_formula` and `f2b_bantime_multipliers` are mutually exclusive. The role fails during validation if both are set.

Example `f2b_jails` value:

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

### Example playbook

```yaml
---
- name: Configure Fail2Ban
  hosts: all
  become: true
  roles:
    - role: guidugli.fail2ban
      vars:
        f2b_systemd_secure: true
        f2b_systemd_sec_delete_old_log: true
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

### Molecule testing instructions

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

A convenience script is also present:

```bash
./scripts/run_local.sh
```

### Execution notes

- **Privilege model:** the role never declares `become`, `become_user`, or `become_method`. Use `become: true` at the play, inventory, or automation-controller level for real hosts that require privilege to install packages, write `/etc` content, write systemd drop-ins, or manage services.
- **Container behavior:** Molecule containers execute with root-level access and scenario playbooks do not rely on role-level privilege escalation. The role avoids embedding escalation so the execution context controls privilege consistently.
- **Systemd behavior:** optional hardening tasks are guarded with `ansible_facts['service_mgr'] == 'systemd'`. The hardening drop-in is applied only when service facts show `fail2ban.service` is sourced from systemd. Service management remains caller-controlled through `f2b_manage_service`.
- **Idempotency:** package, file, copy, service, and INI modules are used instead of shell commands in role tasks. Optional INI values are removed deterministically when unset.

### Release workflow

Generated repository metadata is refreshed through the shared generator scripts. Update generator inputs, especially `templates/meta_main.yml.j2` and `molecule/shared/vars.yml`, then run:

```bash
./scripts/update_release_metadata.sh
./scripts/release.sh --version v1.2.0 --message "Release v1.2.0"
```

`meta/main.yml`, generated Molecule inventories, `.github/`, and `scripts/` are generator-controlled or template-controlled areas and should not be hand-edited during role modernization.

### License

MIT

### Author

Carlos Guidugli
