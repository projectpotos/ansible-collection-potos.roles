# dnf

Role to manage dnf repositories of Potos Linux Clients.

Makes the repositories listed in `dnf_repositories` the only package sources
of the client. They are written to `/etc/yum.repos.d/potos.repo`, and every
other repository is disabled through `/etc/dnf/repos.override.d`.

The role fails without changing anything if no repository would remain
enabled.

## Example Playbook

As this role is tested via Molecule one can use [that
playbook](../../extensions/molecule/dnf/converge.yml) as a starting point:

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true
  tasks:
    - name: Run the dnf role
      ansible.builtin.include_role:
        name: projectpotos.roles.dnf
```

## Role Variables

The default variables are defined in [defaults/main.yml](./defaults/main.yml):

```yaml
# Repositories to manage, every other repository on the client is disabled
dnf_repositories: []

# IDs of repositories not managed by this role that stay enabled
dnf_enabled_unmanaged: []

# Packages excluded from all dnf operations (globs allowed)
dnf_excludes: []
```

### Example

```yaml
dnf_repositories:
  - id: potos-fedora
    name: Fedora $releasever
    baseurl: https://pulp.example.com/pulp/content/fedora/$releasever/
    gpgkey: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
  - id: potos-netbird
    name: NetBird
    baseurl: https://pulp.example.com/pulp/content/netbird/
    gpgkey: https://pulp.example.com/keys/netbird.asc
    options:
      metadata_expire: 1h

dnf_enabled_unmanaged:
  - fedora-cisco-openh264

dnf_excludes:
  - kernel*
```

| variable | Default  | Description                                                           |
|----------|----------|-----------------------------------------------------------------------|
| id       |          | Repository ID                                                         |
| name     | the ID   | Human readable name                                                   |
| baseurl  |          | URL of the repository                                                 |
| enabled  | `true`   | Whether the repository is enabled                                     |
| gpgcheck | `true`   | Whether package signatures are verified                               |
| gpgkey   |          | URL or path of the key the packages are signed with                   |
| options  | `{}`     | Any other dnf repository option; `countme` is `0` unless given here   |

Another option is to use `ansible-doc` to read the argument specification:

```sh
ansible-doc --type role projectpotos.roles.dnf
```
