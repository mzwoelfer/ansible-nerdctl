# Molecule scenario: baremetal_rootless

> ! Always name the yaml file in here `yml`. WITHOUT AN "a" !

**Rootless nerdctl + containerd installation** on real or local (pre-existing) VMs.
Can not be reliably tested inside containers.
So this scenario runs against **bare-metal or VM hosts via SSH**.

## What this scenario does

- Runs the role in **rootless mode**
- Uses a **pre-downloaded nerdctl tarball** instead of GitHub
- Verifies:
  - nerdctl binary works
  - image pull works
  - container run works
  - image build with BuildKit works

## Why this exists

Rootless containerd relies on:
- user systemd
- user namespaces
- CNI networking
- SELinux / AppArmor exceptions

## Assumptions

- Target VMs exist
- Passwordless SSH access
- Inventory is provided via `inventory.yaml`
- Systemd user sessions are functional

## How to run

From the project's root:
```bash
molecule test -s baremetal_rootless
```
