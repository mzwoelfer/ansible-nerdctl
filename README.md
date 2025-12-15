# Ansible Role: nerdctl

Install [nerdctl](https://github.com/containerd/nerdctl) - a Docker-compatible CLI for [containerd](https://github.com/containerd/containerd).

## ⭐ Features

- Installs latest rootless nerdctl + containerd from GitHub releases
- Supports major Long term support distributions:
    - Debian 11/12/13
    - Ubuntu 2204/2404
    - RHEL 9 (Rocky Linux for testing)
    - Fedora 41/42/43
- Installs latest verison by default

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

| Variable | Default | Description |
|----------|---------|-------------|
| `nerdctl_version` | `"latest"` | Version of nerdctl to install (e.g., `"2.2.0"`) |
| `nerdctl_rootless` | `true` | Install in rootless mode (recommended) |
| `nerdctl_user` | `{{ ansible_user_id }}` | User to install nerdctl for |
| `nerdctl_home` | `{{ ansible_env.HOME }}` | Home directory for the user |


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

## Important Notes on supported operating systems
### Why is Rocky10 not supported?
Rocky Linux 10 removed legacy netfilter kernel modules (e.g., `xt_comment`) required by CNI bridge networking. 
Rootless nerdctl + containerd can not create network bridges, so containers fail to start. 
Workarounds are: 
- custom kernels or 
- host networking

so Rocky10 is intentionally unsupported.
OUt of scope for this Ansible role.

### SELinux configuration for rootless containers (Fedora/RHEL)

On SELinux systems we enable the `selinuxuser_execmod` boolean on Fedora and RHEL systems. 
This allows rootless container runtimes to execute modules and mount filesystems without hitting SELinux restrictions.  
The role runs:

```bash
setsebool -P selinuxuser_execmod 1
```
only when `nerdctl_rootless` is true and SELinux is active.

### AppArmor configuration for rootless containers (Ubuntu 24.04+)

Apparmor blocks `rootlesskit` on Ubuntu 24.04 and later.
Because we install `rootlesskit` in `/usr/local/bin`, not a package-managed system path like `/usr/bin`. 
AppArmor blocks rootlesskit from creating user namespaces or performing certain operations.
The role detects the installed `rootlesskit` binary and installs a minimal AppArmor profile to allow it to run unconfined:

- Creates `/etc/apparmor.d/usr.local.bin.rootlesskit` with `flags=(unconfined)` for the detected binary path.
- Reloads the AppArmor service to apply the profile.

This ensures rootless containers using nerdctl + containerd work without AppArmor blocking their user namespaces or filesystem mounts.


## License

MIT

