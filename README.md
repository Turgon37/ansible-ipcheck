Ansible Role Ipcheck
=========

[![Build Status](https://travis-ci.com/Turgon37/ansible-ipcheck.svg?branch=master)](https://travis-ci.com/Turgon37/ansible-ipcheck)
[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![Ansible Role](https://img.shields.io/badge/ansible%20role-Turgon37.ipcheck-blue.svg)](https://galaxy.ansible.com/Turgon37/ipcheck/)


## Description

:grey_exclamation: Before using this role, please know that all my Ansible roles are fully written and accustomed to my IT infrastructure. So, even if they are as generic as possible they will not necessarily fill your needs, I advice you to carrefully analyse what they do and evaluate their capability to be installed securely on your servers.

This roles install the Ipcheck tool. It monitor your public ip address and can run trigger on ip changes (like DynDNS update, mailing....)

## Requirements

Require Ansible >= 2.4


## OS Family

This role is available for Debian

## Features

Currently this role provide the following features :

  * ipcheck tool installation (see https://github.com/Turgon37/IpCheck.git)
  * ipcheck configuration
  * externals resources fetch (scripts or utilities)
  * [local facts](#facts)

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below. To see default values please refer to this file.


| Name                           | Types/Values      | Description                                                                       |
| -------------------------------|-------------------|---------------------------------------------------------------------------------- |
| ipcheck__facts                 | Boolean           | Install the local fact script                                                     |
| ipcheck__repository_version    | String            | Ipcheck's version to install                                                      |
| ipcheck__git_update            | Boolean           | Allow automatic git pull on each ansible run                                      |
| ipcheck__runas_user            | String            | The user that run ipcheck                                                         |
| ipcheck__jobs                  | Dict of jobs      | Configure ipcheck jobs                                                            |
| ipcheck__extensions_ressources | List of ressources| List of externals resources to fetch and install in ipcheck's resources directory |

### Jobs

Each job configure ipcheck to track an ip address from one or several urls and run trigger on changes.

You must use the ipcheck__jobs to configure jobs using a key per job and the available options as value :

| Name           | Types/Values       | Description                                                  |
| ---------------|--------------------|------------------------------------------------------------- |
| urls           | List of urls       | Urls availables to ipcheck for retrieving current external ip|
| script_options | String             | Raw cli options passed to ipcheck runtime                    |
| cron_enabled   | Boolean            | Configure cron but choose to enable it or not                |
| extensions     | Dict of extensions | Dict of configured extensions for this job                   |

Please refer to ipcheck repository to see availables extensions and their usages and options.

## Facts

By default the local fact are installed and expose the following variables :


```
ansible_local.ipcheck:
    version_full: "1.0.0"
    version_major: "1"
```


## Example Playbook

To use this role create or update your playbook according the following example :


### Full example with the dyndns updater, the dig checker and the email extensions


```
- hosts: all
  roles:
    - role: turgon37.ipcheck
      vars:
        ipcheck__extensions_ressources:
          - url: https://raw.githubusercontent.com/Turgon37/DynDNSUpdate/master/dyndnsupdate.py
            file: dynupdate.py
            checksum: sha256:8668b9923cef65928f430c0775e4af025059d7fe4f1eb1aa8df6830cf18e3adc

        ipcheck__jobs:
          ovh:
            urls: &_urls
              - https://api.ipify.org/
              - https://bot.whatismyipaddress.com/
            extensions:
              digchecker:
                server: nsserver.example.com
                hostname: mypersonal_dynhost.example.com
              mail: &_mail
                sender: noreply@example.net
                recipient: supervisor@example.net
                body: 'Hi,\n\n{message}\n\nRegards,\nIpCheck v{ipcheck_version}'
                server: mysmtp
                port: 25
                info_mail: true
              command:
                exec: dynupdate.py
                args: >-
                  --dyn-address {ip}
                  --dyn-hostname mypersonal_dynhost.example.com
                  --username {{ vault__dynupdate_user }}
                  --password {{ vault__dynupdate_password }}
                  --dyn-server https://www.ovh.com
                  --errors-to-stderr
                event: E_START,E_UPDATE
          spdyn:
            urls: *_urls
            extensions:
              digchecker:
                hostname: mypersonal_dynhost2.example.com
              mail: *_mail
              command:
                exec: dynupdate.py
                args: >-
                  --dyn-address {ip}
                  --dyn-hostname mypersonal_dynhost2.example.com
                  --username {{ vault__dynupdate_user }}
                  --password {{ vault__dynupdate_password }}
                  --dyn-server https://update.spdyn.de
                  --errors-to-stderr
                event: E_START,E_UPDATE
```
