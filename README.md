# Ansible Role: Install Nerdctl

Install the latest rootless Nerdctl full (installs containerd) version on a Debian system from GitHub.

[Nerdctl](https://github.com/containerd/nerdctl): contaiNERD CTL - Docker-compatible CLI for containerd.

Tested on:
- [X] Debian 11/12/13

Under development:
- [ ] RHEL 8/9/10 support (Testing on Rocky Linux)
- [ ] rootfull install
- [ ] Own path for containerd tarball

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


## Testing
1. Install requirements
```SHELL
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst
```

2. Setup python environment: 
```SHELL
python3 -m venv Env
source Env/bin/activate

pip install molecule 
```

3. Create molecule scenario
```SHELL

```

## LICENSE
MIT

