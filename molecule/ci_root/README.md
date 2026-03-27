# Molecule scenario: rootful
> ! Always name the yaml file in here `yml`. WITHOUT AN "a" !

Tests a **rootful nerdctl installation inside a Molecule-managed environment**.
**Fast sanity checks** and CI-style validation.

## What this scenario does

- Runs the role in rootful mode
- Focuses on:
  - binary installation
  - service enablement
  - basic nerdctl functionality

## What this does NOT test

- Rootless containers
- User systemd
- CNI bridge networking edge cases

## How to run

From the projects root:
```bash
molecule test -s rootful
```
