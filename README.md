# kea-dhcp

Ansible role to setup Kea DHCP.

This role is written and tested only on Debian 12 with `kea 2.2.0-6` package from Debian repository. This role **does not** currently install third-party repositories i.e. [Cloudsmith](https://cloudsmith.io/~isc/repos/) official repository which described in official ISC documentation - [Using Official ISC Packages for Kea](https://kb.isc.org/docs/isc-kea-packages).

## Requirements

None

## Role Variables

With default variables only packages will be installed. No configuration files will be modified, no services state will be altered.

Check out [vars_example.yml](vars_example.yml) for examples.

Brief description of some variables

```yaml
# Enable or disable management of individual Kea services and its configuration files
kea_dhcp_services:
  - name: kea-ctrl-agent
    # Disable kea-ctrl-agent
    manage: true
    state: stopped
    enabled: false
    template: etc/kea/kea-ctrl-agent.conf.j2
    dest: /etc/kea/kea-ctrl-agent.conf
    # Do not manage kea-ctrl-agent config file
    manage_config: false
  - name: kea-dhcp-ddns-server
    # Enable kea-dhcp-ddns-server servise
    manage: true
    state: started
    enabled: true
    template: etc/kea/kea-dhcp-ddns.conf.j2
    dest: /etc/kea/kea-dhcp-ddns.conf
    # Enable kea-dhcp-ddns-server config management
    manage_config: true
  - name: kea-dhcp4-server
    # Enable kea-dhcp-ddns-server servise
    manage: true
    state: started
    enabled: true
    template: etc/kea/kea-dhcp4.conf.j2
    dest: /etc/kea/kea-dhcp4.conf
    # Enable kea-dhcp4-server config management
    manage_config: true
  - name: kea-dhcp6-server
    # Disable kea-dhcp6-server
    manage: true
    state: stopped
    enabled: false
    template: etc/kea/kea-dhcp6.conf.j2
    dest: /etc/kea/kea-dhcp6.conf
    # Do not manage kea-dhcp6-server config file
    manage_config: false

# Enable management of all configuration files
kea_dhcp_config_manage: true

# Create configuration files backup with .example extension
kea_dhcp_config_backup: true
```

## Dependencies

None

## Example Playbook

```yaml
---
- name: Setup ISC Kea DHCP server
  hosts: pve
  roles:
    - role: hatifnatt.kea-dhcp
      become: true
```

## License

MIT

## Credits

Inspired by [mrlesmithjr ansible-kea-dhcp](https://github.com/mrlesmithjr/ansible-kea-dhcp) role.
