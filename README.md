IBM Spectrum Scale (GPFS) Ansible Role
======================================

Highly-configurable Ansible role to install IBM Spectrum Scale (GPFS)

This project is in very early development. Not ready for production, yet!

Features
--------

- Install IBM Spectrum Scale (GPFS) packages on Linux nodes
- Optionally, verify package integrity by comparing checksums
- Configure SSH public key authentication
- Compile Linux kernel extension
- Create new cluster or extend existing cluster
- Perform (offline) upgrade if daemon is stopped

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

Default variables are defined in `defaults/main.yml`. You'll also find detailed documentation in that file. Define your own variables to override the defaults.

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
    - scale_install_localpkg_path: /path/to/Spectrum_Scale_Standard-4.2.3.4-x86_64-Linux-install
  roles:
    - acch.spectrum-scale
```

In reality you'll want to install IBM Spectrum Scale on a number of nodes, and you'll also want to influence the node roles to ensure high-availability. Note that the cluster will be configured with all nodes in the current play:

```
# hosts:
[cluster01]
scale01  scale_cluster_quorum=true scale_cluster_manager=true
scale02  scale_cluster_quorum=true scale_cluster_manager=true
scale03  scale_cluster_quorum=true scale_cluster_manager=false
scale04  scale_cluster_quorum=false scale_cluster_manager=false
```
```
# playbook.yml:
---
- hosts: cluster01
  vars:
    - scale_version: 4.2.3.4
    - scale_install_repository_url: http://infraserv/gpfs_rpms/
    - scale_cluster_clustername: cluster1.example.com
  roles:
    - acch.spectrum-scale
```

Refer to `defaults/main.yml` for a detailed explanation of possible variables and configuration options.

Copyright and license
---------------------

Copyright 2017 Achim Christ, released under the [MIT license](LICENSE)
