Role Name
=========

Role to download and install Jython. Virtualenv is being preinstalled
and a link to a virtualenv executable will be created under the name
`jvirtualenv`.

Requirements
------------

* Java JRE

Role Variables
--------------

The following variables will change the behavior of this role (default
values are shown below):

```yaml
# Jython version number
jython_version: 2.7.0

# Installation directory for the Jython-versionnumber distribution and
# the Jython symlink
jython_install_dir: /opt

# Directory where a link to the Jython and virtualenv (as jvirtualenv)
# executable should be created
jython_bin_dir: /usr/local/bin

# Directory to store files downloaded for Jython installation
jython_download_dir: "{{ x_ansible_download_dir | default(ansible_env.HOME + '~/.ansible/tmp/downloads') }}"
```

Example Playbook
----------------

Including an example of how to use your role (for instance, with
variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: baztian.jython, jython_version: "2.7.0" }

License
-------

MIT
