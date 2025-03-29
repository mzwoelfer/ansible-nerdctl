# Ansible Role: Install Nerdctl

Install the latest (rootless) Nerdctl full (installs containerd) version on a Debian system from GitHub.

[Nerdctl](https://github.com/containerd/nerdctl): contaiNERD CTL - Docker-compatible CLi for containerd.

Tested on:
- Debian 12

## Requirements
None.

## Dependencies
None.

## Role Variable
None.

## Example Playbook
Installs latest rootless nerdctl + containerd version
```YAML
- hosts: all
  roles:
    - role: ansible-nerdctl
```

To install a specific version:
```YAML
- hosts: all
  roles:
    - role: ansible-nerdctl
      vars:
        nerdctl_version: "1.7.4"
```

## LICENSE
MIT

