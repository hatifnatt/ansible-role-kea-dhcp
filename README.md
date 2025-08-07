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

## Official ISC repositories

- <https://kb.isc.org/docs/isc-kea-packages>
- <https://cloudsmith.io/~isc/repos/kea-2-6/setup/>

It's possible to setup (and remove) official ISC repository with Kea packages hosted by Cloudsmith, usually it's recommended to do so because this repository contains the latest versions of packages, unlike the system repository where packages usually old.

You can enable or disable ISC repository by setting variable

```
# add repository to the system
kea_dhcp_isc_repo_ensure: present

# remove repository from the system
kea_dhcp_isc_repo_ensure: absent
```

Set repository version, only version 2.6 currently supported

```
kea_dhcp_repo_version: 2.6
```

Packages in official repository does have `isc-` prefix, service names also have `isc-` prefix, so in the Debian repository package and service names are

```
kea
kea-ctrl-agent
kea-dhcp-ddns-server
kea-dhcp4-server
kea-dhcp6-server
```

and in ISC repository they are

```
isc-kea
isc-kea-ctrl-agent
isc-kea-dhcp-ddns-server
isc-kea-dhcp4-server
isc-kea-dhcp6-server
```

I's crucial to use correct names for services when you are setting up `kea_dhcp_services` variable.

To be able to use packages from ISC repository or from system repository addition file with variables have been added check out [vars/Debian-isc-repo.yml](vars/Debian-isc-repo.yml) for details.

### Switching (upgrading) from Debian system packages to ISC packages

Case - Debian 12, upgrading from Debian packages version 2.2.0 to ISC packages 2.6.4

Problems and solutions

- **Problem:** `isc-kea-dhcp4-server.service` can't start due leftover AppArmor profile from `kea-dhcp4-server` package
  **Solution:** fully remove all `kea*` packages (you have Ansible to create config files) `apt purge kea*`, than unload AppArmor leftover profiles `aa-remove-unknown`
- **Problem:** `isc-kea-dhcp4-server.service` can't start with error

  ```
  ERROR DHCP4_PARSER_FAIL failed to create or run parser for configuration element subnet4: subnet configuration failed: missing parameter'id'
  ```

  **Solution:** In version 2.6 `id` parameter for each `subnet` is mandatory, add `id: N` where `N` is uniq number between 1 and 4294967295 to each of your `subnet` section.

  ```yaml
  ---
  kea_dhcp_dhcp4_config:
    Dhcp4:
      # ...
      subnet4:
        - id: 1 # add id
          subnet: "192.168.20.0/24"
  ```

  - <https://kea.readthedocs.io/en/kea-2.6.0/arm/dhcp4-srv.html#ipv4-subnet-identifier>
  - <https://discussion.fedoraproject.org/t/following-upgrade-kea-dhcp4-server-fails-to-start-due-to-configuration-error/119564>

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
