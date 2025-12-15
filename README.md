# Ansible Role: nerdctl

Install [nerdctl](https://github.com/containerd/nerdctl) - a Docker-compatible CLI for [containerd](https://github.com/containerd/containerd).

## ⭐ Features

- Installs rootless nerdctl + containerd from GitHub releases
- Installs latest verison by default
- Supports major Long term support distributions:
    - Debian 11/12/13
    - Ubuntu 2204/2404
    - RHEL 9 (Rocky Linux for testing)
- Can use pre downloaded tar.gz nerdctl-archive
- Requires Internet Access to download nerdctl and install packages

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

## 🔧 Variables

| Variable                    | Default                                          | Description                                                                       |
| --------------------------- | ------------------------------------------------ | --------------------------------------------------------------------------------- |
| `nerdctl_version`           | `"latest"`                                       | Nerdctl version to install (e.g. `"2.2.0"`.           |
| `nerdctl_rootless`          | `true`                                           | Install nerdctl + containerd in rootless mode (recommended).                      |
| `nerdctl_user`              | `{{ ansible_user_id \| default(ansible_user) }}` | User that owns the rootless installation and user systemd units.                  |
| `nerdctl_home`              | `{{ ansible_env.HOME }}`                         | Home directory of `nerdctl_user`.                                                 |
| `nerdctl_install_prefix`    | `/usr/local`                                     | Location where nerdctl, containerd, rootlesskit, etc. will be installed. ⚠️ On RHEL/Rocky/Alma, `/usr/local/bin` is NOT in sudo’s PATH by default. Use full paths or adjust `secure_path`.                    |
| `nerdctl_bin`               | `{{ nerdctl_install_prefix }}/bin/nerdctl`       | Full path to the nerdctl binary.                                                  |
| `nerdctl_archive_path`      | `""`                                             | OPTIONAL: local path on the server to a pre-downloaded nerdctl archive (skips GitHub download).  |
| `nerdctl_rootless_packages` | distro-specific                                  | OS-specific packages required for rootless containers (uidmap, dbus, fuse, etc.). |

### 📦 Install prefix recommendation by distribution

By default the role installs non package managed software in `/usr/local`. Therefore following the recommendation in footnote 28 by the [Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch04s09.html)(FHS).

However, behavior differs by distribution:

- **Debian / Ubuntu**
  - `/usr/local/bin` is in the default user and `sudo` $PATH
  - `sudo nerdctl` works out of the box

- **RHEL / Rocky / Alma**
  - `/usr/local/bin` is **not** in sudo’s `secure_path`
  - `sudo nerdctl` will fail unless you:
    - use the full path (`/usr/local/bin/nerdctl`), or
    - adjust `secure_path`, or
    - install into `/usr/bin`

If you want `sudo nerdctl` to work without $PATH changes on RHEL-based systems:
1. Use the full path:
```BASH
sudo /usr/local/bin/nerdctl ps
```
2. Extend the sudo $PATH:
```BASH
sudo visudo
# extend:
Defaults secure_path="/usr/local/bin:"
```

## 🚦 Run tests on bare-metal VMs
> [!CAUTION] 
> WHY LOCAL TESTING?
> I could not find a way to test user systemd stuff in the CI using containers. If you find a way, let me know.

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

## 📚 Sources
- [Nerdctl GitHub Repository](https://github.com/containerd/nerdctl)
- [Common steps for rootless containers](https://rootlesscontaine.rs/getting-jstarted/common/)
- [Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)

## 📜 License

MIT

