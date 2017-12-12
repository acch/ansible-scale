# ansible-scale

Highly-configurable Ansible role to install IBM Spectrum Scale (GPFS)

This project is in very early development. Not ready for production, yet!

## Features

- Install IBM Spectrum Scale (GPFS) packages on Linux nodes
- Optionally, verify package integrity by comparing md5sum
- Compile Linux kernel extension

The following installation methods are available:
- Install from (existing) YUM repository
- Install from remote installation package (accessible on Ansible managed node)
- Install from local installation package (accessible on Ansible control machine)

Future plans:
- Configure Spectrum Scale cluster
- Configure Spectrum Scale storage
- Configure Spectrum Scale filesystem

## Installation

Using Git:

```
git clone https://github.com/acch/ansible-scale.git roles/scale
```

Using Ansible-Galaxy:

```
TBD
```

## Usage

The simplest possible playbook to install IBM Spectrum Scale on a node:

```
---
- hosts: scale01.example.com
  vars:
    - scale_version: 4.2.3.4
    - scale_install_localpath: /path/to/SpectrumScaleInstaller
  roles:
    - scale
...
```

### Configuration

Default variables are defined in `defauls/main.yml`. Either edit this file or define your own variables to override the defaults.

Note that defining the variable `scale_version` is mandatory. Furthermore, you will need to configure an installation method by defining one of the following variables:

- `scale_install_repository_url`
- `scale_install_remotepkg_path`
- `scale_install_localpkg_path`
