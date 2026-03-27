# Molecule scenario: baremetal_rootful

Tests a **rootful nerdctl + containerd installation** on real (pre-existing) VMs.

## What this scenario does

- Runs the role in **rootful mode**
- Enables and starts system services:
  - containerd
  - buildkit
- Verifies basic container lifecycle using nerdctl

## Why this exists

Rootful installs behave differently from rootless:
- system services instead of user services
- no user namespace setup
- different SELinux/AppArmor behavior

This scenario ensures the role works correctly in classic server setups.

## How to run

From the roles root:
```bash
molecule test -s baremetal_rootful
```
