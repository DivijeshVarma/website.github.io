---
title: "Implementing Advanced Storage Features"
---

### MANAGING LAYERED STORAGE WITH STRATIS

#### DESCRIBING THE STRATIS ARCHITECTURE

The current local storage solution in Red Hat Enterprise Linux (RHEL) includes many stable
and mature technologies, including the device mapper (dm), the logical volume manager (LVM)
and the XFS file system. Features provided by these components include massively scalable file
systems, snapshots, redundant (RAID) logical devices, multipathing, thin provisioning, caching,
deduplication, and support for virtual machines and containers. Each storage stack layer (dm,
LVM, and XFS) is managed using layer-specific commands and utilities, requiring that system
administrators manage physical devices, fixed-size volumes, and file systems as separate storage
components.

A new generation of storage management solutions appeared in recent years, referred to as
volume-managing file systems, that dynamically and transparently manage the volume layer as
file systems are created and sized. However, although the community development of these file
systems was ongoing for years, none reached the level of feature support and stability required to
become the primary local storage for Red Hat Enterprise Linux.

With RHEL 8, Red Hat introduces the Stratis storage management solution. Instead of developing
from scratch, as other storage projects attempted, Stratis works with existing RHEL storage
components. Stratis runs as a service that manages pools of physical storage devices, and
transparently creates and manages volumes for the file systems being created. Because Stratis
uses existing storage drivers and tools, all of the advanced storage features that you currently use
in LVM, XFS, and the device mapper are also supported by Stratis.

In a volume-managed file system, file systems are built inside shared pools of disk devices using
a concept known as thin provisioning. Stratis file systems do not have fixed sizes and no longer
preallocate unused block space. Although the file system is still built on a hidden LVM volume,
Stratis manages the underlying volume for you and can expand it when needed. The in-use size of
a file system is seen as the amount of actual blocks in use by contained files. The space available
to a file system is the amount of space still unused in the pooled devices on which it resides.
Multiple file systems can reside in the same pool of disk devices, sharing the available space, but
file systems can also reserve pool space to guarantee availability when needed.

Stratis uses stored metadata to recognize managed pools, volumes, and file systems. Therefore,
file systems created by Stratis should never be reformatted or reconfigured manually; they should
only be managed using Stratis tools and commands. Manually configuring Stratis file systems
could cause the loss of that metadata and prevent Stratis from recognizing the file systems it has
created.

You can create multiple pools with different sets of block devices. From each pool, you can create
one or more file systems. Currently, you can create up to 224 file systems per pool. The following
diagram illustrates how the elements of the Stratis storage management solution are positioned.

