---
title: "Accessing Network-Attached Storage"
---

### MOUNTING NETWORK-ATTACHED STORAGE WITH NFS

#### MOUNTING AND UNMOUNTING NFS SHARES

NFS, the Network File System, is an internet standard protocol used by Linux, UNIX, and similar
operating systems as their native network file system. It is an open standard, still being actively
enhanced, which supports native Linux permissions and file-system features.

The default NFS version in Red Hat Enterprise Linux 8 is 4.2. NFSv4 and NFSv3 major versions are
supported. NFSv2 is no longer supported. NFSv4 uses only the TCP protocol to communicate with
the server; earlier NFS versions could use either TCP or UDP.

NFS servers export shares (directories). NFS clients mount an exported share to a local mount
point (directory), which must exist. NFS shares can be mounted a number of ways:

- Manually, using the **mount** command.

- Automatically at boot time using **/etc/fstab** entries.

- On demand, using either the autofs service or the systemd.automount facility.

**Mounting NFS Shares**

To mount an NFS share, follow these three steps:

1.  **Identify**: The administrator of the NFS client system can identify available NFS shares in
various ways:

The administrator for the NFS server may provide export details, including security
requirements.

Alternatively, the client administrator can identify NFSv4 shares by mounting the root
directory of the NFS server and exploring the exported directories. Do this as the root user.
Access to shares that use Kerberos security will be denied, but the share (directory) name will
be visible. Other shared directories will be browsable.

```
[user@host ~]$ sudo mkdir mountpoint
[user@host ~]$ sudo mount serverb:/ mountpoint
[user@host ~]$ sudo ls mountpoint
```
2.  **Mount point**: Use **mkdir** to create a mount point in a suitable location.

```
[user@host ~]$ mkdir -p mountpoint

```
3.  **Mount**: As with file systems on partitions, NFS shares must be mounted to be available. To
mount an NFS share, select from the following. In each case, you must run these commands
as a superuser either by logging in as root or by using the **sudo** command.

- Mount temporarily: Mount the NFS share using the **mount** command:

```
[user@host ~]$ sudo mount -t nfs -o rw,sync serverb:/share mountpoint

```
The **-t nfs** option is the file-system type for NFS shares (not strictly required but shown
for completeness). The **-o sync** option tells **mount** to immediately synchronize write
operations with the NFS server (the default is asynchronous).

This command mounts the share immediately but not persistently; the next time the system
boots, this NFS share will not be available. This is useful for one-time access to data. It is
also useful for test mounting an NFS share before making the share available persistently.

- Mount persistently: To ensure that the NFS share is mounted at boot time, edit the **/etc/
fstab** file to add the mount entry.

```
[user@host ~]$ sudo vim /etc/fstab
...
serverb:/share       /mountpoint     nfs     rw,soft     0 0

```
Then, mount the NFS share:

```
[user@host ~]$ sudo mount /mountpoint

```
Because the NFS server and mount options are found in the **/etc/fstab** file by the NFS
client service, you do not need to specify these on the command line.

**Unmounting NFS Shares**

As root (or using **sudo**), unmount an NFS share using the **umount** command.

```
[user@host ~]$ sudo umount mountpoint

```
!!! info "NOTE"

    Unmounting a share does not remove its **/etc/fstab** entry. Unless you remove or
    comment out the entry, the NFS share will be remounted either at the next system
    boot or when the NFS client service is restarted.

#### THE nfsconf TOOL

Red Hat Enterprise Linux 8 introduces the **nfsconf** tool to manage the NFS client and server
configuration files under NFSv4 and NFSv3. Configure the **nfsconf** tool using **/etc/nfs.conf**

(the **/etc/sysconfig/nfs** file from earlier versions of the operating system is deprecated now).
Use the **nfsconf** tool to get, set, or unset NFS configuration parameters.

The **/etc/nfs.conf** configuration file is composed of multiple sections starting with a key word
in square brackets (**[keyword]**) with value assignments within the section. For an NFS server,
configure the **[nfsd]** section. A value assignment or key is composed of a name for the value, an
equals sign, and a setting for the value, such as **vers4.2=y**. The lines starting with "#" or ";" are
ignored, as are any blank lines.

```
[user@host ~]$ sudo cat /etc/nfs.conf
...output omitted...
[nfsd]
# debug=0
# threads=8
# host=
# port=0
# grace-time=90
# lease-time=90
# tcp=y
# vers2=n
# vers3=y
# vers4=y
# vers4.0=y
# vers4.1=y
# vers4.2=y
# rdma=n
#

```
By default the **[nfsd]** section's key-value pairs are commented out. However, the comments
show the default options that will take effect if they are unchanged. This provides you with a good
starting point for NFS configuration.

Use the **nfsconf --set section key** value to set a value for the key in the specified section.

```
[user@host ~]$ sudo nfsconf --set nfsd vers4.2 y

```
This command updates the **/etc/nfs.conf** configuration file:

```
[user@host ~]$ sudo cat /etc/nfs.conf
...output omitted...
[nfsd]
vers4.2 = y
# debug=0
# threads=8
# host=
# port=0
# grace-time=90
# lease-time=90
# tcp=y
# vers2=n
# vers3=y
# vers4=y
# vers4.0=y
# vers4.1=y
# vers4.2=y
# rdma=n
#
```

### AUTOMOUNTING NETWORK-ATTACHED STORAGE

#### MOUNTING NFS SHARES WITH THE AUTOMOUNTER

