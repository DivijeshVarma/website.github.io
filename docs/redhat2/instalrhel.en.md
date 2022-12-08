---
title: "Installing Red Hat Enterprise Linux"
---

### INSTALLING RED HAT ENTERPRISE LINUX

#### SELECTING INSTALLATION MEDIA

Red Hat provides several installation media that you can download from the Customer Portal
website using your active subscription.

- A binary DVD containing Anaconda, the Red Hat Enterprise Linux installation program, and the
BaseOS and AppStream package repositories. These repositories contain the packages needed
to complete the installation without additional material.

- A boot ISO containing Anaconda, but requires a configured network to access package
repositories made available using HTTP, FTP, or NFS.

- A QCOW2 image containing a prebuilt system disk ready to deploy as a virtual machine in cloud
or enterprise virtual environments. QCOW2 (QEMU Copy On Write) is the standard image
format used by Red Hat.

Red Hat provides installation media for four supported processor architectures: x86 64-bit (AMD
and Intel), IBM Power Systems (Little Endian), IBM Z, and ARM 64-bit.

After downloading, burn the DVD or boot ISO to physical media, copy each to a USB flash drive or
similar, or publish each from a network server for automated Kickstart use.

**Building Images with Composer**

Composer is a new tool available in RHEL 8. For specialized use cases, Composer allows
administrators to build custom system images for deployment on cloud platforms or virtual
environments.

Composer uses the Cockpit graphical web console. It can also be invoked from a command line
using the **composer-cli** command.

#### MANUAL INSTALLATION WITH ANACONDA

Using the binary DVD or boot ISO, administrators can install a new RHEL system on a bare-metal
server or a virtual machine. The Anaconda program supports two installation methods:

- The manual installation interacts with the user to query how Anaconda should install and
configure the system.

- The automated installation uses a Kickstart file which tells Anaconda how to install the system. A
later section discusses Kickstart installations in greater detail.

**Installing RHEL with the Graphical Interface**

When you boot the system from the binary DVD or the boot ISO, Anaconda starts as a graphical
application.

At the **Welcome to Red Hat Enterprise Linux 8** screen, select the language to use during
installation. This also sets the default language of the system after installation. Individual users can
select their own account's preferred language after installation.

Anaconda presents the **Installation Summary** window, the central place to customize parameters
before beginning the installation.

