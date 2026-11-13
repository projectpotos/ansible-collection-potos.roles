# netbird

Role to install the NetBird client on Potos Linux Clients and connect it to a
NetBird server through NetworkManager.

The role

- configures the NetBird package repository in `/etc/yum.repos.d/netbird.repo`,
  installs the packages listed in `netbird_packages` and enables the `netbird`
  service,
- creates the NetworkManager VPN connection
  `/etc/NetworkManager/system-connections/netbird.nmconnection` for the user of
  the machine,
- creates the NetBird profile `nm-<uuid>` behind that connection, hands it to
  the user and applies `netbird_profile_settings` to it.

The user connects from the network menu of the desktop or with
`nmcli connection up NetBird --ask` and logs in through SSO in the browser.

## Example Playbook

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true
  tasks:
    - name: Run the netbird role
      ansible.builtin.include_role:
        name: projectpotos.roles.netbird
```

## Role Variables

The default variables are defined in [defaults/main.yml](./defaults/main.yml):

```yaml
# Package repository, override to use a mirror
netbird_repo_baseurl: https://pkgs.netbird.io/yum/
netbird_repo_gpgkey: https://pkgs.netbird.io/yum/repodata/repomd.xml.key
netbird_repo_gpgcheck: true
netbird_repo_repo_gpgcheck: true

# Packages to install
netbird_packages:
  - netbird
  - netbird-ui
  - gtk4
  - webkitgtk6.0
  - xdg-utils
  - network-manager-netbird

# NetBird server, users log in there through SSO
netbird_management_url: https://api.netbird.io
netbird_admin_url: https://app.netbird.io

# NetworkManager VPN connection
netbird_network_manager: true
netbird_nm_connection_name: NetBird
netbird_nm_connection_uuid: 5c33fb92-d200-48ba-8eba-5b6c0a9302c8
netbird_nm_username: ""
netbird_nm_uid: ""

# Settings of the NetBird profile
netbird_profile_settings: {}
```

### Example

A self-hosted server with a stricter profile, set in the specs repository:

```yaml
netbird_management_url: https://netbird.company.example
netbird_admin_url: https://netbird.company.example
netbird_profile_settings:
  blockLanAccess: true
  blockInbound: true
  disableServerRoutes: true
```

| variable                   | Possible values | Description                                                                |
|----------------------------|-----------------|----------------------------------------------------------------------------|
| netbird_repo_baseurl       | URL             | Base URL of the NetBird package repository                                 |
| netbird_repo_gpgkey        | URL             | GPG key the repository is signed with                                      |
| netbird_repo_gpgcheck      | true, false     | Verify package signatures                                                  |
| netbird_repo_repo_gpgcheck | true, false     | Verify repository metadata signatures                                      |
| netbird_packages           | list of str     | Packages to install, add `netbird-ui` for the NetBird desktop client       |
| netbird_management_url     | URL             | NetBird management service                                                 |
| netbird_admin_url          | URL             | NetBird dashboard, only used for the NetworkManager connection             |
| netbird_network_manager    | true, false     | Create the NetworkManager connection and its NetBird profile               |
| netbird_nm_connection_name | str             | Name of the NetworkManager connection                                      |
| netbird_nm_connection_uuid | UUID            | UUID of the NetworkManager connection, the profile is named `nm-<uuid>`    |
| netbird_nm_username        | str             | User of the connection and owner of the profile, by name                   |
| netbird_nm_uid             | UID             | User of the connection and owner of the profile, by UID                    |
| netbird_profile_settings   | dict            | Profile settings enforced on every run                                     |

Another option is to use `ansible-doc` to read the argument specification:

```sh
ansible-doc --type role projectpotos.roles.netbird
```

### User of the connection

Potos creates the user during the first boot, so its name differs per machine.
The role resolves the user in this order:

1. `netbird_nm_username`, the name of the user
2. `netbird_nm_uid`, the UID of the user, for example `1000` for the first
   user of every machine
3. the regular user (UID 1000 to 59999) with the lowest UID

Only this user sees the connection in NetworkManager and the profile in
`netbird profile list`.

On a machine without a matching user the connection is open to all users and
the profile stays with root. The next run hands both to the user.

### Profile settings

`netbird_profile_settings` takes the JSON fields of the `SetConfig` call of the
NetBird daemon, the common ones are:

| field               | Description                                                              |
|---------------------|--------------------------------------------------------------------------|
| blockLanAccess      | Block access to the local network when the client routes for other peers |
| blockInbound        | Block all connections from other peers to the client                     |
| disableServerRoutes | Never route traffic of other peers                                       |
| disableClientRoutes | Ignore the routes received from the management service                   |
| disableDns          | Leave the DNS configuration untouched                                    |
| serverSSHAllowed    | Run the NetBird SSH server                                               |
| rosenpassEnabled    | Post-quantum key exchange                                                |
| wireguardPort       | WireGuard listening port                                                 |

Listed settings are reset on every run, the owner of the profile can change
them in between. Settings that are not listed are left to the user. Fields the
daemon does not report back, like `disableFirewall`, are sent on every run
without being reported as changed. The daemon ignores unknown fields.

### Without NetworkManager

With `netbird_network_manager: false` no connection is created, and
`netbird_management_url` and `netbird_profile_settings` are applied to the
`default` NetBird profile. Replace `network-manager-netbird` with `netbird-ui`
in `netbird_packages` to let users log in with the desktop client.

## Requirements

- The potos host needs to reach `netbird_repo_baseurl` to download the packages.
- NetworkManager is running, unless `netbird_network_manager` is `false`.
- NetBird 0.83 or newer. The role enables the JSON API socket of the daemon
  (`/var/run/netbird-http.sock`) to manage the profile.
