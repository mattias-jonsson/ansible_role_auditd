Ansible Role: ansible_role_auditd
=========

Installs and configures auditd on the following Linux distributions:

<ul>
<li> Enterprise Linux 7 (RHEL/CentOS/etc.)
<li> Enterprise Linux 8 (RHEL/CentOS/etc.)
<li> Debian 11
</ul>

Requirements
------------

This role has no dependencies besides what is included in Ansible Core.

Role Variables
--------------

Available variables are listed below, along with default values where applicable (see `defaults/main.yml`):

    ansible_role_auditd_options

Key/value list of configuration options for auditd.

    ansible_role_auditd_options

List of rules foir auditd.  

Dependencies
------------

None.

Example Playbook
----------------

    - hosts: servers

      vars:
        ansible_role_auditd_options:
        freq: '60'

        ansible_role_auditd_rules: 
            -a always,exit -F arch=b32 -S adjtimex,settimeofday,stime -F key=time-change
            -a always,exit -F arch=b64 -S adjtimex,settimeofday -F key=time-change
            -a always,exit -F arch=b32 -S clock_settime -F a0=0x0 -F key=time-change
            -a always,exit -F arch=b64 -S clock_settime -F a0=0x0 -F key=time-change
            -w /etc/localtime -p wa -k time-change
            -w /etc/group -p wa -k identity
            -w /etc/passwd -p wa -k identity
            -w /etc/gshadow -p wa -k identity
            -w /etc/shadow -p wa -k identity
            -w /etc/security/opasswd -p wa -k identity
            -a always,exit -F arch=b32 -S sethostname,setdomainname -F key=system-locale
            -a always,exit -F arch=b64 -S sethostname,setdomainname -F key=system-locale
            -w /etc/issue -p wa -k system-locale
            -w /etc/issue.net -p wa -k system-locale
            -w /etc/hosts -p wa -k system-locale
            -w /etc/hostname -p wa -k system-locale
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
            -w /etc/sudoers -p wa -k actions
            -w /etc/sudoers.d/ -p wa -k actions

      roles:
         - ansible_role_auditd

License
-------

MIT

Author Information
------------------

Mattias Jonsson