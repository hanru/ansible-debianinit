debianinit: Debian Server Initialization
========================================

[![Build Status](https://travis-ci.org/hanru/ansible-debianinit.svg?branch=master)](https://travis-ci.org/hanru/ansible-debianinit)

This Ansible role configures a minimal Debian server that is ready for future use. Please note that currently only Debian Jessie (8.x), Debian Stretch (9.x) and Ubuntu Xenial (16.04) are guaranteed to work.

Requirements
------------

* The SSH user on remote server has root privilege.
* Python 2.x is installed on remote server

Role Variables
--------------

    di_ssh_port: 22

SSH deamon listening port. Putting SSH on a non-default port and ignoring other security measures are considered security through obscurity.

SSH listen on port 22 by default. It's not advised to change this setting.

    di_ssh_password_authentication: 'no'

Whether SSH password authentication is enabled.

Password authentication is disabled by default. Unless you have specific requirements, this setting should not be changed.

    di_ssh_permit_root_login: 'without-password'

Whether `root` user is allowed to login. If you run ansible as `root`, then `without-password` is a sane choice. Otherwise this can be safely set to `no`.

By default this setting is `without-password`.

    di_ssh_allow_users: '{{ ansible_user }}'

Which users are allowed to login via SSH. All allowed users are defined as a string, separated by spaces.

By default only the SSH user performs this role is allowed to login.

    di_system_removed_packages:
      - apache2
      - bind9
      - rpcbind
      - samba
      - sendmail
      - snmp

A list of packages to be removed (purged). By default some packages are removed as in the above block.

    di_system_installed_packages:
      - apt-transport-https
      - bzip2
      - ca-certificates
      - cron
      - curl
      - dbus
      - dnsutils
      - haveged
      - less
      - logrotate
      - mtr-tiny
      - openssl
      - rsyslog
      - screen
      - sudo
      - time
      - vim-tiny
      - vnstat
      - wget
      - whiptail

A list of packages to be installed. By default some packages are installed as in the above block.

    di_system_fail2ban_enabled: yes

Whether to install fail2ban, a service to ban bad hosts according to certain rules. After installation, brute-force SSH login attempts are automatically blocked.

By default fail2ban is installed.

    di_system_timezone: 'UTC'

Server timezone. Unless you have specific requirements, `UTC` is the preferred and default timezone.

    di_system_timesync_enabled: yes

Whether to enable the time sync service. This service is provided by systemd and is lighter than ntp service. On Xen, KVM virtual servers and dedicated servers, this service should be enabled. On OpenVZ virtual servers, this service may not work.

By default this service is enabled.

    di_system_unattended_upgrades_enabled: no

Whether to enable the unattended upgrades which will automatically upgrade the system on a daily basis. Please note that servers should still be taken care of even if this feature is enabled. For example, some new packages (esp. new Linux kernel) won't take effect unless the server is rebooted.

By default unattended upgrades is disabled. It can be safely enabled on a standard system.

    di_add_users: []

A list of users to be created on the server. Every user **must** have three fields defined: `name`, `password` and `shell`. See example section for more on how to define new users.

By default, no users are created.

    di_sudoers_password: []

A list of users who can execute `sudo` command after providing their passwords.

By default no users are added to this list.

    di_sudoers_passwordless: []

A list of users who can execute `sudo` without providing their passwords. Since `sudo` runs a command as root, it's inherently insecure if password is not required before running. This setting is better left blank for important servers.

By default no users are added to this list.

    di_ufw_enabled: no

Whether to install ufw, a human-friendly iptables front-end. By enabling ufw, the sane default policies (allow outgoing, deny incoming) are set and TCP on SSH port is allowed. If the server has further usage, such as http, you will need to further tweak the `di_ufw_rules` variable, see below and example playbook.

By default ufw is not installed.

    di_ufw_rules:
      - { rule: allow, from: any, to: any, port: '{{ di_ssh_port }}', proto: tcp }

A list of user-defined ufw rules. These rules will be applied when ufw is enabled. Each rule **must** have five fields.

1. `rule` defines the type of the rule. Possible values are `allow`, `deny` and `reject`.
2. `from` defines source IP address. Set `from` to `any` if there's no source IP restriction.
3. `to` defines destination IP address. Set `to` to `any` if there's no destination IP restriction.
4. `port` defines destination port.
5. `proto` defines network protocol. Possible values are `tcp`, `udp` and `any`.

Example section shows how to define ufw rules. Note that if you need to change `di_ufw_rules`, the first rule which allows SSH port must be kept or you may risk locking yourself out of your server. This role is only capable of simple ufw rules. For more complex rules, you may have to define them manually.

By default TCP on SSH port is allowed. No additional ufw rules are defined.

Dependencies
------------

This role has no dependencies.

Example Playbook
----------------

When *root* is running the playbook:

    - hosts: testservers
      vars:
        di_add_users:
          - name: test
            password: randompassword
            shell: /bin/bash
          - name: git
            password: anotherrandompassword
            shell: /usr/bin/git-shell
        di_ssh_allow_users: 'root test git'
        di_sudoers_password:
          - test
        di_ufw_enabled: yes
        di_ufw_rules:
          - { rule: allow, from: any, to: any, port: '{{ di_ssh_port }}', proto: tcp }
          - { rule: allow, from: any, to: any, port: 80, proto: tcp }
          - { rule: allow, from: any, to: any, port: 443, proto: tcp }
          - { rule: deny, from: 192.168.1.0/24, to: any, port: 53, proto: any }
      roles:
        - { role: hanru.debianinit }

When a user with *sudo* privilege is running the playbook:

    - hosts: testservers
      vars:
        ...
      roles:
        - { role: hanru.debianinit, become: yes }

License
-------

MIT

Reference
---------

This role is inspired by [My First 5 Minutes On A Server](https://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-for-linux-servers). When developing this role, I learned a lot from the following Ansible playbooks/roles.

* https://github.com/cturner80/digital-ocean-ansible 
* https://github.com/geerlingguy/ansible-role-security