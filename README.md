# Ansible Role: chromium

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-chromium)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-chromium)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-chromium)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-chromium/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-chromium/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-chromium/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-chromium/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for configuring system-wide Chromium browser policies.

## Purpose

Configure Chromium for all users through a system-wide JSON policy file.
Policy values support booleans, numbers, strings, lists and nested dictionaries.
The role replaces its file with exactly the supplied policies; applying the
same inputs again is idempotent. An empty dictionary writes an empty JSON
object, clearing policies previously supplied through this file.

## Scope

### Managed

- The mandatory policy directory and its ansible.json file

### Not Managed

- Browser installation, upgrades, user profiles and browser restarts
- Other policy files, Google Chrome and Flatpak policy deployment

## Requirements

- A Chromium installation that reads the configured policy directory is needed
  to consume the file; configuration can be deployed before installation.
- On Ubuntu, the Chromium snap needs its etc-chromium-browser-policies interface
  connected. The official snap normally connects it automatically.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
  - name: ansible.posix
    version: '>=2.0.0'
```

## Role Variables

### `chromium_policies`

Type: `dict`. Required: `false`.

Chromium policy names mapped to their JSON-compatible values.
The dictionary replaces the contents of ansible.json; an empty dictionary clears
its policies.
Other policy files are preserved.

Default:

```yaml
chromium_policies: {}
```

### `chromium_policy_dir`

Type: `path`. Required: `false`.

Absolute directory in which ansible.json is managed; null selects the platform
default.
Uses /etc/chromium-browser/policies/managed on Ubuntu and
/etc/chromium/policies/managed on other supported platforms.

Default:

```yaml
chromium_policy_dir: null
```

## Managed Files

- `/etc/chromium/policies/managed/ansible.json` Default on AlmaLinux, Debian,
  Fedora, openSUSE Leap and openSUSE Tumbleweed
- `/etc/chromium-browser/policies/managed/ansible.json` Default on Ubuntu,
  including the official Chromium snap

## Check Mode

Check mode reports pending directory and JSON changes without writing them.

## Service Behavior

No browser process is started or restarted. Inspect chrome://policy and reload
policies there; policies without dynamic refresh require a browser restart.

## Security Notes

- The policy directory is owned by root with mode 0755; ansible.json is owned by
  root with mode 0644.
- Policy values are readable by all local users and must not contain secrets.

## Operational Notes

- Set chromium_policy_dir to the absolute managed-policy directory of a
  differently packaged Chromium build. Changing it leaves the previous file in
  place; remove that file explicitly when migrating paths.
- Avoid defining the same policy in other JSON files: Chromium does not
  guarantee which conflicting value wins.
- JSON serialization preserves value types and escapes strings. Policy names,
  value constraints and browser-version support are checked by Chromium, not by
  a standalone policy validator; inspect chrome://policy for errors.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Configure mandatory browser policies

Enforce a homepage, startup URLs and password-manager settings for all users.

```yaml
---
- name: Configure Chromium
  hosts: workstations
  gather_facts: true
  roles:
    - role: jomrr.chromium
      chromium_policies:
        HomepageLocation: https://intranet.example.org/
        HomepageIsNewTabPage: false
        RestoreOnStartup: 4
        RestoreOnStartupURLs:
          - https://intranet.example.org/
        PasswordManagerEnabled: false
```

### Load policies from a JSON file

Read a controller-side JSON object as the complete managed policy dictionary.

```yaml
---
- name: Configure Chromium from JSON
  hosts: workstations
  gather_facts: true
  roles:
    - role: jomrr.chromium
      chromium_policies: >-
        {{ lookup('ansible.builtin.file', 'files/policies.json')
           | from_json }}
```

## References

- [Chromium Linux policy deployment](https://www.chromium.org/administrators/linux-quick-start/)
- [Chromium policy list](https://chromeenterprise.google/policies/)
- [Ubuntu Chromium snap policy access](https://forum.snapcraft.io/t/auto-connecting-the-system-files-interface-for-the-chromium-snap/20245)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2026 Jonas Mauer.
