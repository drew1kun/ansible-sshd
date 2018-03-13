Ansible role: sshd
=========

[![MIT licensed][mit-badge]][mit-link]
[![Galaxy Role][role-badge]][galaxy-link]

Cross-platform ansible role for hardening ssh daemon configuration

Requirements
------------

NOTE: Role requires Fact Gathering by ansible!

One of the following OS (or deriviatives):
 - Debian | Ubuntu
 - CentOS
 - MacOS


[OPTIONAL]

If on MacOS the openssh from [Homebrew][homebrew] is intended to be used, then install Homebrew.
This can be done via galaxy role:

    ansible-galaxy install drew-kun.homebrew

And include it in the playbook:

    roles:
        - drew-kun.homebrew

NOTE1: As ssh is essential service for ansible, this role must be included in the playbook at the end position. No other roles should follow after otherwise the playbook will fail.

NOTE2: in order to manage multiple users, the sshd on the managed host must allow enough sessions.
Paramiko module spits out the Warning and Error.
Warning: No handlers could be found for logger "paramiko.transport"
Error: 'Failed to open session'
Solution: set 'MaxSessions 4' (or higher)in sshd_config before provisioning

Role Variables
--------------

**defaults/main.yml:**

| Variable | Description | Default |
|----------|-------------|---------|
| **sshd_port** | Port ssh daemon to listen on | `22` |
| **sshd_users[]** | List of users to configure ssh keys for and allow in etc/sshd_config | see [`defaults/main.yml`](defaults/main.yml) |
| **sshd_root_login** | Allow ssh root login? No - recommended. If yes - then only passwordless (pubkey) root login will be configured | `no` |

MacOS-Specific:

| Variable | Description | Default |
|----------|-------------|---------|
| **sshd_from_homebrew** | Configure openssh daemon from homebrew instead of using the system's native? was useful for older versions oh MacOSX. High Sierra includes the latest openssh. At the moment of publishing this role, so not necessary to use homebrew. | `no` |

**vars/*:**

Distribution-Specific:

| Variable | Description | Default |
|----------|-------------|---------|
| **sshd_bin_path** | Path to sshd system binary | <ul><li>homebrew.yml: `/usr/local/sbin/sshd`</li><li>Linux,BSD (main.yml): `/usr/sbin/sshd`</li></ul> |
| **sshd_cfg_dir** | sshd configuration file path (different for Homebrew) | <ul><li>homebrew.yml: `/usr/local/etc/ssh`</li><li>Linux,BSD (main.yml): `/etc/ssh`</li></ul> |
| **sshd_cfg_user** | Owner of sshd configuration files | <ul><li>homebrew.yml: `"{{ ansible_user_id }}"`</li><li>Linux,BSD (main.yml): `root`</li></ul> |
| **sshd_cfg_group** | Group of sshd configuration files | <ul><li>Darwin.yml: `wheel`</li><li>homebrew.yml: `admin`</li><li>Linux,BSD (main.yml): `root`</li></ul> |
| **sshd_sftp_server** | Path to sftp-server | <ul><li>Darwin.yml: `/usr/libexec/sftp-server`</li><li>Linux,BSD (main.yml): `/usr/lib/openssh/sftp-server`</li></ul> |

MacOS-Specific:

| Variable | Description | Default |
|----------|-------------|---------|
| **sshd_wrapper_path** | Path to system's sshd-keygen-wrapper to be set in ssh.plist | <ul><li>Darwin.yml: `/usr/libexec/sshd-keygen-wrapper`</li><li>homebrew.yml: `/usr/local/sbin/sshd`</li></ul> |

Dependencies
------------

None

Example Playbook
----------------

    - hosts: macos_hosts
      gather_facts: yes
      roles:
         - { role: drew-kun.sshd, sshd_port: 2222, sshd_from_homebrew: yes}

License
-------

MIT

Author Information
------------------

Andrew Shagayev | [e-mail](mailto:drewshg@gmail.com)

[role-badge]: https://img.shields.io/badge/role-drew--kun.sshd-green.svg
[galaxy-link]: https://galaxy.ansible.com/drew-kun/sshd/
[mit-badge]: https://img.shields.io/badge/license-MIT-blue.svg
[mit-link]: https://raw.githubusercontent.com/drew-kun/ansible-sshd/master/LICENSE
[homebrew]: http://brew.sh/

