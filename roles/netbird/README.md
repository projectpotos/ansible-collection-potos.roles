# netbird

Role to install the NetBird client on Potos Linux Clients.

Configures the NetBird package repository in `/etc/yum.repos.d/netbird.repo`,
installs the packages listed in `netbird_packages` and enables the `netbird`
service.


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

# Packages to install, the desktop client needs gtk4, webkitgtk6.0 and xdg-utils
netbird_packages:
  - netbird
  - netbird-ui
  - gtk4
  - webkitgtk6.0
  - xdg-utils
```

### Example

```yaml
netbird_repo_baseurl: https://mirror.company.example/netbird/yum/
netbird_repo_gpgkey: https://mirror.company.example/netbird/yum/repodata/repomd.xml.key
netbird_packages:
  - netbird
```

| variable                   | Possible values | Description                                                              |
|----------------------------|-----------------|--------------------------------------------------------------------------|
| netbird_repo_baseurl       | URL             | Base URL of the NetBird package repository                               |
| netbird_repo_gpgkey        | URL             | GPG key the repository is signed with                                    |
| netbird_repo_gpgcheck      | true, false     | Verify package signatures                                                |
| netbird_repo_repo_gpgcheck | true, false     | Verify repository metadata signatures                                    |
| netbird_packages           | list of str     | Packages to install, e.g. `netbird-ui-gtk3` or `network-manager-netbird` |

Another option is to use `ansible-doc` to read the argument specification:

```sh
ansible-doc --type role projectpotos.roles.netbird
```

## Requirements

The potos host needs to reach `netbird_repo_baseurl` to download the packages.
