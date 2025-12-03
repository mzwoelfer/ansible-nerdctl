# Ansible Role: nerdctl

Installs [nerdctl](https://github.com/containerd/nerdctl) - a Docker-compatible CLI for containerd.

## ⭐ Features

- Installs latest rootless nerdctl + containerd from GitHub releases
- Supports Debian 11/12/13
- Handles Debian 11 rootless quirks automatically
- Optional version pinning

## 🔧 Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `nerdctl_version` | `"latest"` | Version of nerdctl to install (e.g., `"2.2.0"`) |
| `nerdctl_rootless` | `true` | Install in rootless mode (recommended) |
| `nerdctl_user` | `{{ ansible_user_id }}` | User to install nerdctl for |
| `nerdctl_home` | `{{ ansible_env.HOME }}` | Home directory for the user |

## 🚀 Usage

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
        nerdctl_version: "2.2.0"
```

## 🚦 Run tests on bare-metal VMs
Runs Molecule against existing VMs. 
Does not create or manage them.

#### 🔧 Prerequisites
- VMs running 
- Reach VMs via SSH without a password
- `molecule` + `molecule-plugins` installed (“default” driver).

#### Install Molecule + plugins:
```SHELL
pip install molecule molecule-plugins
```
#### 🧩 Inventory Setup
Copy the example inventory and fill in your VM details:
```SHELL
cp example.inventory.yaml inventory.yaml
```

Edit the file and add:
- VM hostnames
- Their IPs
- SSH user
- SSH private key path
(example from the Nerdctl role:)
```YAML
all:
  vars:
    ansible_user: user
    ansible_ssh_private_key_file: ~/.ssh/github
  hosts:
    baremetal-vm-1:
      ansible_host: 192.168.22.151
    baremetal-vm-2:
      ansible_host: 192.168.22.152
```

Molecule scenarios link this file automatically.

#### ▶️ Running the Tests
From the project root, run your scenario:

Rootful install test:
```SHELL
molecule test -s baremetal_rootful
```

Rootless install test:
```SHELL
molecule test -s baremetal_rootless
```


Run verification individually:
```
molecule verify -s baremetal_rootful
# or:
molecule verify -s baremetal_rootless
```

#### 🧹 Resetting a Scenario
If molecule complains:
```SHELL
molecule reset -s baremetal_rootful
```

This clears Molecule’s local working directory — it does not touch your VMs.

## License

MIT

