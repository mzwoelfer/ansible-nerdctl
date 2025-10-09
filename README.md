# Ansible Role: nerdctl

Installs [nerdctl](https://github.com/containerd/nerdctl) - a Docker-compatible CLI for containerd.

## Features

- Installs latest rootless nerdctl + containerd from GitHub releases
- Supports Debian 11/12/13
- Handles Debian 11 rootless quirks automatically
- Optional version pinning

## Usage

```yaml
- hosts: all
  roles:
    - role: ansible-nerdctl
```

With specific version:
```yaml
- hosts: all
  roles:
    - role: ansible-nerdctl
      vars:
        nerdctl_version: "1.7.4"
```

## License

MIT

