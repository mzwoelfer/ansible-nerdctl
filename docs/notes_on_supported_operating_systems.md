## ❗Important Notes on supported operating systems
### ❓ Why is Rocky10 not supported?
Rocky Linux 10 removed legacy netfilter kernel modules (e.g., `xt_comment`) required by CNI bridge networking. 
Rootless nerdctl + containerd can not create network bridges, so containers fail to start. 
Workarounds are: 
- custom kernels or 
- host networking

so Rocky10 is intentionally unsupported.
OUt of scope for this Ansible role.

### ❓ Why is Fedora not supported?
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

### 🔐 SELinux configuration for rootless containers (Fedora/RHEL)

On SELinux systems we enable the `selinuxuser_execmod` boolean on Fedora and RHEL systems. 
This allows rootless container runtimes to execute modules and mount filesystems without hitting SELinux restrictions.  
The role runs:

```bash
setsebool -P selinuxuser_execmod 1
```
only when `nerdctl_rootless` is true and SELinux is active.

### 🛡️ AppArmor configuration for rootless containers (Ubuntu 24.04+)

AppArmor blocks `rootlesskit` from creating user namespaces on Ubuntu 24.04 and later.
Because we install `rootlesskit` in `/usr/local/bin` and not in a package-managed system path like `/usr/bin`. 
The role detects the installed `rootlesskit` binary and installs a minimal AppArmor profile to allow it to run unconfined:

- Creates `/etc/apparmor.d/usr.local.bin.rootlesskit` with `flags=(unconfined)` for the detected binary path.
- Reloads the AppArmor service to apply the profile.

