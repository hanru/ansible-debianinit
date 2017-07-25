debianinit: Debian Server Initialization
========================================

[![Build Status](https://travis-ci.org/hanru/debianinit.svg?branch=master)](https://travis-ci.org/hanru/debianinit)

This Ansible role configures a minimal Debian server that is ready for future use. Please note that currently only Debian Jessie (8.x) and Ubuntu Xenial (16.04) is guaranteed to work.

Requirements
------------

* SSH login for user `root` is enabled
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

    di_ssh_allow_users: 'root'

Which users are allowed to login via SSH. All allowed users are defined as a string, separated by spaces.

By default only `root` is allowed to login.

    di_system_removed_packages:
      - apache2
      - bind9
      - postfix
      - rpcbind
      - samba
      - sendmail
      - snmp

A list of packages to be removed (purged). By default 7 packages are removed as in the above block.

    di_system_installed_packages:
      - apt-transport-https
      - bzip2
      - ca-certificates
      - cron
      - curl
      - dbus
      - dnsutils
      - less
      - logrotate
      - mtr-tiny
      - openssl
      - rsyslog
      - screen
      - sudo
      - time
      - vim
      - wget
      - whiptail

A list of packages to be installed. By default 18 packages are installed as in the above block.

    di_system_fail2ban_enabled: yes

Whether to install fail2ban, a service to ban bad hosts according to certain rules. After installation, brute-force SSH login attempts are automatically blocked.

By default fail2ban is installed.

    di_system_vnstat_enabled: yes

Whether to install vnstat, a light-weight service to monitor and report network traffic usage.

By default vnstat is installed.

    di_system_timezone: 'UTC'

Server timezone. Unless you have specific requirements, `UTC` is the preferred and default timezone.

    di_system_timesync_enabled: yes

Whether to enable the time sync service. This service is provided by systemd and is lighter than ntp service. On Xen, KVM virtual servers and dedicated servers, this service should be enabled. On OpenVZ virtual servers, this service may not work.

By default this service is enabled.

    di_system_unattended_upgrades_enabled: no

Whether to enable the unattended upgrades which will automatically upgrade the system on a daily basis. Please note that servers should still be taken care of even if this feature is enabled. For example, some new packages (esp. new Linux kernel) won't take effect unless the server is rebooted.

By default unattended upgrades is disabled. It can be safely enabled on a standard system.

    di_add_users: []

A list of users to be created on the server. Every user should have three fields defined: `name`, `password` and `shell`. See example section for more on how to define new users.

By default, no users are created.

    di_sudoers_password: []

A list of users who can execute `sudo` command after providing their passwords.

By default no users are added to this list.

    di_sudoers_passwordless: []

A list of users who can execute `sudo` without providing their passwords. Since `sudo` runs a command as root, it's inherently insecure if password is not required before running. This setting is better left blank for important servers.

By default no users are added to this list.

    di_ufw_enabled: yes

Whether to install ufw, a human-friendly iptables front-end. By enabling ufw, the sane default policies (allow outgoing, deny incoming) are set and TCP on SSH port is allowed. If the server has further usage, such as http, you will need to manually add ufw rules, e.g. allow TCP on port 80 and 443.

By default ufw is installed.

Dependencies
------------

This role has no dependencies.

Example Playbook
----------------

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
        di_ufw_enabled: no
      roles:
         - { role: hanru.debianinit }

License
-------

MIT

Reference
---------

This role is inspired by [My First 5 Minutes On A Server](https://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-for-linux-servers). When developing this role, I learned a lot from the following Ansible playbooks/roles.

* https://github.com/cturner80/digital-ocean-ansible 
* https://github.com/geerlingguy/ansible-role-security