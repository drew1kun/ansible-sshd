Ansible role: sshd
=========

[![MIT licensed][mit-badge]][mit-link]
[![Galaxy Role][role-badge]][galaxy-link]

Cross-platform ansible role for hardening ssh daemon configuration

Requirements
------------

One of the following OS (or deriviatives):
 - Debian | Ubuntu
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

defaults/main.yml:

    sshd_port: 22                       # port ssh daemon to listen on


    sshd_users:                         # list of users to configure ssh keys for and allow in etc/sshd_config
      - username: {{ ansible_user_id }} # username
        key:  "{{ lookup('file', 'pub_keys/id_rsa.pub') }}"  # put public keys named properly or each user to files/pub-keys
        comment:                        # can be left blank or specify the email for example...
      - username: root
        key:  "{{ lookup('file', 'pub_keys/id_rsa.pub') }}"

    sshd_root_login: yes | no           # Allow ssh root login? No - recommended
                                        # If yes - then only passwordless (pubkey) root login will be configured

MacOS-Specific:

    sshd_from_homebrew: yes | no        # Configure openssh daemon from homebrew instead of using the system's native?
                                        # was useful for older versions oh MacOSX. High Sierra includes the latest openssh
                                        # at the moment of publishing this role, so not necessary to use homebrew.

vars/*

Distribution-Specific:

    sshd_bin_path:                      # Path to sshd system binary
    sshd_cfg_dir:                       # sshd configuration file path (different for Homebrew)
    sshd_cfg_user:                      # owner of sshd configuration files
    sshd_cfg_group:                     # group of sshd configuration files
    sshd_sftp_server:                   # path to sftp-server

MacOS-Specific:

    sshd_wrapper_path:                  # Path to system's sshd-keygen-wrapper to be set in ssh.plist

Dependencies
------------

None

Example Playbook
----------------

    - hosts: macos_hosts
      roles:
         - { role: drew-kun.sshd, sshd_port: 2222, sshd_from_homebrew: yes}

License
-------

MIT

Author Information
------------------

Andrew Shagayev

[role-badge]: https://img.shields.io/badge/role-drew--kun.sshd-green.svg
[galaxy-link]: https://galaxy.ansible.com/drew-kun/sshd/
[mit-badge]: https://img.shields.io/badge/license-MIT-blue.svg
[mit-link]: https://raw.githubusercontent.com/drew-kun/ansible-sshd/master/LICENSE
[homebrew]: http://brew.sh/