![Elements of stratis](https://www.linuxprobe.com/links/RH134/images/advstorage/storage-modern.svg)

A pool groups block devices into the data tier and optionally the cache tier. The data tier focuses
on flexibility and integrity and the cache tier focuses on improved performance. Because the
cache tier is intended to improve performance, you should use block devices that have higher
input/output per second (IOPS), such as SSDs.

**Describing the Simplified Storage Stack**

Stratis simplifies many aspects of local storage provisioning and configuration across a range of
Red Hat products. For example, in earlier versions of the **Anaconda** installer, system administrators
had to layer each aspect of disk management over the other. Now, the installer uses Stratis,
simplifying disk setup. Other products that use Stratis include **Cockpit**, Red Hat Virtualization,
and Red Hat Enterprise Linux Atomic Host. For all of these products, Stratis makes it simpler and
less error prone to manage storage space and snapshots. Stratis allows easier integration with the
higher-level management tools than using any CLI programmatically.

![Stratis](https://www.linuxprobe.com/links/RH134/images/advstorage/storage-stratis-in-linux-stack.svg)

**Describing Stratis Layers**

Internally, Stratis uses the Backstore subsystem to manage the block devices, and the
Thinpool subsystem to manage the pools. The Backstore subsystem has a data tier that
maintains the on-disk metadata on block devices, and detects and corrects data corruption. The
cache tier uses high-performance block devices to act as a cache on top of the data tier. The
Thinpool subsystem manages the thin-provisioned volumes associated with the Stratis file
systems. This subsystem uses the dm-thin device mapper driver to replace LVM on the virtual
volume sizing and management. dm-thin creates volumes with a large virtual size, formatted with
XFS, but with a small physical size. As the physical size nears full, Stratis enlarges it automatically.

![layers](https://www.linuxprobe.com/links/RH134/images/advstorage/storage-modern-layers.svg)

**Managing Thin-provisioned File Systems**

To manage the thin-provisioned file systems using the Stratis storage management solution,
install the stratis-cli and stratisd packages. The stratis-cli package provides the **stratis**
command, which translates user requests to the stratisd service via the D-Bus API. The stratisd
package provides the stratisd service, which implements the D-Bus interface, and manages
and monitors the elements of Stratis, such as block devices, pools, and file systems. The D-Bus
API is available if the stratisd service is running.

Install and activate Stratis using the usual tools:

- Install _stratis-cli_ and _stratisd_ using the **yum install** command.

```
[root@host ~]# yum install stratis-cli stratisd
...output omitted...
Is this ok [y/N]: y
...output omitted...
Complete!

```
- Activate the stratisd service using the **systemctl** command.

```
[root@host ~]# systemctl enable --now stratisd

```
The following are common management operations performed using the Stratis storage
management solution.

- Create pools of one or more block devices using the **stratis pool create** command.

```
[root@host ~]# stratis pool create pool1 /dev/vdb

```

Each pool is a subdirectory under the **/stratis** directory.

- Use the **stratis pool list** command to view the list of available pools.

```
[root@host ~]# stratis pool list
Name       Total Physical Size       Total Physical Used
pool1        5 GiB                       52 MiB

```
- Use the **stratis pool add-data** command to add additional block devices to a pool.

[root@host ~]# stratis pool add-data pool1 /dev/vdc

- Use the **stratis blockdev list** command to view the block devices of a pool.

```
[root@host ~]# stratis blockdev list pool1
Pool Name        Device Node         Physical Size       State      Tier  
  pool1             /dev/vdb            5 GiB             In-use     Data
  pool1             /dev/vdc            5 GiB             In-use     Data

```

- Use the **stratis filesystem create** command to create a dynamic and flexible file
system from a pool.

```
[root@host ~]# stratis filesystem create pool1 filesystem1

```
The links to the Stratis file systems are in the **/stratis/pool1** directory.

- Stratis supports file-system snapshotting with the **stratis filesystem snapshot**
command. Snapshots are independent of the source file systems.

```
[root@host ~]# stratis filesystem snapshot pool1 filesystem1 snapshot1

```
- Use the **stratis filesystem list** command to view the list of available file systems.

```
[root@host ~]# stratis filesystem list
...output omitted...

```
To ensure that the Stratis file systems are persistently mounted, edit **/etc/fstab** and specify the
details of the file system. The following command displays the UUID of the file system that you
should use in **/etc/fstab** to identify the file system.

```
[root@host ~]# lsblk --output=UUID /stratis/pool1/filesystem1
UUID        31b9363b-add8-4b46-a4bf-c199cd478c55

```
The following is an example entry in the **/etc/fstab** file to persistently mount a Stratis file
system.

```
UUID=31b9...8c55    /dir1   xfs     defaults,x-systemd.requires=stratisd.service    0 0

```
The **x-systemd.requires=stratisd.service** mount option delays mounting the file system
until after systemd starts the stratisd.service during the boot process.

!!! info "NOTE"

    Failure to use the x-systemd.requires=stratisd.service mount option in /etc/fstab for the Stratis file system will result in the machine booting to emergency.target on the next reboot.

### COMPRESSING AND DEDUPLICATING STORAGE WITH VDO

#### DESCRIBING VIRTUAL DATA OPTIMIZER

Red Hat Enterprise Linux 8 includes the Virtual Data Optimizer (VDO) driver, which optimizes
the data footprint on block devices. VDO is a Linux device mapper driver that reduces disk
space usage on block devices, and minimizes the replication of data, saving disk space and even
increasing data throughput. VDO includes two kernel modules: the kvdo module to transparently
control data compression, and the uds module for deduplication.

The VDO layer is placed on top of an existing block storage device, such as a RAID device or a local
disk. Those block devices can also be encrypted devices. The storage layers, such as LVM logical
volumes and file systems, are placed on top of a VDO device. The following diagram shows the
placement of VDO in an infrastructure consisting of KVM virtual machines that are using optimized
storage devices.

![VDO based virtual machine](https://www.linuxprobe.com/links/RH134/images/advstorage/storage-enhance-virtualdisks.svg)

VDO applies three phases to data in the following order to reduce the footprint on storage
devices:

1. Zero-Block Elimination filters out data blocks that contain only zeroes (0) and records the
information of those blocks only in the metadata. The nonzero data blocks are then passed
to the next phase of processing. This phase enables the thin provisioning feature in the VDO
devices.

2. Deduplication eliminates redundant data blocks. When you create multiple copies of the
same data, VDO detects the duplicate data blocks and updates the metadata to use those
duplicate blocks as references to the original data block without creating redundant data
blocks. The universal deduplication service (UDS) kernel module checks redundancy of the
data through the metadata it maintains. This kernel module ships as part of the VDO.

3. Compression is the last phase. The kvdo kernel module compresses the data blocks using
LZ4 compression and groups them on 4 KB blocks.

#### IMPLEMENTING VIRTUAL DATA OPTIMIZER

The logical devices that you create using VDO are called VDO volumes. VDO volumes are similar
to disk partitions; you can format the volumes with the desired file-system type and mount it like a
regular file system. You can also use a VDO volume as an LVM physical volume.

To create a VDO volume, specify a block device and the name of the logical device that VDO
presents to the user. You can optionally specify the logical size of the VDO volume. The logical size
of the VDO volume can be more than the physical size of the actual block device.

Because the VDO volumes are thinly provisioned, users can only see the logical space in use and
are unaware of the actual physical space available. If you do not specify the logical size while
creating the volume, VDO assumes the actual physical size as the logical size of the volume. This
1:1 ratio of mapping logical size to physical size gives better performance but provides less efficient
use of storage space. Based on your infrastructure requirements, you should prioritize either
performance or space efficiency.

When the logical size of a VDO volume is more than the actual physical size, you should proactively
monitor the volume statistics to view the actual usage using the **vdostats --verbose** command.

**Enabling VDO**

Install the vdo and kmod-kvdo packages to enable VDO in the system.

```
[root@host ~]# yum install vdo kmod-kvdo
...output omitted...
Is this ok [y/N]: y
...output omitted...
Complete!

```
**Creating a VDO Volume**

To create a VDO volume, run the **vdo create** command.

```
[root@host ~]# vdo create --name=vdo1 --device=/dev/vdd --vdoLogicalSize=50G
...output omitted...

```

If you omit the logical size, the resulting VDO volume gets the same size as its physical device.

When the VDO volume is in place, you can format it with the file-system type of your choice and
mount it under the file-system hierarchy on your system.

**Analyzing a VDO Volume**

To analyze a VDO volume, run the vdo status command. This command displays a report on the
VDO system, and the status of the VDO volume in YAML format. It also displays attributes of the
VDO volume. Use the **--name=** option to specify the name of a particular volume. If you omit the
name of the specific volume, the output of the **vdo status** command displays the status of all
the VDO volumes.

```
[root@host ~]# vdo status --name=vdo1
...output omitted...
```
The **vdo list** command displays the list of VDO volumes that are currently started. You can start
and stop a VDO volume using the **vdo start** and **vdo stop** commands, respectively.

### SUMMARY

In this chapter, you learned:

- The Stratis storage management solution implements flexible file systems that grow dynamically
with data.

- The Stratis storage management solution supports thin provisioning, snapshotting, and
monitoring.

- The Virtual Data Optimizer (VDO) aims to reduce the cost of data storage.

- The Virtual Data Optimizer applies zero-block elimination, data deduplication, and data
compression to optimize disk space efficiency.

### LAB

pdf - 247