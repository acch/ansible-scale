IBM Spectrum Scale (GPFS) Ansible Role
======================================

[![Build Status](https://travis-ci.org/acch/ansible-scale.svg?branch=master)](https://travis-ci.org/acch/ansible-scale) [![GitHub Issues](https://img.shields.io/github/issues/acch/ansible-scale.svg)](https://github.com/acch/ansible-scale/issues) [![GitHub Stars](https://img.shields.io/github/stars/acch/ansible-scale.svg?label=github%20%E2%98%85)](https://github.com/acch/ansible-scale/) [![Role Downloads](https://img.shields.io/ansible/role/d/29538.svg)](https://galaxy.ansible.com/acch/spectrum_scale) [![License](https://img.shields.io/github/license/acch/ansible-scale.svg)](LICENSE)

Highly-customizable Ansible role for installing and configuring IBM Spectrum Scale (GPFS)

Particularly looking for [feedback](https://github.com/acch/ansible-scale/issues) and future [requirements](https://github.com/acch/ansible-scale/issues/new)!


Features
--------

- Install Spectrum Scale packages on Linux nodes
- Optionally, verify package integrity by comparing checksums
- Perform (offline) upgrade if daemon is stopped
- Compile-, or install pre-compiled Linux kernel extension
- Configure SSH public key authentication
- Create new-, or extend existing cluster
- Configure Network Shared Disks (NSDs)
- Create new-, or extend existing file systems
- Configure node classes
- Define configuration parameters based on node classes

The following installation methods are available:
- Install from (existing) YUM repository
- Install from remote installation package (accessible on Ansible managed node)
- Install from local installation package (accessible on Ansible control machine)

Future plans:
- Install CES packages
- Install GUI and zimon packages


Installation
------------

```
$ ansible-galaxy install acch.spectrum_scale
```


Requirements
------------

As there's no public repository available, you'll need to download Spectrum Scale (GPFS) packages from the IBM website. Visit https://www.ibm.com/support/fixcentral and search for 'IBM Spectrum Scale (Software defined storage)'.

##### **IBM Spectrum Scale Developer Edition**
If you don't have a subscription you can now download the IBM Spectrum Scale Developer Edition:
- Provides all the features of the IBM Spectrum Scale Data Management Edition but it is limited to 12 TB per cluster.
- There is no support from IBM for Spectrum Scale Developer Edition. Additionally, use in a production environment is prohibited.
- The Developer Edition can be accessed via https://www.ibm.com/us-en/marketplace/scale-out-file-and-object-storage.


Local Repository
-------------------------------

To create a local Spectrum Scale repository on a web server:

```
cd /your/webserver/folder
# Download the installation package. Get the direct link or copy it manually...
wget <Spectrum Scale installation package>
sh ./Spectrum_Scale_Data_Management-5.0.X.X-x86_64-Linux-install --dir ./SpectrumScaleRpms/5.0.X.X/ --silent
cd SpectrumScaleRpms/5.0.X.X/
yum -y install createrepo
createrepo .
```


Role Variables
--------------

Default variables are defined in `defaults/main.yml`. You'll also find detailed documentation in that file. Define your own host variables in your inventory to override the defaults.

Defining the variable `scale_version` is mandatory. Furthermore, you'll need to configure an installation method by defining *one* of the following variables:

- `scale_install_repository_url`
- `scale_install_remotepkg_path` (accessible on Ansible managed node)
- `scale_install_localpkg_path` (accessible on Ansible control machine)


Cluster Membership
------------------

All hosts in the play will be configured as nodes in the same cluster. If you want to add hosts to an existing cluster then add at least one node from that existing cluster to the play.

You can create multiple clusters by running multiple plays.


Example Playbook
----------------

The simplest possible playbook to install Spectrum Scale on a node:

```
---
- hosts: scale01.example.com
  vars:
    - scale_version: 4.2.3.4
    - scale_install_localpkg_path: /path/to/Spectrum_Scale_Standard-4.2.3.4-x86_64-Linux-install
  roles:
    - acch.spectrum_scale
```

This will install all required packages and create a single-node Spectrum Scale cluster.

In reality you'll most probably want to install Spectrum Scale on a number of nodes, and you'll also want to consider the node roles in order to achieve high-availability. The cluster will be configured with all hosts in the current play:

```
# hosts:
[cluster01]
scale01  scale_cluster_quorum=true   scale_cluster_manager=true
scale02  scale_cluster_quorum=true   scale_cluster_manager=true
scale03  scale_cluster_quorum=true   scale_cluster_manager=false
scale04  scale_cluster_quorum=false  scale_cluster_manager=false
scale05  scale_cluster_quorum=false  scale_cluster_manager=false
```
```
# playbook.yml:
---
- hosts: cluster01
  vars:
    - scale_version: 4.2.3.4
    - scale_install_repository_url: http://infraserv/gpfs_rpms/
    - scale_cluster_clustername: cluster01.example.com
  roles:
    - acch.spectrum_scale
```

Refer to `defaults/main.yml` for a detailed explanation of possible variables and configuration options.

Defining node roles such as `scale_cluster_quorum` and `scale_cluster_manager` is optional. If you don't specify any quorum nodes then the first seven hosts in your inventory will automatically be assigned the quorum role.

The above examples will install required packages and create a functional Spectrum Scale cluster which can be used to e.g. mount existing remote file systems. To also create local file systems in the new cluster you'll need to provide additional information. It's suggested to use `host_vars` inventory files for that purpose:

```
# host_vars/scale01:
---
scale_storage:
  - filesystem: gpfs01
    blockSize: 4M
    defaultMetadataReplicas: 2
    defaultDataReplicas: 2
    numNodes: 16
    automaticMountOption: true
    defaultMountPoint: /mnt/gpfs01
    disks:
      - device: /dev/sdb
        nsd: nsd_1
        servers: scale01
        failureGroup: 10
        usage: metadataOnly
        pool: system
      - device: /dev/sdc
        nsd: nsd_2
        servers: scale01
        failureGroup: 10
        usage: dataOnly
        pool: data
```
```
# host_vars/scale02:
---
scale_storage:
  - filesystem: gpfs01
    disks:
      - device: /dev/sdb
        nsd: nsd_3
        servers: scale02
        failureGroup: 20
        usage: metadataOnly
        pool: system
      - device: /dev/sdc
        nsd: nsd_4
        servers: scale02
        failureGroup: 20
        usage: dataOnly
        pool: data
```

Refer to `man mmchfs` and `man mmchnsd` for a description of the above storage parameters.

The `filesystem` name is mandatory, and the `device` variable is mandatory for each of the file system's `disks`. All other file system and disk parameters are optional. Hence, a minimal file system configuration would look like this:

```
# host_vars/scale01:
---
scale_storage:
  - filesystem: gpfs01
    disks:
      - device: /dev/sdb
      - device: /dev/sdc
```
```
# host_vars/scale02:
---
scale_storage:
  - filesystem: gpfs01
    disks:
      - device: /dev/sdb
      - device: /dev/sdc
```

Note that filesystem parameters can be defined as variables for *any* host in the play &mdash; the host for which you define the filesystem parameters is irrelevant. For disk parameters the host is only relevant if you omit the `servers` variable. When omitting the `servers` variable then the host for which you define the disk is automatically considered the (only) NSD server for that particular disk.

> **Important**: `scale_storage` *must* be defined for individual host(s) using `host_vars` inventory files. Do *not* define disk parameters using `group_vars` inventory files or inline `vars:` in your playbook. Doing so would apply them to all hosts in the group/play, thus defining the same disk multiple times...

Furthermore, Spectrum Scale node classes can be defined on a per-node basis by defining the `scale_nodeclass` variable:

```
# host_vars/scale01:
---
scale_nodeclass:
  - classA
  - classB
```
```
# host_vars/scale02:
---
scale_nodeclass:
  - classA
  - classC
```

These node classes can optionally be used to define Spectrum Scale configuration parameters. It's suggested to use `group_vars` inventory files for that purpose:

```
# group_vars/all:
---
scale_config:
  - nodeclass: classA
    params:
      - pagepool: 16G
      - autoload: no
      - ignorePrefetchLUNCount: yes
```

Refer to `man mmchconfig` for a list of available configuration parameters.

Note that configuration parameters can be defined as variables for *any* host in the play &mdash; the host for which you define the configuration parameters is irrelevant.


Limitations
-----------

This role can (currently) be used to create new-, or extend existing clusters. Similarly, new file systems can be created or extended. But this role will *not* remove existing nodes, disks, file systems or node classes &mdash; on purpose! This is also the reason why it can not be used to e.g. change the file system pool of a disk. Changing the pool requires one to remove and then re-add the disk from a file system, which is not currently in scope of this role.

Furthermore, online upgrades are not currently in scope of this role. Spetrum Scale supports rolling online upgrades (by taking down one node at a time), but this requires careful planning and monitoring and might require manual intervention in case of unforeseen problems.


Troubleshooting
---------------

This role stores configuration files in `/var/tmp` on the first host in the play. These configuration files are kept to determine if definitions have changed since the previous run, and to decide if it's necessary to run certain Spectrum Scale commands (again). When experiencing problems one can simply delete these configuration files from `/var/tmp` in order to clear the cache &mdash; this will force re-application of all definitions upon the next run. As a downside, the next run may take longer than expected as it might re-run unnecessary Spectrum Scale commands. Doing so will automatically re-generate the cache.

Please use the [issue tracker](https://github.com/acch/ansible-scale/issues) to ask questions, report bugs and request features.


Copyright and license
---------------------

Copyright 2017 Achim Christ, released under the [MIT license](LICENSE)
