[![CI](https://github.com/guidugli/ansible-role-pam/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-pam/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/release/guidugli/ansible-role-pam?sort=semver)](https://github.com/guidugli/ansible-role-pam/tags)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.pam-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/pam/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

# Ansible Role: pam

Install and configure PAM hardening controls for supported Linux systems. The role manages password-quality policy, password-history policy, account lockout parameters, optional authselect profile content, and group-based access restrictions for `su`.

## Requirements

- Ansible Core 2.17.x for the provided development toolchain.
- Supported targets: Debian, Ubuntu, Fedora, and Red Hat-family hosts represented by the Molecule platform matrix.
- On Debian/Ubuntu, the role installs `libpam-pwquality`.
- On authselect-capable Red Hat-family systems, the role installs `authselect` when authselect management is enabled.
- Collections are declared in `requirements.yml` with minimum versions.

## Variables

A `null` default means the variable is intentionally optional and the matching template line is not rendered unless the user sets a value.

| Variable | Type | Default | Description |
| --- | --- | --- | --- |
| `authselect_enabled` | bool | `{{ _authselect_enabled }}` | Enables authselect management on supported distributions. |
| `authselect_create_profile` | bool | `false` | Creates a custom authselect profile when enabled. |
| `authselect_custom_profile_name` | string | `site-profile` | Custom authselect profile name. |
| `authselect_base_profile` | string | `sssd` | Base authselect profile used to create a custom profile. |
| `authselect_select_profile` | string | `cis_local` | Existing or role-provided profile selected when not creating a custom profile. |
| `authselect_options` | list(string) | `['with-faillock', 'without-nullok']` | Authselect features passed during profile selection. |
| `auth_su_group` | string | `admin` | Group whose members are allowed to use `su`. Root is added to this group. |
| `auth_remember` | int | `5` | Number of previous passwords remembered by pwhistory. |
| `auth_remember_file` | string | `null` | Optional password history file path. |
| `auth_faillock_dir` | string | `null` | Optional faillock failure-record directory. Falls back to `/var/run/faillock`. |
| `auth_audit` | bool | `false` | Enables unknown-user logging in faillock. |
| `auth_silent` | bool | `false` | Suppresses faillock informative messages. |
| `auth_no_log_info` | bool | `false` | Suppresses faillock informational syslog messages. |
| `auth_deny_after` | int | `5` | Failed login attempts before account lockout. |
| `auth_fail_interval` | int | `900` | Failure-counting interval in seconds. |
| `auth_unlock_time` | int | `600` | Seconds before automatic unlock. |
| `auth_even_deny_root` | bool | `false` | Allows root account lockout when enabled. |
| `auth_root_unlock_time` | int | `null` | Optional root-specific unlock time. |
| `auth_admin_group` | string | `null` | Optional faillock administrative group. |
| `auth_retry` | int | `3` | Number of password prompt retries. |
| `auth_min_length` | int | `14` | Minimum password length. |
| `auth_dcredit` | int | `-1` | Digit credit, or minimum digits when negative. |
| `auth_ucredit` | int | `-1` | Uppercase credit, or minimum uppercase characters when negative. |
| `auth_ocredit` | int | `-1` | Other-character credit, or minimum special characters when negative. |
| `auth_lcredit` | int | `-1` | Lowercase credit, or minimum lowercase characters when negative. |
| `auth_enforce_for_root` | bool | `true` | Enforces password quality/history controls for root password changes. |
| `auth_local_users_only` | bool | `true` | Applies local-user-only behavior where supported by the PAM modules. |
| `auth_difok` | int | `null` | Optional minimum number of character changes from the old password. |
| `auth_maxrepeat` | int | `null` | Optional maximum allowed consecutive same characters. |
| `auth_maxclassrepeat` | int | `null` | Optional maximum consecutive characters from the same class. |
| `auth_gecoscheck` | int | `null` | Optional GECOS password-content check flag. |
| `auth_dictcheck` | int | `null` | Optional dictionary check flag. |
| `auth_usercheck` | int | `null` | Optional user-name password-content check flag. |
| `auth_usersubstr` | int | `null` | Optional minimum user-name substring length checked in passwords. |
| `auth_enforcing` | int | `null` | Optional password quality enforcement flag. |
| `auth_dictpath` | string | `null` | Optional alternative cracklib dictionary path. |

## Example Playbook

```yaml
---
- name: Configure PAM hardening
  hosts: linux
  become: true
  roles:
    - role: guidugli.pam
      vars:
        auth_su_group: wheel
        auth_deny_after: 5
        auth_unlock_time: 600
        auth_min_length: 14
        auth_maxrepeat: 3
```

## Molecule Testing

```bash
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

## Execution Notes

- **Privilege model:** the role does not set `become`, `become_user`, or `become_method` in role tasks. Use `become: true` in real playbooks when writing to privileged paths or managing packages/users/groups.
- **Container behavior:** Molecule containers run as root and the shared converge playbook uses external execution context rather than role-level escalation.
- **Systemd behavior:** the role does not manage services directly. Use the systemd Molecule scenario when validating PAM paths that depend on systemd-capable containers.
- **Generated metadata:** `meta/main.yml` is generated from `templates/meta_main.yml.j2`. Update the template and regenerate metadata through the repository tooling.
