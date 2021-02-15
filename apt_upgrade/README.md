Role Name
=========

Role to configure and upgrade base/official apt sources for ubuntu and mint.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
    - vars:
        official_apt_sources_replacements:
        - regexp: http://packages.linuxmint.com
          replace: http://ftp.fau.de/mint/packages
        - regexp: http://archive.ubuntu.com/ubuntu
          replace: http://ftp.fau.de/ubuntu
      roles:
         - role: baztian.apt_upgrade

License
-------

GPLv3
