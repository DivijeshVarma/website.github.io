---
title: "Comprehensive Review"
---

### REVIEWING RED HAT SYSTEM ADMINISTRATION II

Before beginning the comprehensive review for this course, you should be comfortable with the
topics covered in each chapter.

You can refer to earlier sections in the textbook for extra study.

**Chapter 1, Improving Command-line Productivity**

Run commands more efficiently by using advanced features of the Bash shell, shell scripts, and
various utilities provided by Red Hat Enterprise Linux.

- Automate sequences of commands by writing a simple shell script.

- Efficiently run commands over lists of items in a script or from the command-line using for loops
and conditionals.

- Find text matching a pattern in log files and command output using the **grep** command and
regular expressions.

**Chapter 2, Scheduling Future Tasks**

Schedule tasks to automatically execute in the future.

- Set up a command that runs once at some point in the future.

- Schedule commands to run on a repeating schedule using a user’s crontab file.

- Schedule commands to run on a repeating schedule using the system crontab file and directories.

- Enable and disable systemd timers, and configure a timer that manages temporary files.

**Chapter 3, Tuning System Performance**

Improve system performance by setting tuning parameters and adjusting scheduling priority of
processes.

- Optimize system performance by selecting a tuning profile managed by the tuned daemon.

- Prioritize or de-prioritize specific processes with the nice and renice commands.

**Chapter 4, Controlling Access to Files with ACLs**

Interpret and set Access Control Lists (ACLs) on files to handle situations requiring complex user
and group access permissions.

- Describe use cases for ACLs, identify files that have ACLs set, and interpret the effect of those
ACLs.

- Set and remove ACLs on files and define default ACLs automatically set by a directory on newly
created files.

**Chapter 5, Managing SELinux Security**

Protect and manage the security of a server by using SELinux.

- Describe how SELinux protects resources and how to select the enforcement mode.

- Configure a file's SELinux context to control how processes interact with that file.

- Configure SELinux booleans to allow runtime policy changes for varying access needs.

- Investigate SELinux log messages and troubleshoot SELinux AVC denials.

**Chapter 6, Managing Basic Storage**

Create and manage storage devices, partitions, file systems, and swap spaces from the command
line.

- Create storage partitions, format them with file systems, and mount them for use.

- Create and manage swap spaces to supplement physical memory.

**Chapter 7, Managing Logical Volumes**

Create and manage logical volumes containing file systems and swap spaces from the command
line.

- Create and manage logical volumes from storage devices, and format them with file systems or
prepare them with swap spaces.

- Add and remove storage assigned to volume groups, and nondestructively extend the size of a
logical volume formatted with a file system.

**Chapter 8, Implementing Advanced Storage Features**

Manage storage using the Stratis local storage management system and use the VDO volumes to
optimize storage space in use.

- Manage multiple storage layers using Stratis local storage management.

- Optimize the use of storage space by using VDO to compress and deduplicate data on storage devices.

**Chapter 9, Accessing Network-Attached Storage**

Access network-attached storage using the NFS protocol.

- Mount, use, and unmount an NFS export from the command line and at boot time.

- Configure the automounter with direct and indirect maps to automatically mount an NFS file
system on demand, and unmount it when it is no longer in use.

- Configure an NFS client to use NFSv4 using the new nfsconf tool.

**Chapter 10, Controlling the Boot Process**

Manage the boot process to control services offered and to troubleshoot and repair problems.

- Describe the Red Hat Enterprise Linux boot process, set the default target used when booting,
and boot a system to a non-default target.

- Log in to a system and change the root password when the current root password has been lost.

- Manually repair file system configuration or corruption issues that stop the boot process.

**Chapter 11, Managing Network Security**

Control network connections to services using the system firewall and SELinux rules.

- Accept or reject network connections to system services using firewalld rules.

- Control whether network services can use specific networking ports by managing SELinux port
labels.

**Chapter 12, Installing Red Hat Enterprise Linux**

Install Red Hat Enterprise Linux on servers and virtual machines.

- Install Red Hat Enterprise Linux on a server.

- Automate the installation process using Kickstart.

- Install a virtual machine on your Red Hat Enterprise Linux server using Cockpit.

### Complete LAB

pdf - 387