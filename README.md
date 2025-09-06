Podman
======

This role sets up [Podman](https://podman.io/) on the target system.

Requirements
------------

This role has no special requirements on the controller.

However, Ansible needs to become an unprivileged user if all of the following conditions are met:
* The target system runs systemd.
* The user as whom Ansible connects to the target system is not `root`.
* The Podman API socket is enabled for a non-root user (rootless Podman, see `podman_api_socket_user` below).

Becoming an unprivileged user only works if Ansible is configured appropriately.
Refer to [the Ansible documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_privilege_escalation.html#risks-of-becoming-an-unprivileged-user) for more information.
The easiest way may be to enable pipelining in `ansible.cfg` and not using `su` as the become method.

Role Variables
--------------

* `podman_api_socket_uri`  
  The URI for Podman' API socket.
  Optional.
  If not set, the target system determines the URI.
  Ignored if `podman_api_socket_user` is not set.
* `podman_api_socket_user`  
  The user to run Podman as when using the API socket.
  It may either be `root` to run Podman in rootfull mode or an unprivileged user to run Podman in rootless mode.
  If not set, the Podman system socket gets disabled.
  If set and not `root`, the user account is created during deployment.
  If set and not `root`, it may either be the name of the user or a dictionary with the following keys:
  * `name`  
    The name of the user.
    Mandatory.
  * `password`  
    The user's hashed password.
    Defaults to `*`, i.e. the user is locked.
  * `group`  
    The name of the user's primary group.
    This role creates the group during deployment.
    Optional.
  * `groups`  
    A list of supplementary groups the user will be made a member of.
    This role does not create these groups, so they must already exist on the target system.
    Optional.
  * `shell`  
    The user's login shell.
    Defaults to `/sbin/nologin`.
  * `home`  
    The user's home directory.
    Optional.
  * `comment`  
    The comment/description of the user account.
    Optional.
* `podman_users`  
  A list of system users that should be allowed to run Podman in rootless mode.
  Each user is assigned 65536 subordinate UID and GIDs.
  `podman_api_socket_user` is automatically added to the list if set to a value other than `root`.
  Note that changing the order of the list also changes the IDs assigned to each user.
  Optional.
* `podman_first_subuid`  
  The first sub UID (and GID) assigned to `podman_users`.
  There should be no need to change this unless regular users have very high UIDs.
  Defaults to `100000`.
* `podman_storage_conf`  
  A dictionary of options to set in `/etc/containers/storage.conf`.
  See `containers-storage.conf(5)` for valid options and their meaning.
  To add subtables, add them as a dictionary named like the subtable (without the parent table prefix).
  Optional.

Dependencies
------------

This role does not depend on any specific roles.

Example Configuration
---------------------

The following is a short example for some of the configuration options this role provides:

```yaml
podman_api_socket_uri: 'unix:///run/podman/podman.sock'
podman_api_socket_user:
  name: podman
  group: podman
  home: '/var/lib/podman'
podman_storage_conf:
  driver: btrfs
  options:
    btrfs:
      size: 128m
```

License
-------

MIT