The automounter is a service (autofs) that automatically mounts NFS shares "on-demand," and
will automatically unmount NFS shares when they are no longer being used.

**Automounter Benefits**

- Users do not need to have root privileges to run the **mount** and **umount** commands.

- NFS shares configured in the automounter are available to all users on the machine, subject to
access permissions.

- NFS shares are not permanently connected like entries in **/etc/fstab**, freeing network and
system resources.

- The automounter is configured on the client side; no server-side configuration is required.

- The automounter uses the same options as the **mount** command, including security options.

- The automounter supports both direct and indirect mount-point mapping, for flexibility in
mount-point locations.

- autofs creates and removes indirect mount points, eliminating manual management.

- NFS is the default automounter network file system, but other network file systems can be
automatically mounted.

- autofs is a service that is managed like other system services.

**Create an automount**

Configuring an automount is a multiple step process:

1.  Install the autofs package.

```
[user@host ~]$ sudo yum install autofs

```
 This package contains everything needed to use the automounter for NFS shares.

2. Add a master map file to **/etc/auto.master.d**. This file identifies the base directory used
for mount points and identifies the mapping file used for creating the automounts.

```
[user@host ~]$ sudo vim /etc/auto.master.d/demo.autofs
```

The name of the master map file is arbitrary (although typically meaningful), but it must have
an extension of **.autofs** for the subsystem to recognize it. You can place multiple entries in
a single master map file; alternatively, you can create multiple master map files each with its
own entries grouped logically.

Add the master map entry, in this case, for indirectly mapped mounts:

/shares      /etc/auto.demo

This entry uses the **/shares** directory as the base for indirect automounts. The **/etc/
auto.demo** file contains the mount details. Use an absolute file name. The **auto.demo** file
needs to be created before starting the autofs service.

3. Create the mapping files. Each mapping file identifies the mount point, mount options, and
source location to mount for a set of automounts.

```
[user@host ~]$ sudo vim /etc/auto.demo

```
The mapping file-naming convention is **/etc/auto.name**, where name reflects the content
of the map.

```
work     -rw,sync      serverb:/shares/work

```

The format of an entry is mount point, mount options, and source location. This example shows
a basic indirect mapping entry. Direct maps and indirect maps using wildcards are covered
later in this section.

- Known as the key in the man pages, the mount point is created and removed automatically
by the autofs service. In this case, the fully qualified mount point is **/shares/work** (see
the master map file). The **/shares** directory and the **/shares/work** directories are
created and removed as needed by the autofs service.

In this example, the local mount point mirrors the server's directory structure, however this
is not required; the local mount point can be named anything. The autofs service does not
enforce a specific naming structure on the client.

- Mount options start with a dash character (-) and are comma-separated with no white
space. Mount options available to a manual mounting of a file system are available when
automounting. In this example, the automounter mounts the share with read/write access
(**rw** option), and the server is synchronized immediately during write operations (**sync**
option).

Useful automounter-specific options include **-fstype=** and **-strict**. Use **fstype** to
specify the file system type, for example, **nfs4** or **xfs**, and use **strict** to treat errors
when mounting file systems as fatal.

- The source location for NFS shares follows the host:/pathname pattern; in this example,
serverb:/shares/work. For this automount to succeed, the NFS server, serverb,
must export the directory with read/write access and the user requesting access must have
standard Linux file permissions on the directory. If serverb exports the directory with
read/only access, then the client will get read/only access even though it requested read/
write access.

4.  Start and enable the automounter service.

Use **systemctl** to start and enable the autofs service.

```
[user@host ~]$ sudo systemctl enable --now autofs
Created symlink /etc/systemd/system/multi-user.target.wants/autofs.service → /
usr/lib/systemd/system/autofs.service.
```
**Direct Maps**

Direct maps are used to map an NFS share to an existing absolute path mount point.

To use directly mapped mount points, the master map file might appear as follows:

```
/-     /etc/auto.direct

```
All direct map entries use /- as the base directory. In this case, the mapping file that contains the
mount details is **/etc/auto.direct**.

The content for the **/etc/auto.direct** file might appear as follows:

```
/mnt/docs    -rw,sync      serverb:/shares/docs

```
The mount point (or key) is always an absolute path. The rest of the mapping file uses the same
structure.

In this example the **/mnt** directory exists, and it is not managed by autofs. The full directory **/
mnt/docs** will be created and removed automatically by the autofs service.

**Indirect Wildcard Maps**

When an NFS server exports multiple subdirectories within a directory, then the automounter can
be configured to access any one of those subdirectories using a single mapping entry.

Continuing the previous example, if serverb:/shares exports two or more subdirectories and
they are accessible using the same mount options, then the content for the **/etc/auto.demo** file
might appear as follows:

```
*     -rw,sync       serverb:/shares/&

```
The mount point (or key) is an asterisk character (*), and the subdirectory on the source location is
an ampersand character (&). Everything else in the entry is the same.

When a user attempts to access **/shares/work**, the key * (which is **work** in this example)
replaces the ampersand in the source location and serverb:/shares/work is mounted. As with
the indirect example, the **work** directory is created and removed automatically by autofs.

### SUMMARY

In this chapter, you learned how to:

- Mount and unmount an NFS export from the command line.

- Configure an NFS export to automatically mount at startup.

- Configure the automounter with direct and indirect maps, and describe their differences.

- Configure NFS clients to use NFSv4 using the new **nfsconf** tool.

### LAB

pdf - 279