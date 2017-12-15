IBM Spectrum Scale (GPFS) Ansible Role
======================================

Highly-configurable Ansible role to install IBM Spectrum Scale (GPFS)

This project is in very early development. Not ready for production, yet!

Features
--------

- Install IBM Spectrum Scale (GPFS) packages on Linux nodes
- Optionally, verify package integrity by comparing checksums
- Perform (offline) upgrade if daemon is stopped
- Configure SSH pubkey authentication
- Compile Linux kernel extension

The following installation methods are available:
- Install from (existing) YUM repository
- Install from remote installation package (accessible on Ansible managed node)
- Install from local installation package (accessible on Ansible control machine)

Future plans:
- Configure Spectrum Scale cluster
- Configure Spectrum Scale storage
- Configure Spectrum Scale filesystem

Installation
------------

```
$ ansible-galaxy install acch.spectrum-scale
```

Requirements
------------

You will need to download the Spectrum Scale (GPFS) packages from the IBM website, as there's no public repository available. Visit https://www.ibm.com/support/fixcentral and search for 'IBM Spectrum Scale (Software defined storage)'.

Role Variables
--------------

Default variables are defined in `defauls/main.yml`. Either edit this file or define your own variables to override the defaults.

Note that defining the variable `scale_version` is mandatory. Furthermore, you will need to configure an installation method by defining *one* of the following variables:

- `scale_install_repository_url`
- `scale_install_remotepkg_path` (accessible on Ansible managed node)
- `scale_install_localpkg_path` (accessible on Ansible control machine)

Example Playbook
----------------

The simplest possible playbook to install IBM Spectrum Scale on a node:

```
---
- hosts: scale01.example.com
  vars:
    - scale_version: 4.2.3.4
    - scale_install_localpath: /path/to/Spectrum_Scale_Standard-4.2.3.4-x86_64-Linux-install
  roles:
    - acch.spectrum-scale
```

Refer to `defauls/main.yml` for a detailed explanation of possible variables and configuration options.

Copyright and license
---------------------

Copyright 2017 Achim Christ, released under the [MIT license](LICENSE)
