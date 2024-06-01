Ansible Role: pam
=========

An Ansible Role that install and configure pam on RHEL/CentOS, Fedora and Debian/Ubuntu. In RedHat/CentOS and Fedora, it uses authselect to configure pam.

If using authselect, the role will create 3 additional profiles: cis_local, cis_winbind and cis_sssd that follows the recommendations from CIS. These profiles
can be selected or can be used as based for a new custom profile that allow users to do further changes.

Requirements
------------

No requirements.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

    authselect_enabled: "{{ _authselect_enabled }}"

If true, tasks will be executed to configure authselect settings; if false, authselect settings will not be changed.

    authselect_create_profile: false

If true, the role will create a custom profile that can be further customized by the user.
If false, use one of the existing authselect profiles or the profiles created by the role (cis_local, cis_winbing, cis_sssd).

    authselect_profile_name: 'site-profile'

Name of the custom profile to be created

    authselect_base_profile: sssd

Base of the custom profile. Allowed values can be listed with authselect list.

    authselect_options:
      - with-faillock
      - without-nullok

If not defined, it will use 'with-faillock' and 'without-nullok' as default

    auth_su_group: admin

Restrict su access to users on the specified group.

    auth_remember: 5

Number of passwords to 'remember' so user cannot set new password as the old passwords

    auth_deny_after: 5

Number of login attempts before locking user

    auth_unlock_time: 900

Time, in seconds, to wait before unlocking a locked user


    auth_retry: 3

Tries before sending back a failure to pwquality

    auth_min_length: 14

Password minimum length


    auth_dcredit: -1  # Require at least 1 digit
    auth_ucredit: -1  # Require at least 1 uppercase
    auth_ocredit: -1  # Require at least 1 special char
    auth_lcredit: -1  # Require at least 1 lower case char

Password complexity rules

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    _authselect_enabled:

This variable is set to true on RedHat 8.x, CentOS 8.x and Fedora systems and false for other systems.

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: guidugli.pam }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