![Summary installation](https://www.linuxprobe.com/links/RH134/images/installing/install-summary.png)

From this window, configure the installation parameters by selecting the icons in any order. Select
an item to view or edit. In any item, click **Done** to return to this central screen.

Anaconda marks mandatory items with a triangle warning symbol and message. The orange status
bar at the bottom of the screen reminds you that mandatory items remain to be completed before
the installation can begin.

Complete the following items as needed:

- **Keyboard** - Add additional keyboard layouts.

- **Language Support** - Select additional languages to install.

- **Time & Date** - Select the system's location city by clicking on the interactive map, or select it
from the drop-down list. Specify the local time zone even when using Network Time Protocol (NTP).

- **Installation Source** - Provide the source package location that Anaconda needs for installation.
If using the binary DVD, the installation source field already refers to the DVD.

- **Software Selection** - Select the base environment to install, plus any additional add-ons. The
**Minimal Install** environment installs only the essential packages to run Red Hat Enterprise Linux.

- **Installation Destination** - Select and partition the disks onto which Red Hat Enterprise Linux will
install. This item expects an administrator to comprehend partitioning schemes and file system
selection criteria. The default radio button for automatic partitioning allocates the selected
storage devices using all available space.

- **KDUMP** - Kdump is a kernel feature that collects system memory contents when the kernel
crashes. Red Hat engineers can analyze a kdump to identify the cause of a crash. Use this
Anaconda item to enable or disable Kdump.

- **Network & Host Name** - Detected network connections list on the left. Select a connection to
display its details. To configure the selected network connection, click **Configure**.

- **SECURITY POLICY** - By activating a security policy profile, such as the Payment Card Industry
Data Security Standard (PCI DSS) profile, Anaconda applies restrictions and recommendations,
defined by the selected profile, during installation.

- **System Purpose** - A new installation feature that allocates active system entitlements to match
the intended system use.

After completing the installation configuration, and resolving all warnings, click **Begin Installation**.
Clicking **Quit** aborts the installation without applying any changes to the system.

While the system is installing, complete the following items when they display:

- **Root Password** - The installation program prompts to set a root password. The final stage of
the installation process will not continue until you define a root password.

- **User Creation** - Create an optional non-root account. Maintaining a local, general use account is
a recommended practice. Accounts can also be created after the installation is complete.

Click **Reboot** when the installation is done. Anaconda displays the **Initial Setup** screen, if a
graphical desktop was installed. Accept the license information and optionally register the system
with the subscription manager. You may skip system registration, and perform it later.

**Troubleshooting the Installation**

During a Red Hat Enterprise Linux 8 installation, Anaconda provides two virtual consoles. The first
has five windows provided by the **tmux** software terminal multiplexer. You can access that console
with **Ctrl+Alt+F1**. The second virtual console, which displays by default, shows the Anaconda
graphical interface. You can access it with **Ctrl+Alt+F6**.

In the first virtual console, **tmux** provides a shell prompt in the second window. You may use it
to enter commands to inspect and troubleshoot the system while the installation continues. The
other windows provide diagnostic messages, logs and other information.

The following table lists the keystroke combinations to access the virtual consoles and the **tmux**
windows. For **tmux**, the keyboard shortcuts are performed in two actions: press and release
**Ctrl+b**, then press the number key of the window you want to access. With **tmux**, you can also
use **Alt+Tab** to rotate the current focus between the windows.


KEY SEQUENCE | CONTENTS |
---------|----------|
 Ctrl+Alt+F1 | Access the tmux terminal multiplexer. |
 Ctrl+b 1 | When in tmux, access the main information page for the installation process. |
 Ctrl+b 2 | When in tmux, provide a root shell. Anaconda stores the installation log files in the /tmp file. |
 Ctrl+b 3    |  When in tmux, display the contents of the /tmp/anaconda.log file.   |
 Ctrl+b 4   |  When in tmux, display the contents of the /tmp/storage.log file.   |
  Ctrl+b 5    |   When in tmux, display the contents of the /tmp/program.log file.  |
 Ctrl+Alt+F6   |   Access the Anaconda graphical interface.  |

!!! info "NOTE"

    For compatibility with earlier Red Hat Enterprise Linux versions, the virtual consoles
    from **Ctrl+Alt+F2** through **Ctrl+Alt+F5** also present root shells during
    installation.

### AUTOMATING INSTALLATION WITH KICKSTART

#### CREATING A KICKSTART PROFILE

You can automate the installation of Red Hat Enterprise Linux using a feature called Kickstart.
Using Kickstart, you specify everything Anaconda needs to complete an installation, including
disk partitioning, network interface configuration, package selection, and other parameters, in a
Kickstart text file. By referencing the text file, Anaconda performs the installation without further
user interaction.

!!! info "NOTE"

    Kickstart in Red Hat Enterprise Linux is similar to the Jumpstart facility in Oracle
    Solaris, or to using an unattended Setup answer file for Microsoft Windows.

Kickstart files begin with a list of commands that define how to install the target machine. Lines
starting with **#** characters are comments to be ignored by the installer. Additional sections begin
with a directive, recognized by a **%** first character, and end on a line with the **%end** directive.

The **%packages** section specifies the software to be installed on the target system. Specify
individual packages by name (without versions). Package groups, specified by name or ID, are
recognized by beginning them with an @ character. Environment groups (groups of package
groups) are recognized by beginning them with the @^ characters. Specify modules, streams, and
profiles with @module:stream/profile syntax.

Groups have mandatory, default, and optional components. Normally, Kickstart installs mandatory
and default components. To exclude a package or package group from installation, precede it
with a - character. However, excluded packages or package groups may still install if they are
mandatory dependencies of other requested packages.

A Kickstart configuration commonly uses two additional sections, **%pre** and **%post**, which contain
shell scripting commands that further configure the system. The **%pre** script is executed before
any disk partitioning is done. Typically, this section is used only if actions are required to recognize
or initialize a device before disk partitioning. The **%post** script is executed after the installation is
otherwise completed.

You must specify the primary Kickstart commands before the **%pre, %post**, and **%packages**
sections, but otherwise, you can place these sections in any order in the file.

#### KICKSTART FILE COMMANDS

**Installation Commands**

Define the installation source and how to perform the installation. Each is followed by an example.

- **url**: Specifies the URL pointing to the installation media.

```
url --url="http://classroom.example.com/content/rhel8.0/x86_64/dvd/"

```
**repo**: Specifies where to find additional packages for installation. This option must point to a valid **yum** repository.

```
repo --name="appstream" --baseurl=http://classroom.example.com/content/rhel8.0/x86_64/dvd/AppStream/

```
**text**: Forces a text mode install.
 
- **vnc**: Allows the graphical installation to be viewed remotely over VNC.

`vnc --password=redhat`

**Partitioning Commands**

Define the devices and partitioning scheme to be used.

- **clearpart**: Removes partitions from the system prior to creation of new partitions. By default,
no partitions are removed.

```
clearpart --all --drives=sda,sdb --initlabel

```
**part**: Specifies the size, format, and name of a partition.

```
part /home --fstype=ext4 --label=homes --size=4096 --maxsize=8192 --grow

```
**autopart**: Automatically creates a root partition, a swap partition, and an appropriate boot
partition for the architecture. On large enough drives, this also creates a /home partition.

- **ignoredisk**: Controls Anaconda's access to disks attached to the system.

```
ignoredisk --drives=sdc

```
**bootloader**: Defines where to install the bootloader.

```
bootloader --location=mbr --boot-drive=sda

```
**volgroup, logvol**: Creates LVM volume groups and logical volumes.

```
part pv.01 --size=8192
volgroup myvg pv.01
logvol / --vgname=myvg --fstype=xfs --size=2048 --name=rootvol --grow
logvol /var --vgname=myvg --fstype=xfs --size=4096 --name=varvol
```

- **zerombr**: Initialize disks whose formatting is unrecognized.

**Network Commands**

Define the network features used by the host.

- **network**: Configures network information for target system. Activates network devices in
installer environment.

```
network --device=eth0 --bootproto=dhcp

```
**firewall**: Defines the firewall configuration for the target system.

```
firewall --enabled --service=ssh,http

```
**Location and Security Commands**

Configure settings related to security, language, and regions.

- **lang**: Sets the language to use during installation and the default language of the installed
system. Required.

lang en_US.UTF-8

- **keyboard**: Sets the system keyboard type. Required.

```
keyboard --vckeymap=us --xlayouts=''

```
**timezone**: Defines the timezone, NTP servers, and whether the hardware clock uses UTC.

```
timezone --utc --ntpservers=time.example.com Europe/Amsterdam

```
**authselect**: Sets up authentication options. Options recognized by **authselect** are valid for this command. See authselect(8).

- **rootpw**: Defines the initial **root** password.

```
rootpw --plaintext redhat
or
rootpw --iscrypted $6$KUnFfrTzO8jv.PiH$YlBbOtXBkWzoMuRfb0.SpbQ....XDR1UuchoMG1

```
**selinux**: Sets the SELinux mode for the installed system.

```
selinux --enforcing

```
**services**: Modifies the default set of services to run under the default **systemd** target.

```
services --disabled=network,iptables,ip6tables --enabled=NetworkManager,firewalld

```
**group, user**: Create a local group or user on the system.

```
group --name=admins --gid=10001
user --name=jdoe --gecos="John Doe" --groups=admins --password=changeme --
plaintext

```
**Miscellaneous Commands**

Configure miscellaneous items related to logging during the installation and the host power state
at completion.

- **logging**: This command defines how Anaconda will perform logging during the installation.

```
logging --host=loghost.example.com --level=info

```
**firstboot**: If enabled, the Setup Agent starts the first time the system boots. The initial-setup
package must be installed.

```
firstboot --disabled

```
**reboot, poweroff, halt**: Specify the final action to take when the installation completes.

!!! info "NOTE"

    The **ksverdiff** utility from the pykickstart package is useful for identifying
    changes in Kickstart file syntax between two versions of Red Hat Enterprise Linux or
    Fedora.

    For example, **ksverdiff -f RHEL7 -t RHEL8** identifies changes in syntax from
    RHEL 7 to RHEL 8. Available versions are listed in the top of the file **/usr/lib/
    python3.6/site-packages/pykickstart/version.py**.

#### EXAMPLE KICKSTART FILE

The first part of the file consists of the installation commands, such as disk partitioning and
installation source.

```
#version=RHEL8
ignoredisk --only-use=vda
# System bootloader configuration
bootloader --append="console=ttyS0 console=ttyS0,115200n8 no_timer_check
net.ifnames=0 crashkernel=auto" --location=mbr --timeout=1 --boot-drive=vda
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Use text mode install
text
repo --name="appstream" --baseurl=http://classroom.example.com/content/rhel8.0/
x86_64/dvd/AppStream/
# Use network installation
url --url="http://classroom.example.com/content/rhel8.0/x86_64/dvd/"
# Keyboard layouts
# old format: keyboard us
# new format:
keyboard --vckeymap=us --xlayouts=''
# System language
lang en_US.UTF-8
# Root password
rootpw --plaintext redhat
# System authorization information
auth --enableshadow --passalgo=sha512
# SELinux configuration
selinux --enforcing
firstboot --disable
# Do not configure the X Window System
skipx
# System services
services --disabled="kdump,rhsmcertd" --enabled="sshd,rngd,chronyd"
# System timezone
timezone America/New_York --isUtc
# Disk partitioning information
part / --fstype="xfs" --ondisk=vda --size=10000
```
The second part contains the **%packages** section, detailing which packages and packageshould be installed, and which packages should not be installed.

```
%packages
@core
chrony
cloud-init
dracut-config-generic
dracut-norescue
firewalld
grub2
kernel
rsync
tar
-plymouth

%end

```
The last part contains any **%pre** and **%post** installation scripts.

```
%post --erroronfail
# For cloud images, 'eth0' _is_ the predictable device name, since
# we don't want to be tied to specific virtual (!) hardware
rm -f /etc/udev/rules.d/70*
ln -s /dev/null /etc/udev/rules.d/80-net-name-slot.rules
# simple eth0 config, again not hard-coded to the build hardware
cat > /etc/sysconfig/network-scripts/ifcfg-eth0 << EOF
DEVICE="eth0"
BOOTPROTO="dhcp"
ONBOOT="yes"
TYPE="Ethernet"
USERCTL="yes"
PEERDNS="yes"
IPV6INIT="no"
EOF
%end
```

!!! info "NOTE"

    In a Kickstart file, missing required values cause the installer to interactively prompt
    for an answer or to abort the installation entirely.

#### KICKSTART INSTALLATION STEPS

To successfully automate installation of Red Hat Enterprise Linux, follow these steps:

1. Create a Kickstart file.

2. Publish the Kickstart file to the installer.

3. Boot Anaconda and point it to the Kickstart file.

#### CREATING A KICKSTART FILE

Use either of these methods to create a Kickstart file:

- Use the Kickstart Generator website.

- Use a text editor.

The Kickstart Generator website at https://access.redhat.com/labs/kickstartconfig/ presents dialog boxes for user inputs, and creates a Kickstart directives text file with the user's choices. Each dialog box corresponds to the configurable items in the Anaconda installer.

![kickstarter generator](https://www.linuxprobe.com/links/RH134/images/kickstart/kickstart-generator-1.png)

!!! info "NOTE"

    At the time of writing, the Kickstart Generator website did not provide Red Hat
    Enterprise Linux 8 as a menu option. Red Hat Enterprise Linux 8 Beta was a valid
    selection.

Creating a Kickstart file from scratch is typically too complex, but editing an existing Kickstart file
is common and useful. Every installation creates a **/root/anaconda-ks.cfg** file containing the
Kickstart directives used in the installation. This file makes a good starting point when creating
Kickstart file manually.

**ksvalidator** is a utility that checks for syntax errors in a Kickstart file. It ensures that keywords
and options are properly used, but it does not validate that URL paths, individual packages, or
groups, nor any part of **%post** or **%pre** scripts will succeed. For instance, if the **firewall --**
**disabled** directive is misspelled, **ksvalidator** could produce one of the following errors:

```
[user@host ~]$ ksvalidator /tmp/anaconda-ks.cfg
The following problem occurred on line 10 of the kickstart file:

Unknown command: frewall

[user@host ~]$ ksvalidator /tmp/anaconda-ks.cfg
The following problem occurred on line 10 of the kickstart file:

no such option: --dsabled

```
The pykickstart package provides **ksvalidator**.

#### PUBLISH THE KICKSTART FILE TO ANACONDA

Make the Kickstart file available to the installer by placing it in one of these locations:

- A network server available at install time using FTP, HTTP, or NFS.
- An available USB disk or CD-ROM.
- A local hard disk on the system to be installed.

The installer must access the Kickstart file to begin an automated installation. The most common
automation method uses a network server such as an FTP, web, or NFS server. Network servers
facilitate Kickstart file maintenance because changes can be made once, and then immediately
used for multiple future installations.

Providing Kickstart files on USB or CD-ROM is also convenient. The Kickstart file can be
embedded on the boot media used to start the installation. However, when the Kickstart file is
changed, you must generate new installation media.

Providing the Kickstart file on a local disk allows you to quickly rebuild a system.

#### BOOT ANACONDA AND POINT IT TO THE KICKSTART FILE

Once a Kickstart method is chosen, the installer is told where to locate the Kickstart file by passing
the **inst.ks=LOCATION** parameter to the installation kernel. Some examples:

- inst.ks=http://server/dir/file

- inst.ks=ftp://server/dir/file

- inst.ks=nfs:server:/dir/file

- inst.ks=hd:device:/dir/file

- inst.ks=cdrom:device

![Kickstart](https://www.linuxprobe.com/links/RH134/images/kickstart/boot-kickstart.png)

For virtual machine installations using the **Virtual Machine Manager** or **virt-manager**, the
Kickstart URL can be specified in a box under **URL Options**. When installing physical machines,
boot using installation media and press the **Tab** key to interrupt the boot process. Add an
**inst.ks=LOCATION** parameter to the installation kernel.

### INSTALLING AND CONFIGURING VIRTUAL MACHINES

#### INTRODUCING KVM VIRTUALIZATION

Virtualization is a feature that allows a single physical machine to be divided into multiple virtual
machines (VM), each of which can run an independent operating system.

Red Hat Enterprise Linux 8 supports KVM (Kernel-based Virtual Machine), a full virtualization
solution built into the standard Linux kernel. KVM can run multiple Windows and Linux guest
operating systems.

![kvm virtualization](https://www.linuxprobe.com/links/RH134/images/installing/configure-kvm.svg)

In Red Hat Enterprise Linux, manage KVM with the **virsh** command or with Cockpit's Virtual
Machines tool.

KVM virtual machine technology is available across all Red Hat products, from standalone physical
instances of Red Hat Enterprise Linux to the Red Hat OpenStack Platform:

- Physical hardware systems run Red Hat Enterprise Linux to provide KVM virtualization. Red Hat
Enterprise Linux is typically a thick host, a system that supports VMs while also providing other
local and network services, applications, and management functions.

- Red Hat Virtualization (RHV) provides a centralized web interface that allows administrators to
manage an entire virtual infrastructure. It includes advanced features such as KVM migration,
redundancy, and high availability. A Red Hat Virtualization Hypervisor is a tuned version of
Red Hat Enterprise Linux dedicated to the singular purpose of provisioning and supporting VMs.

- Red Hat OpenStack Platform (RHOSP) provides the foundation to create, deploy, and scale a
public or a private cloud.

Red Hat supports virtual machines running these operating systems:

- Red Hat Enterprise Linux 6 and later
- Microsoft Windows 10 and later
- Microsoft Windows Server 2016 and later

#### CONFIGURING A RED HAT ENTERPRISE LINUX PHYSICAL SYSTEM AS A VIRTUALIZATION HOST

Administrators can configure a Red Hat Enterprise Linux system as a virtualization host,
appropriate for development, testing, training, or when needing to work in multiple operating
systems at the same time.

**Installing the Virtualization Tools**

Install the virt Yum module to prepare a system to become a virtualization host.

```
[root@host ~]# yum module list virt
Name       Stream           Profiles          Summary
 virt     rhel [d][e]       common [d]        Virtualizationmodule

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
[root@host ~]# yum module install virt
...output omitted...

```
**Verifying the System Requirements**

KVM requires either an Intel processor with the Intel VT-x and Intel 64 extensions for x86-
based systems or an AMD processor with the AMD-V and the AMD64 extensions. To verify your
hardware and check the system requirements, use the **virt-host-validate** command.

```
[root@host ~]# virt-host-validate
QEMU:     Checking for hardware virtualization         : PASS
QEMU:     Checking if device /dev/kvm exists           : PASS
QEMU:     Checking if device /dev/kvm is accessible    : PASS
QEMU:     Checking if device /dev/vhost-net exists     : PASS

```
The system must pass all of the validation items in order to be able to be a KVM host.

#### MANAGING VIRTUAL MACHINES WITH COCKPIT

The virt Yum module provides the **virsh** command to manage your virtual machines. The Cockpit
tool provides a web console interface for KVM management and virtual machine creation.

![Cockpit](https://www.linuxprobe.com/links/RH134/images/installing/configure-cockpit-console.png)

Install the cockpit-machines package to add the **Virtual Machines** menu to Cockpit.

```
[root@host ~]# yum install cockpit-machines

```
If Cockpit is not already running, start and enable it.

```
[root@host ~]# systemctl enable --now cockpit.socket

```
To create a new virtual machine with Cockpit, access the **Virtual Machines** menu in the Cockpit
web interface. From there, click **Create VM** and enter the VM configuration in the **Create New
Virtual Machine** window.

![Cockpit2](https://www.linuxtechi.com/wp-content/uploads/2021/11/Create-New-Virtual-Machine-Cockpit.png)

- **Name** sets a domain name for the virtual machine configuration. This name is unrelated to the
network host name you give the system in the installed VM.

- **Installation Source Type** is the method for retrieving the installation ISO file. Choices include
the local file system, or an HTTPS, FTP, or NFS URL.

- **Installation Source** provides the path to the installation source.

- **OS Vendor** and **Operating System** indicates the virtual machine's operating system. The
virtualization layer presents hardware emulation compatible with the operating system chosen.

- **Memory** is the amount of RAM to make available to the new VM.

- **Storage Size** is the disk size to give the new VM. Associate additional disks with the VM after
installation.

- **Immediately Start VM** indicates whether the VM should immediately start after you click
**Create**.

Click **Create** to create the VM and **Install** to start the operating system installation. Cockpit
displays the VM console from which you can install the system.

### SUMMARY

In this chapter, you learned:

- The RHEL 8 binary DVD includes Anaconda and all repositories required for installation.

- The RHEL 8 boot ISO includes the Anaconda installer, accessing repositories over the network
during installation.

- The Kickstart system performs unattended installations.

- Kickstart files can be created using the Kickstart Generator website or by copying and editing **/
root/anaconda-ks.cfg**.

- The virt Yum module provides the packages for a RHEL system to become a virtualization host.

- The cockpit-machines package adds the **Virtual Machines** menu to Cockpit.

### LAB

pdf - 374

