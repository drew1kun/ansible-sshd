Ansible role: sshd
=========

[![MIT licensed][mit-badge]][mit-link]
[![Galaxy Role][role-badge]][galaxy-link]

Cross-platform ansible role for hardening ssh daemon configuration using PKI and certificates

The role does the following:

 - Adds the specified user ssh public keys to ssh server's `authorized_hosts` file, if no certificate setup is used (`sshd_certs` set to `no`).
 - Creates secure `sshd_cofnig` which adds a bunch of security tweaks like disable password authentication, root login,
    enable pki authentication and only allow users, which are specified in `sshd_users`. See deatials in
    [*templates/sshd_config.j2*](templates/sshd_config.j2)

Certifiacte Setup:

The following will be done when variable `sshd_certs` set to `yes` (default):

 - The role will create all keypairs, retrieve any public keys from their private portions and generate necessary
    certificates only if needed (complete idempotency). If you have any keys you want to use (user private keys, Host or User
    CA private keys) just specify them in corresponding variables.
 - It will look for HostCA and UserCA which are set in `sshd_host_ca_key` and `sshd_user_ca_key` and if the files are not there, then
    they will be generated using algorithm set in `sshd_algo`.
 - The host keys encrypted with `sshd_algo` algorithm will be generated and signed with HostCA if they are not already
    there.
 - The specified user keys will be generated (again - only if needed) for each of specified user (var `sshd_users`) and
    signed with UserCA.
 - The local machine (ansible control host) will be configured to trust all host keys signed with specified Host CA
    (so no more annoying host keys fingerprint verifications - all guaranteed with SSH trusted Host CA)
 - The remote ssh server (ansible managed host) will be configured to trust all user keys signed with specified User CA

NOTE: For security reasons the User and Host CA fingerprint crosscheck is implemented, so if you are using your old CA
keys, make sure you have specified the correct full path to their private keys (`sshd_host_ca_key` and `sshd_user_ca_key`)
and their fingerprints as **SHA512** (`sshd_host_ca_key_fpr` and `sshd_user_ca_key_fpr`) hashes. You just need private
keys.

Public keys will be retrieved from the private keys of specified CA automatically if that CA private key exists.

NOTE: if any existing private key files are encrypted (protected by passphrase), the role will ask your to enter the
passphrase for that private key in order to retrieve the public key. If you enter it wrong, the role will fail (and the
handlers will not be executed). This is ansible issue which makes ansible not completely idempotend at the moment.

If there is no specific User or Host CA you want to use, then the specified CA files (UserCA and HostCA) will be created automatically.

To provide complete non-interactivness User and Host CA private key files will be UNENCRYPTED (no passphrase protection)
so please back up them and store in a secure place (like [KeepassXC][keepass] db etc..). By default the files will be
stored in */etc/ssh/* (or in */etc* for MacOS 10.12 or earlier), owned by root and protected by Unix permissions.
The corresponding notifications wil pop up upon successful role execution.

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

    ansible-galaxy install drew1kun.homebrew

And include it in the playbook:

    roles:
        - drew1kun.homebrew

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
| **sshd_algo** | Algorithm for SSH keys and CAs, which will be generated if there none already exist | `ed25519` |
| **sshd_port** | Port ssh daemon to listen on | `22` |
| **sshd_users: []** | List of users to configure ssh keys for and allow in etc/sshd_config | see [`defaults/main.yml`](defaults/main.yml) |
| **sshd_root_login** | Allow ssh root login? No - recommended. If yes - then only passwordless (pubkey) root login will be configured | `no` |


**Certifiacte Setup Variables:**

| Variable | Description | Default |
|----------|-------------|---------|
| **sshd_certs** | Harden sshd config using Host and User certificates | `yes` |
| **sshd_host_ca_key_fpr** | Host CA fingerprint for cross check | `SHA512:JW9LfG8wfWkPhQZ4n3ljYKijKpOH/oNuJYjtKbzHvWkjrqseTdobkOskjqAHqj8WAfpCkmOp/d1buSyHkK+Ipw` |
| **sshd_user_ca_key_fpr** | User CA fingerprint for cross check | `SHA512:UwXpDQfzlsX1H6RNTZd1xDgoOe4X/SzvGSD9H2r8gAnn4+/ZuBDnqNp4guasNnESYEzUdV2kmgOiFwpQ5NCtBQ` |
| **sshd_host_ca_key** | Full path to Host CA | `"{{ sshd_cfg_dir }}/id_{{ sshd_algo }}-HostCA"` |
| **sshd_user_ca_key** | Full path to User CA | `"{{ sshd_cfg_dir }}/id_{{ sshd_algo }}-UserCA"` |
| **sshd_host_ca_comment** | Full path to User CA | `"{{ sshd_cfg_dir }}/id_{{ sshd_algo }}-UserCA"` |
| **sshd_user_ca_comment** | Full path to User CA | `"{{ sshd_cfg_dir }}/id_{{ sshd_algo }}-UserCA"` |
| **sshd_cert_id** | Host Identifier, used in Logging and Revoking the Certificate | `domain.local` |
| **sshd_certlogger** | The log file that holds the certificate signing facts | `/usr/local/var/log/cert-signing.log` |

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

Role Usage
----------

###Simple PKI Setup
 - Add


###Certificate Setup

If you want to use the existing CA keys:

 - Add your HostCA and UserCA private keys to the sshd config directory on your ansible management host

     By default: `sshd_config: /etc/ssh`

   If you do not want to use the existing CA keys, then just skip this step. HostCA and UserCA will be generated
   automatically during the role execution. It may ask you for passphrases, so prepare them and just Copy/Paste in order
   the role not to fail.

 - Specify the path to them in `sshd_host_ca_key`:

     By default:

    `sshd_host_ca_key: /etc/ssh/id_ed25519-HostCA`
    `sshd_user_ca_key: /etc/ssh/id_ed25519-UserCA`

 - Run the play!

Example Playbook
----------------

```yaml
- hosts: macos_hosts
  gather_facts: yes
  roles:
  - { role: drew1kun.sshd, sshd_port: 2222, sshd_from_homebrew: yes, sshd_cert_id: mycooldomain.com }
```

License
-------

MIT

Author Information
------------------

Andrew Shagayev | [e-mail](mailto:drewshg@gmail.com)

[role-badge]: https://img.shields.io/badge/role-drew--kun.sshd-green.svg
[galaxy-link]: https://galaxy.ansible.com/drew1kun/sshd/
[mit-badge]: https://img.shields.io/badge/license-MIT-blue.svg
[mit-link]: https://raw.githubusercontent.com/drew1kun/ansible-sshd/master/LICENSE
[homebrew]: http://brew.sh/
[keepass]: https://keepassxc.org/

