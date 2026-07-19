[![CI](https://github.com/guidugli/ansible-role-pam/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-pam/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/release/guidugli/ansible-role-pam?sort=semver)](https://github.com/guidugli/ansible-role-pam/tags)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.pam-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/pam/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

# Ansible Role: pam

Install and configure PAM hardening controls on supported Linux distributions.

This role provides a CIS-oriented PAM implementation using:

- `pam-auth-update` on Debian and Ubuntu.
- `authselect` vendor profiles on Fedora and Red Hat-family systems.
- Centralized policy management under `/etc/security`.
- Password quality controls through `pam_pwquality`.
- Password history controls through `pam_pwhistory`.
- Account lockout controls through `pam_faillock`.
- Restricted `su` access through a configurable administrative group.

The role is designed to keep PAM stack construction separate from policy values. PAM stack placement is handled by `pam-auth-update` or `authselect`, while policy values are managed through `/etc/security` configuration files.

## Requirements

- Ansible Core 2.17.x for the development toolchain.
- `containers.podman` collection for Molecule container management.
- `community.general` collection where required by the role dependencies.
- External privilege escalation, such as `become: true`, when running against real hosts.

### Debian and Ubuntu

Debian and Ubuntu targets use `pam-auth-update` and package-managed profiles under:

```text
/usr/share/pam-configs/
```

The role installs the base PAM packages:

```text
libpam-runtime
libpam-modules
libpam-pwquality
```

Ubuntu targets additionally install:

```text
cracklib-runtime
```

### Fedora and Red Hat-family systems

Fedora and Red Hat-family targets use `authselect` when `authselect_enabled` evaluates to `true`.

The role installs and uses:

```text
authselect
libpwquality
```

## Supported Platforms

The supported platform matrix is aligned with the Molecule matrix and generated Galaxy metadata.

| Distribution | Versions |
| --- | --- |
| Fedora | 43, 44 |
| Ubuntu | 24.04, 26.04 |
| Debian | 12, 13 |

## PAM Architecture

### Debian and Ubuntu

Debian and Ubuntu systems are managed through `pam-auth-update`.

The role enables these profiles:

```text
unix
faillock
faillock_notify
pwquality
pwhistory
```

The generated PAM stack files are managed by `pam-auth-update`:

```text
/etc/pam.d/common-auth
/etc/pam.d/common-account
/etc/pam.d/common-password
/etc/pam.d/common-session
```

The role does not directly template those `common-*` files.

Policy values are centralized in:

```text
/etc/security/faillock.conf
/etc/security/pwhistory.conf
/etc/security/pwquality.conf.d/50-cis-pwquality.conf
```

### Fedora and Red Hat-family systems

Fedora and Red Hat-family systems are managed through authselect.

The role creates vendor authselect profiles:

```text
cis_local
cis_sssd
cis_winbind
```

under:

```text
/usr/share/authselect/vendor/
```

The authselect profile templates control:

- PAM module placement.
- PAM stack ordering.
- Authselect feature toggles.
- Identity-provider-specific stack behavior.

Policy values remain centralized in:

```text
/etc/security/faillock.conf
/etc/security/pwhistory.conf
/etc/security/pwquality.conf
```

This keeps the active PAM stack aligned with authselect while avoiding embedded policy values in the PAM stack templates.

## Variables

A `null` default means the setting is intentionally optional and is not rendered unless explicitly configured.

| Variable | Type | Default | Description |
| --- | --- | --- | --- |
| `pam_auth_update_enabled` | bool | `true` | Enable `pam-auth-update` management on Debian and Ubuntu. |
| `pam_auth_update_profiles` | list(string) | `['unix', 'faillock', 'faillock_notify', 'pwquality', 'pwhistory']` | PAM profiles enabled through `pam-auth-update`. |
| `authselect_enabled` | bool | derived | Enable authselect management on supported Fedora and Red Hat-family systems. |
| `authselect_create_profile` | bool | `false` | Create a custom authselect profile from `authselect_base_profile`. |
| `authselect_custom_profile_name` | string | `site-profile` | Name of the custom authselect profile to create. |
| `authselect_base_profile` | string | `sssd` | Base authselect profile used when creating a custom profile. |
| `authselect_select_profile` | string | `cis_local` | Existing or role-provided authselect profile to select when custom profile creation is disabled. |
| `authselect_options` | list(string) | `['with-faillock', 'without-nullok']` | Authselect feature options applied during profile selection. |
| `auth_su_group` | string | `admin` | Group whose members are allowed to use `su`. Root is added to this group. |
| `auth_remember` | int | `24` | Number of previous passwords remembered by `pam_pwhistory`. |
| `auth_remember_file` | string | `null` | Optional password history file path. |
| `auth_pwhistory_enforce_for_root` | bool | `true` | Enforce password history for root. |
| `auth_pwhistory_use_authtok` | bool | `true` | Enable `use_authtok` for password history stack behavior. |
| `auth_faillock_dir` | string | `null` | Optional faillock failure-record directory. |
| `auth_audit` | bool | `false` | Enable unknown-user logging for faillock. |
| `auth_silent` | bool | `false` | Suppress faillock informative messages. |
| `auth_no_log_info` | bool | `false` | Suppress faillock informational syslog messages. |
| `auth_deny_after` | int | `5` | Failed login attempts before account lockout. |
| `auth_fail_interval` | int | `900` | Failure-counting interval in seconds. |
| `auth_unlock_time` | int | `900` | Seconds before automatic unlock. |
| `auth_even_deny_root` | bool | `false` | Allow root account lockout when enabled. |
| `auth_root_unlock_time` | int | `null` | Optional root-specific unlock time. |
| `auth_admin_group` | string | `null` | Optional faillock administrative group. |
| `auth_retry` | int | `3` | Password prompt retry count. |
| `auth_min_length` | int | `14` | Minimum password length. |
| `auth_dcredit` | int | `-1` | Digit credit, or minimum digits when negative. |
| `auth_ucredit` | int | `-1` | Uppercase credit, or minimum uppercase characters when negative. |
| `auth_ocredit` | int | `-1` | Other-character credit, or minimum special characters when negative. |
| `auth_lcredit` | int | `-1` | Lowercase credit, or minimum lowercase characters when negative. |
| `auth_enforce_for_root` | bool | `true` | Enforce password quality checks for root. |
| `auth_local_users_only` | bool | `true` | Apply local-user-only behavior where supported by PAM modules. |
| `auth_difok` | int | `2` | Minimum number of changed characters from the old password. |
| `auth_minclass` | int | `null` | Optional minimum number of required character classes. |
| `auth_maxrepeat` | int | `3` | Maximum number of allowed consecutive same characters. |
| `auth_maxsequence` | int | `3` | Maximum length of monotonic character sequences. |
| `auth_maxclassrepeat` | int | `null` | Optional maximum number of consecutive characters from the same class. |
| `auth_gecoscheck` | int | `null` | Optional GECOS password-content check flag. |
| `auth_dictcheck` | int | `1` | Enable dictionary checking in password quality policy. |
| `auth_usercheck` | int | `null` | Optional user-name password-content check flag. |
| `auth_usersubstr` | int | `null` | Optional minimum user-name substring length checked in passwords. |
| `auth_enforcing` | int | `1` | Enforce rejection of passwords that fail quality checks. |
| `auth_dictpath` | string | `null` | Optional alternative cracklib dictionary path. |

See `defaults/main.yml` and `meta/argument_specs.yml` for the complete variable specification.

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

Example Level 2-style root lockout policy:

```yaml
auth_even_deny_root: true
auth_root_unlock_time: 60
```

Example Fedora authselect selection:

```yaml
authselect_select_profile: cis_local
authselect_options:
  - with-faillock
  - without-nullok
  - with-pwhistory
```

## Molecule Testing

Install development dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
```

Run the default scenario:

```bash
molecule test -s default
```

Run the systemd scenario:

```bash
molecule test -s systemd
```

Login to a running container:

```bash
./scripts/login.sh fedora44
```

## Execution Notes

### Privilege model

The role intentionally does not set:

```yaml
become:
become_user:
become_method:
```

Privilege must be controlled by the calling playbook or execution environment.

Use external privilege when running against real systems:

```yaml
become: true
```

### Container behavior

Molecule containers run as root. The shared Molecule converge playbook does not need to enforce privilege escalation inside containers.

### Systemd behavior

The role does not directly manage services. The `systemd` Molecule scenario exists to validate platform behavior in systemd-capable containers.

### Idempotency

The role is designed to be idempotent:

- package installation uses package modules;
- profile and configuration files are managed declaratively;
- `authselect apply-changes` is handled through a handler;
- `pam-auth-update` runs only when PAM regeneration is required;
- centralized `/etc/security` files hold policy values instead of duplicating policy arguments in PAM stack lines.

### Compliance notes

The role is designed around modern PAM hardening practices:

- centralized password quality configuration;
- centralized password history configuration;
- centralized account lockout configuration;
- authselect-managed PAM stacks on Fedora and Red Hat-family systems;
- pam-auth-update-managed PAM stacks on Debian and Ubuntu systems.

Organizations should validate configured values against the benchmark version and profile level they are required to follow before production deployment.

## License

MIT
