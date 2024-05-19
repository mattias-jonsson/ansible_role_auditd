Ansible Role: auditd
=========

Installs and configures the Linux Auditing System (auditd) on Enterprise Linux 7/8/9 (RHEL/CentOS/etc.) and Debian 11/12.

Requirements
------------

This role has no dependencies besides what is included in Ansible Core.

Role Variables
--------------

Available variables are listed below, along with default values where applicable (see `defaults/main.yml`):

| Variable | Required | Default | Comments |
| -------- | -------- | ------- | -------- |
| `auditd_backup` | No | true | If set to true, backup copies will be created for all configuration files modified during the role execution. |
| `auditd_cleanup` | No | true | Setting this option to true will rename all rule files in the rules.d directory that are not listed in the auditd_rules list, except for 00_default.rules (the distribution’s default rules). The renamed files will have .disabled as their new extension. |
| `auditd_options` | No | [] | A list of key/value pairs to configure additional `auditd` settings. Each pair represents a setting in the `auditd.conf` file, such as `freq`, `action_mail_acct`, etc. Refer to the example playbook below for usage. |
| `auditd_rules` | No | [] | A list of audit rules to be applied. Each rule should be specified as a dictionary with keys such as `name`, `priority`, and `rules`. Refer to the example playbook below for examples. |

