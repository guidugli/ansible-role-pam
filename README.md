[![CI](https://github.com/guidugli/ansible-role-pam/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-pam/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/release/guidugli/ansible-role-pam?sort=semver)](https://github.com/guidugli/ansible-role-pam/releases)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.pam-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/pam/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

# Ansible Role: pam

Install and configure PAM hardening controls for supported Linux systems. Debian and Ubuntu systems are managed through `pam-auth-update` profiles plus centralized `/etc/security` configuration. Fedora and Red Hat-family systems continue to use authselect and native PAM configuration paths.

## Requirements

- Ansible Core 2.17.x for the provided development toolchain.
- Debian and Ubuntu targets use `pam-auth-update` and package-managed profiles from `/usr/share/pam-configs/`.
- Fedora and Red Hat-family targets use authselect when `authselect_enabled` evaluates to `true`.
- Run the role with external privilege, for example `become: true`, when targeting real hosts.

## Variables

A `null` default means the variable is intentionally optional and the matching setting is not rendered unless the user sets a value.

| Variable | Type | Default | Description |
| --- | --- | --- | --- |
| `pam_auth_update_enabled` | bool | `true` | Manage Debian and Ubuntu PAM through `pam-auth-update`. |
| `pam_auth_update_profiles` | list(string) | `['unix', 'faillock', 'faillock_notify', 'pwquality', 'pwhistory']` | Profiles enabled by `pam-auth-update`. |
| `authselect_enabled` | bool | derived | Enables authselect management on supported Red Hat/Fedora-family hosts. |
| `authselect_create_profile` | bool | `false` | Creates a custom authselect profile when enabled. |
| `authselect_custom_profile_name` | string | `site-profile` | Custom authselect profile name. |
| `authselect_base_profile` | string | `sssd` | Base authselect profile used to create a custom profile. |
| `authselect_select_profile` | string | `cis_local` | Existing or role-provided profile selected when not creating a custom profile. |
| `authselect_options` | list(string) | `['with-faillock', 'without-nullok']` | Authselect features passed during profile selection. |
| `auth_su_group` | string | `admin` | Group whose members are allowed to use `su`. Root is added to this group. |
| `auth_remember` | int | `24` | Number of previous passwords remembered by pwhistory. |
| `auth_pwhistory_enforce_for_root` | bool | `true` | Enforce password history for root. |
| `auth_pwhistory_use_authtok` | bool | `true` | Enable `use_authtok` for pwhistory. |
| `auth_faillock_dir` | string | `null` | Optional faillock failure-record directory. |
| `auth_deny_after` | int | `5` | Failed login attempts before account lockout. |
| `auth_fail_interval` | int | `900` | Failure-counting interval in seconds. |
| `auth_unlock_time` | int | `900` | Seconds before automatic unlock. |
| `auth_even_deny_root` | bool | `false` | Allows root account lockout when enabled. |
| `auth_root_unlock_time` | int | `null` | Optional root-specific unlock time. |
| `auth_admin_group` | string | `null` | Optional faillock administrative group. |
| `auth_retry` | int | `3` | Number of password prompt retries. |
| `auth_min_length` | int | `14` | Minimum password length. |
| `auth_difok` | int | `2` | Minimum number of changed characters from the old password. |
| `auth_dcredit` | int | `-1` | Digit credit, or minimum digits when negative. |
| `auth_ucredit` | int | `-1` | Uppercase credit, or minimum uppercase characters when negative. |
| `auth_ocredit` | int | `-1` | Other-character credit, or minimum special characters when negative. |
| `auth_lcredit` | int | `-1` | Lowercase credit, or minimum lowercase characters when negative. |
| `auth_maxrepeat` | int | `3` | Maximum allowed consecutive same characters. |
| `auth_maxsequence` | int | `3` | Maximum allowed monotonic character sequence length. |
| `auth_dictcheck` | int | `1` | Enables dictionary checking in pwquality. |
| `auth_enforcing` | int | `1` | Reject passwords that fail quality checks. |
| `auth_enforce_for_root` | bool | `true` | Enforces password quality controls for root password changes. |
| `auth_local_users_only` | bool | `true` | Applies local-user-only behavior where supported by PAM modules. |
| `auth_minclass` | int | `null` | Optional minimum number of character classes. |
| `auth_maxclassrepeat` | int | `null` | Optional maximum consecutive characters from the same class. |
| `auth_gecoscheck` | int | `null` | Optional GECOS password-content check flag. |
| `auth_usercheck` | int | `null` | Optional user-name password-content check flag. |
| `auth_usersubstr` | int | `null` | Optional minimum user-name substring length checked in passwords. |
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
        auth_deny_after: 5
        auth_unlock_time: 900
        auth_min_length: 14
        auth_remember: 24
```

## Molecule Testing

```bash
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

## Execution Notes

- **Privilege model:** the role does not set `become`, `become_user`, or `become_method`. Use external privilege such as `become: true` when managing real hosts.
- **Debian/Ubuntu behavior:** the role installs required PAM packages, creates CIS-oriented `pam-auth-update` profiles, updates `/etc/security` configuration, and runs `pam-auth-update` only when generated common PAM files need refreshing.
- **Fedora/Red Hat behavior:** the role keeps authselect-based management for platforms where `authselect_enabled` evaluates to `true`.
- **Container behavior:** Molecule containers run as root, so the shared converge playbook does not need to enforce privilege.
- **Systemd behavior:** the role does not manage services directly. Use the systemd Molecule scenario when validating environments where PAM session behavior relies on systemd-capable containers.
