Ansible Role Hosts
=========

[![Build Status](https://travis-ci.org/Turgon37/ansible-ipcheck.svg?branch=master)](https://travis-ci.org/Turgon37/ansible-ipcheck)

:warning: This role is under development, some important (and possibly breaking) changes may happend. Don't use it in production level environments but you can eventually base your own role on this one :hammer:

:grey_exclamation: Before using this role, please know that all my Ansible roles are fully written and accustomed to my IT infrastructure. So, even if they are as generic as possible they will not necessarily fill your needs, I advice you to carrefully analyse what they do and evaluate their capability to be installed securely on your servers.

**This roles allow configuration of hosts file (commonly located at /etc/hosts).**

## Features

Currently this role provide the following features :

  * ipcheck tool installation (see https://github.com/Turgon37/IpCheck.git)
  * ipcheck configuration
  * externals ressources fetching

## Requirements

### OS Family

This role is available for

  * Debian/Raspbian 8/9

### Dependencies


## Role Variables

The variables that can be passed to this role and a brief description about them are as follows:

| Name                           | Types/Values       | Description                                                |
| -------------------------------|--------------------|----------------------------------------------------------- |
| ipcheck__facts                 | Boolean            | Install the local fact script                              |
| ipcheck__git_update            | Boolean            | Allow automatic git pull on each ansible run               |
| ipcheck__cron_enabled          | Boolean            | Enable the script cron job                                 |
| ipcheck__urls                  | List of urls       | List of url from which to retrieve the external ip address |
| ipcheck__extensions            | Dict               | Configuration of all extensions (for ipcheck advanced mode)|
| ipcheck__extensions_ressources | List of ressources | List of externals ressources to fetch                      |


## Example Playbook

To use this role create or update your playbook according the following example :


### Full example with the dyndns updater, the dig checker and the email extensions


```
    - hosts: servers
      roles:
         - ipcheck
      vars:
        ipcheck__extensions_ressources_url:
          - url: https://raw.githubusercontent.com/Turgon37/DynUpdate/master/dynupdate.py
            file: dynupdate.py

        ipcheck__extensions:
          digchecker:
            server: nsserver.example.com
            hostname: mypersonal_dynhost.example.com
          mail:
            sender: noreply@example.net
            recipient: supervisor@example.net
            body: 'Hi,\n\n{message}\n\nRegards,\nIpCheck supervisor for Nancy'
            server: mysmtp
            port: 25
            info_mail: True
          command:
            exec: dynupdate.py
            args: '-a {ip} --no-output -s {{ dynupdate_server }} -h {{ dynupdate_dynhost }} -u {{ dynupdate_user }} -p {{ dynupdate_password }}'
            event: "{{ ['E_START', 'E_UPDATE']|join(',') }}"
```

## License

MIT