Example Playbook
----------------

    - hosts: servers

      vars:
        auditd_backup: false
        auditd_cleanup: true
        auditd_options:
          freq: '60'
        auditd_rules:
          - name: my_custom_rules
            priority: 97
            rules: |
              -a always,exit -F arch=b32 -S adjtimex,settimeofday,stime -F key=time-change
              -a always,exit -F arch=b64 -S adjtimex,settimeofday -F key=time-change
              -a always,exit -F arch=b32 -S clock_settime -F a0=0x0 -F key=time-change
              -a always,exit -F arch=b64 -S clock_settime -F a0=0x0 -F key=time-change
              -a always,exit -F arch=b32 -S sethostname,setdomainname -F key=system-locale
              -a always,exit -F arch=b64 -S sethostname,setdomainname -F key=system-locale
              -a always,exit -F dir=/etc/NetworkManager/ -F perm=wa -F key=system-locale
              -a always,exit -F dir=/etc/selinux/ -F perm=wa -F key=MAC-policy
              -a always,exit -F arch=b32 -S chmod,fchmod,fchmodat -F auid>=1000 -F auid!=unset -F key=perm_mod
              -a always,exit -F arch=b64 -S chmod,fchmod,fchmodat -F auid>=1000 -F auid!=unset -F key=perm_mod
              -a always,exit -F arch=b32 -S lchown,fchown,chown,fchownat -F auid>=1000 -F auid!=unset -F key=perm_mod
              -a always,exit -F arch=b64 -S chown,fchown,lchown,fchownat -F auid>=1000 -F auid!=unset -F key=perm_mod
              -a always,exit -F arch=b32 -S setxattr,lsetxattr,fsetxattr,removexattr,lremovexattr,fremovexattr -F auid>=1000 -F auid!=unset -F key=perm_mod
              -a always,exit -F arch=b64 -S setxattr,lsetxattr,fsetxattr,removexattr,lremovexattr,fremovexattr -F auid>=1000 -F auid!=unset -F key=perm_mod
              -a always,exit -F arch=b32 -S open,creat,truncate,ftruncate,openat,open_by_handle_at -F exit=-EACCES -F auid>=1000 -F auid!=unset -F key=access
              -a always,exit -F arch=b32 -S open,creat,truncate,ftruncate,openat,open_by_handle_at -F exit=-EPERM -F auid>=1000 -F auid!=unset -F key=access
              -a always,exit -F arch=b64 -S open,truncate,ftruncate,creat,openat,open_by_handle_at -F exit=-EACCES -F auid>=1000 -F auid!=unset -F key=access
              -a always,exit -F arch=b64 -S open,truncate,ftruncate,creat,openat,open_by_handle_at -F exit=-EPERM -F auid>=1000 -F auid!=unset -F key=access
              -a always,exit -F arch=b32 -S mount -F auid>=1000 -F auid!=unset -F key=export
              -a always,exit -F arch=b64 -S mount -F auid>=1000 -F auid!=unset -F key=export
              -a always,exit -F arch=b32 -S unlink,unlinkat,rename,renameat -F auid>=1000 -F auid!=unset -F key=delete
              -a always,exit -F arch=b64 -S unlink,unlinkat,rename,renameat -F auid>=1000 -F auid!=unset -F key=delete
          - name: nsa_usgcb
            priority: 99
            rules: |
              #2.6.2.4.1 Records Events that Modify Date and Time Information
              -a always,exit -F arch=b64 -S adjtimex -S settimeofday -S stime -k time-change
              -a always,exit -F arch=b64 -S clock_settime -k time-change
              -w /etc/localtime -p wa -k time-change
              #2.6.2.4.2 Record Events that Modify User/Group Information
              -w /etc/group -p wa -k identity
              -w /etc/passwd -p wa -k identity
              -w /etc/gshadow -p wa -k identity
              -w /etc/shadow -p wa -k identity
              -w /etc/security/opasswd -p wa -k identity
              #2.6.2.4.3 Record Events that Modify the System’s Network Environment
              -a exit,always -F arch=b64 -S sethostname -S setdomainname -k system-locale
              -w /etc/issue -p wa -k system-locale
              -w /etc/issue.net -p wa -k system-locale
              -w /etc/hosts -p wa -k system-locale
              -w /etc/sysconfig/network -p wa -k system-locale
              #2.6.2.4.4 Record Events that Modify the System’s Mandatory Access Controls
              -w /etc/selinux/ -p wa -k MAC-policy
              #2.6.2.4.5 Ensure auditd Collects Logon and Logout Events
              -w /var/log/faillog -p wa -k logins
              -w /var/log/lastlog -p wa -k logins
              #2.6.2.4.6 Ensure auditd Collects Process and Session Initiation Information
              -w /var/run/utmp -p wa -k session
              -w /var/log/btmp -p wa -k session
              -w /var/log/wtmp -p wa -k session
              #2.6.2.4.7 Ensure auditd Collects Discretionary Access Control Permission Modification Events
              -a always,exit -F arch=b64 -S chmod -S fchmod -S fchmodat -F auid>=500 -F auid!=4294967295 -k perm_mod
              -a always,exit -F arch=b64 -S chown -S fchown -S fchownat -S lchown -F auid>=500 -F auid!=4294967295 -k perm_mod
              -a always,exit -F arch=b64 -S setxattr -S lsetxattr -S fsetxattr -S removexattr -S lremovexattr -S fremovexattr -F auid>=500 -F auid!=4294967295 -k perm_mod
              #2.6.2.4.8 Ensure auditd Collects Unauthorized Access Attempts to Files (unsuccessful)
              -a always,exit -F arch=b64 -S creat -S open -S openat -S truncate -S ftruncate -F exit=-EACCES -F auid>=500 -F auid!=4294967295 -k access
              -a always,exit -F arch=b64 -S creat -S open -S openat -S truncate -S ftruncate -F exit=-EPERM -F auid>=500 -F auid!=4294967295 -k access
              #2.6.2.4.9 Ensure auditd Collects Information on the Use of Privileged Commands
              -a always,exit -F path=/bin/ping -F perm=x -F auid>=500 -F auid!=4294967295 -k privileged
              -a always,exit -F path=/bin/umount -F perm=x -F auid>=500 -F auid!=4294967295 -k privileged
              -a always,exit -F path=/bin/mount -F perm=x -F auid>=500 -F auid!=4294967295 -k privileged
              -a always,exit -F path=/bin/su -F perm=x -F auid>=500 -F auid!=4294967295 -k privileged
              -a always,exit -F path=/bin/ping6 -F perm=x -F auid>=500 -F auid!=4294967295 -k privileged
              -a always,exit -F path=/lib/dbus-1/dbus-daemon-launch-helper -F perm=x -F auid>=500 -F auid!=4294967295 -k privileged
              -a always,exit -F path=/sbin/pam_timestamp_check -F perm=x -F auid>=500 -F auid!=4294967295 -k privileged
              -a always,exit -F path=/sbin/unix_chkpwd -F perm=x -F auid>=500 -F auid!=4294967295 -k privileged
              -a always,exit -F path=/usr/sbin/suexec -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/sbin/usernetctl -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/sbin/ccreds_validate -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/sbin/userhelper -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/libexec/openssh/ssh-keysign -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/kerberos/bin/ksu -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/newgrp -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/Xorg -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/rlogin -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/sudoedit -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/at -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/rsh -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/gpasswd -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/kgrantpty -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/crontab -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/sudo -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/staprun -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/rcp -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/passwd -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/chsh -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/chfn -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/chage -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/bin/kpac_dhcp_helper -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              -a always,exit -F path=/usr/lib/nspluginwrapper/plugin-config -F perm=x -F auid>=500 -F auid!=4294967295 -k priveleged
              #2.6.2.4.10 Ensure auditd Collects Information on Exporting to Media (successful)
              -a always,exit -F arch=b64 -S mount -F auid>=500 -F auid!=4294967295 -k export
              #2.6.2.4.11 Ensure auditd Collects Files Deletion Events by User (successful and unsuc-cessful)
              -a always,exit -F arch=b64 -S unlink -S unlinkat -S rename -S renameat -F auid>=500 -F auid!=4294967295 -k delete
              #2.6.2.4.12 Ensure auditd Collects System Administrator Actions
              -w /etc/sudoers -p wa -k actions
              #2.6.2.4.13 Make the auditd Configuration Immutable
              -e 2

      roles:
         - auditd

License
-------

MIT

Author Information
------------------

Mattias Jonsson
