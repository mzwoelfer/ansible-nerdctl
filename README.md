# Ansible Role: nerdctl

Installs [nerdctl](https://github.com/containerd/nerdctl) - a Docker-compatible CLI for containerd.

## Features

- Installs latest rootless nerdctl + containerd from GitHub releases
- Supports Debian 11/12/13
- Handles Debian 11 rootless quirks automatically
- Optional version pinning

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `nerdctl_version` | `"latest"` | Version of nerdctl to install (e.g., `"1.7.4"`) |
| `nerdctl_rootless` | `true` | Install in rootless mode (recommended) |
| `nerdctl_user` | `{{ ansible_user_id }}` | User to install nerdctl for |
| `nerdctl_home` | `{{ ansible_env.HOME }}` | Home directory for the user |

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

