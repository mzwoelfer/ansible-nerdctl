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
| `nerdctl_install_prefix`    | `/usr/local`                                     | Location where nerdctl, containerd, rootlesskit, etc. will be installed                    |
| `nerdctl_bin`               | `{{ nerdctl_install_prefix }}/bin/nerdctl`       | Full path to the nerdctl binary.                                                  |
| `nerdctl_archive_path`      | `""`                                             | OPTIONAL: local path on the server to a pre-downloaded nerdctl archive (skips GitHub download).  |
| `nerdctl_rootless_packages` | distro-specific                                  | OS-specific packages required for rootless containers (uidmap, dbus, fuse, etc.). |


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

## ❗Important Notes on supported operating systems
### Why is Rocky10 not supported?
Rocky Linux 10 removed legacy netfilter kernel modules (e.g., `xt_comment`) required by CNI bridge networking. 
Rootless nerdctl + containerd can not create network bridges, so containers fail to start. 
Workarounds are: 
- custom kernels or 
- host networking

so Rocky10 is intentionally unsupported.
OUt of scope for this Ansible role.

### Why is Fedora not supported?
##### Fedora41 - python3-libdnf5 issues
Fedora41 nears it's end of life.
Rootless networking with containerd/BuildKit requires Python bindings for DNF5 (python3-libdnf5). 
On Fedora41, these are not installed by default, so Ansible cannot manage packages reliably during provisioning. 
Supporting Fedora41 would require bootstrapping these dependencies manually, which adds extra complexity and fragility to the molecule tests.

Therefore, I skip supporting it.

##### Fedora42 - rootless networking
Fedora42 hardens unprivileged user namespaces.
That prevents rootless containers from accessing `CAP_NET_ADMIN` for `iptables`. 
This breaks rootless networking and BuildKit’s default CNI setup. 
Fixing it requires kernel tweaks or preloading modules, which is outside the scope of automated testing.
So I skip supporting rootless scenarios on Fedora42.

### SELinux configuration for rootless containers (Fedora/RHEL)

On SELinux systems we enable the `selinuxuser_execmod` boolean on Fedora and RHEL systems. 
This allows rootless container runtimes to execute modules and mount filesystems without hitting SELinux restrictions.  
The role runs:

```bash
setsebool -P selinuxuser_execmod 1
```
only when `nerdctl_rootless` is true and SELinux is active.

### AppArmor configuration for rootless containers (Ubuntu 24.04+)

AppArmor blocks `rootlesskit` from creating user namespaces on Ubuntu 24.04 and later.
Because we install `rootlesskit` in `/usr/local/bin` and not in a package-managed system path like `/usr/bin`. 
The role detects the installed `rootlesskit` binary and installs a minimal AppArmor profile to allow it to run unconfined:

- Creates `/etc/apparmor.d/usr.local.bin.rootlesskit` with `flags=(unconfined)` for the detected binary path.
- Reloads the AppArmor service to apply the profile.


## 📚 Sources
- [Nerdctl GitHub Repository](https://github.com/containerd/nerdctl)
- [Common steps for rootless containers](https://rootlesscontaine.rs/getting-jstarted/common/)

## License

MIT

