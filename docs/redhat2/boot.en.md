---
title: "Controlling The Boot Process"
---

### SELECTING THE BOOT TARGET

#### DESCRIBING THE RED HAT ENTERPRISE LINUX 8 BOOT PROCESS

Modern computer systems are complex combinations of hardware and software. Starting from an
undefined, powered-down state to a running system with a login prompt requires a large number
of pieces of hardware and software to work together. The following list gives a high-level overview
of the tasks involved for a physical **x86_64** system booting Red Hat Enterprise Linux 8. The list for
**x86_64** virtual machines is roughly the same, but the hypervisor handles some of the hardware-
specific steps in software.

- The machine is powered on. The system firmware, either modern UEFI or older BIOS, runs a
Power On Self Test (POST) and starts to initialize some of the hardware.

Configured using the system BIOS or UEFI configuration screens that you typically reach by
pressing a specific key combination, such as **F2**, early during the boot process.

- The system firmware searches for a bootable device, either configured in the UEFI boot
firmware or by searching for a Master Boot Record (MBR) on all disks, in the order configured in
the BIOS.

Configured using the system BIOS or UEFI configuration screens that you typically reach by
pressing a specific key combination, such as **F2**, early during the boot process.

- The system firmware reads a boot loader from disk and then passes control of the system to
the boot loader. On a Red Hat Enterprise Linux 8 system, the boot loader is the GRand Unified
Bootloader version 2 (GRUB2).

Configured using the **grub2-install** command, which installs GRUB2 as the boot loader on
the disk.

- GRUB2 loads its configuration from the **/boot/grub2/grub.cfg** file and displays a menu
where you can select which kernel to boot.

Configured using the **/etc/grub.d/** directory, the **/etc/default/grub** file, and the **grub2-
mkconfig** command to generate the **/boot/grub2/grub.cfg** file.

- After you select a kernel, or the timeout expires, the boot loader loads the kernel and initramfs
from disk and places them in memory. An **initramfs** is an archive containing the kernel
modules for all the hardware required at boot, initialization scripts, and more. On Red Hat
Enterprise Linux 8, the **initramfs** contains an entire usable system by itself.

Configured using the **/etc/dracut.conf.d/** directory, the **dracut** command, and the
**lsinitrd** command to inspect the **initramfs** file.

- The boot loader hands control over to the kernel, passing in any options specified on the kernel
command line in the boot loader, and the location of the **initramfs** in memory.

Configured using the **/etc/grub.d/** directory, the **/etc/default/grub** file, and the **grub2-
mkconfig** command to generate the **/boot/grub2/grub.cfg** file.

- The kernel initializes all hardware for which it can find a driver in the **initramfs**, then executes
**/sbin/init** from the **initramfs** as PID 1. On Red Hat Enterprise Linux 8, **/sbin/init** is a
link to **systemd**.

Configured using the kernel **init=** command-line parameter.

- The systemd instance from the **initramfs** executes all units for the initrd.target target.
This includes mounting the root file system on disk on to the /sysroot directory.

Configured using **/etc/fstab**

- The kernel switches (pivots) the root file system from **initramfs** to the root file system in **/
sysroot**. systemd then re-executes itself using the copy of systemd installed on the disk.

- systemd looks for a default target, either passed in from the kernel command line or configured
on the system, then starts (and stops) units to comply with the configuration for that target,
solving dependencies between units automatically. In essence, a systemd target is a set of units
that the system should activate to reach the desired state. These targets typically start a text-
based login or a graphical login screen.

Configured using **/etc/systemd/system/default.target** and **/etc/systemd/system/**.

#### REBOOTING AND SHUTTING DOWN

To power off or reboot a running system from the command line, you can use the **systemctl**
command.

**systemctl poweroff** stops all running services, unmounts all file systems (or remounts them
read-only when they cannot be unmounted), and then powers down the system.

**systemctl reboot** stops all running services, unmounts all file systems, and then reboots the
system.

You can also use the shorter version of these commands, **poweroff** and **reboot**, which are
symbolic links to their **systemctl** equivalents.

!!! info "NOTE"

    **systemctl halt** and **halt** are also available to stop the system, but unlike
    **poweroff**, these commands do not power off the system; they bring a system
    down to a point where it is safe to power it off manually.

#### SELECTING A SYSTEMD TARGET

A systemd target is a set of systemd units that the system should start to reach a desired state.
The following table lists the most important targets.

**Commonly Used Targets**


TARGET | PURPOSE |
---------|----------|
 graphical.target | System supports multiple users, graphical- and text-based logins. |
 multi-user.target | System supports multiple users, text-based logins only. |
 rescue.target | sulogin prompt, basic system initialization completed |
  emergency.target    |   sulogin prompt, initramfs pivot complete, and system root mounted on / read only.  |

A target can be a part of another target. For example, the graphical.target includes multi-
user.target, which in turn depends on basic.target and others. You can view these
dependencies with the following command.

```
[user@host ~]$ systemctl list-dependencies graphical.target | grep target
graphical.target
*     └─multi-user.target
*      ├─basic.target
*      │ ├─paths.target
*      │ ├─slices.target
*      │ ├─sockets.target
*      │ ├─sysinit.target
*      │ │ ├─cryptsetup.target
*      │ │ ├─local-fs.target
*      │ │ └─swap.target

```
To list the available targets, use the following command.

```
[user@host ~]$ systemctl list-units --type=target --all
UNIT               LOAD      ACTIVE        SUB            DESCRIPTION
---------------------------------------------------------------------------
basic.target       loaded    active      active           Basic System    
cryptsetup.target  loaded    active      active           Local Encrypted Volumes
emergency.target   loaded    inactive    dead             Emergency Mode
getty-pre.target   loaded    inactive    dead             Login Prompts (Pre)
getty.target       loaded    active      active           Login Prompts
graphical.target   loaded    inactive    dead             Graphical Interface

```
**Selecting a Target at Runtime**

On a running system, administrators can switch to a different target using the **systemctl
isolate** command.

```
[root@host ~]# systemctl isolate multi-user.target

```
Isolating a target stops all services not required by that target (and its dependencies), and starts
any required services not yet started.

Not all targets can be isolated. You can only isolate targets that have AllowIsolate=yes set in
their unit files. For example, you can isolate the graphical target, but not the cryptsetup target.

```
[user@host ~]$ systemctl cat graphical.target
# /usr/lib/systemd/system/graphical.target
...output omitted...
[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
[user@host ~]$ systemctl cat cryptsetup.target
# /usr/lib/systemd/system/cryptsetup.target
...output omitted...
[Unit]
Description=Local Encrypted Volumes
Documentation=man:systemd.special(7)

```
**Setting a Default Target**

When the system starts, systemd activates the default.target target. Normally the default
target in /etc/systemd/system/ is a symbolic link to either graphical.target or multi-
user.target. Instead of editing this symbolic link by hand, the **systemctl** command provides
two subcommands to manage this link: **get-default** and **set-default**.

```
[root@host ~]# systemctl get-default
multi-user.target
[root@host ~]# systemctl set-default graphical.target
Removed /etc/systemd/system/default.target.
Created symlink /etc/systemd/system/default.target -> /usr/lib/systemd/system/
graphical.target.
[root@host ~]# systemctl get-default
graphical.target

```
**Selecting a Different Target at Boot Time**

To select a different target at boot time, append the **systemd.unit=target.target** option to
the kernel command line from the boot loader.

For example, to boot the system into a rescue shell where you can change the system
configuration with almost no services running, append the following option to the kernel command
line from the boot loader.

```
systemd.unit=rescue.target

```
This configuration change only affects a single boot, making it a useful tool for troubleshooting
the boot process.

To use this method of selecting a different target, use the following procedure:

1. Boot or reboot the system.

2. Interrupt the boot loader menu countdown by pressing any key (except **Enter** which would
initiate a normal boot).

3. Move the cursor to the kernel entry that you want to start.

4. Press **e** to edit the current entry.

5. Move the cursor to the line that starts with **linux**. This is the kernel command line.

6. Append **systemd.unit=target.target**. For example,
**systemd.unit=emergency.target**.

7. Press **Ctrl+x** to boot with these changes.

### RESETTING THE ROOT PASSWORD

#### RESETTING THE ROOT PASSWORD FROM THE BOOT LOADER

One task that every system administrator should be able to accomplish is resetting a lost root
password. If the administrator is still logged in, either as an unprivileged user but with full **sudo**
access, or as root, this task is trivial. When the administrator is not logged in, this task becomes
slightly more involved.

Several methods exist to set a new root password. A system administrator could, for example,
boot the system using a Live CD, mount the root file system from there, and edit **/etc/shadow**. In
this section, we explore a method that does not require the use of external media.

!!! info "NOTE"

    On Red Hat Enterprise Linux 6 and earlier, administrators can boot the system into
    runlevel 1 to get a root prompt. The closest analogs to runlevel 1 on a Red Hat
    Enterprise Linux 8 machine are the rescue and emergency targets, both of which
    require the root password to log in.

On Red Hat Enterprise Linux 8, it is possible to have the scripts that run from the **initramfs**
pause at certain points, provide a root shell, and then continue when that shell exits. This is
mostly meant for debugging, but you can also use this method to reset a lost root password.

To access that root shell, follow these steps:

1. Reboot the system.

2. Interrupt the boot loader countdown by pressing any key, except **Enter**.

3. Move the cursor to the kernel entry to boot.

4. Press **e** to edit the selected entry.

5. Move the cursor to the kernel command line (the line that starts with **linux**).

6. Append **rd.break**. With that option, the system breaks just before the system hands control
from the **initramfs** to the actual system.

7. Press **Ctrl+x** to boot with the changes.

At this point, the system presents a root shell, with the actual root file system on the disk
mounted read-only on **/sysroot**. Because troubleshooting often requires modification to the
root file system, you need to change the root file system to read/write. The following step shows
how the **remount,rw** option to the **mount** command remounts the file system with the new
option (**rw**) set.

!!! info "NOTE"

    Prebuilt images may place multiple **console=** arguments to the kernel to support
    a wide array of implementation scenarios. Those **console=** arguments indicate the
    devices to use for console output. The caveat with **rd.break** is that even though
    the system sends the kernel messages to all the consoles, the prompt ultimately
    uses whichever console is given last. If you do not get your prompt, you may want to
    temporarily reorder the **console=** arguments when you edit the kernel command
    line from the boot loader.

!!! danger "IMPORTANT"

    The system has not yet enabled SELinux, so any file you create does not have
    an SELinux context. Some tools, such as the **passwd** command, first create a
    temporary file, then move it in place of the file they are intended to edit, effectively
    creating a new file without an SELinux context. For this reason, when you use the
    **passwd** command with **rd.break**, the **/etc/shadow** file does not get an SELinux
    context.

To reset the root password from this point, use the following procedure:

1. Remount **/sysroot** as read/write.

```
switch_root:/# mount -o remount,rw /sysroot

```
2. Switch into a **chroot** jail, where **/sysroot** is treated as the root of the file-system tree.

```
switch_root:/# chroot /sysroot

```
3. Set a new root password.

```
sh-4.4# passwd root

```

4. Make sure that all unlabeled files, including **/etc/shadow** at this point, get relabeled during
boot.

```
sh-4.4# touch /.autorelabel

```
5. Type **exit** twice. The first command exits the **chroot** jail, and the second command exits the
**initramfs** debug shell.

At this point, the system continues booting, performs a full SELinux relabel, and then reboots
again.

#### INSPECTING LOGS

Looking at the logs of previously failed boots can be useful. If the system journals are persistent
across reboots, you can use the **journalctl** tool to inspect those logs.

Remember that by default, the system journals are kept in the **/run/log/journal** directory,
which means the journals are cleared when the system reboots. To store journals in the **/var/log/journal** directory, which persists across reboots, set the Storage parameter to **persistent** in **/etc/systemd/journald.conf**.

```
[root@host ~]# vim /etc/systemd/journald.conf
...output omitted...
[Journal]
Storage=persistent
...output omitted...
[root@host ~]# systemctl restart systemd-journald.service

```

To inspect the logs of a previous boot, use the **-b** option of **journalctl**. Without any arguments,
the **-b** option only displays messages since the last boot. With a negative number as an argument,
it displays the logs of previous boots.

```
[root@host ~]# journalctl -b -1 -p err

```
This command shows all messages rated as an error or worse from the previous boot.

#### REPAIRING SYSTEMD BOOT ISSUES

To troubleshoot service startup issues at boot time, Red Hat Enterprise Linux 8 makes the
following tools available.

**Enabling the Early Debug Shell**

By enabling the debug-shell service with **systemctl enable debug-shell.service**, the
system spawns a root shell on **TTY9 (Ctrl+Alt+F9)** early during the boot sequence. This shell is
automatically logged in as root, so that administrators can debug the system while the operating
system is still booting.

!!! danger "WARNING"

    Do not forget to disable the debug-shell.service service when you are done
    debugging, because it leaves an unauthenticated root shell open to anyone with
    local console access.

**Using the Emergency and Rescue Targets**

By appending either **systemd.unit=rescue.target** or **systemd.unit=emergency.target** to the kernel command line from the boot loader, the system spawns into a rescue or emergency shell instead of starting normally. Both of these shells require the root password.

The emergency target keeps the root file system mounted read-only, while the rescue target waits
for sysinit.target to complete, so that more of the system is initialized, such as the logging
service or the file systems. The root user at this point can not make changes to /etc/fstab until the
drive is remounted in a read write state **mount -o remount,rw /**

Administrators can use these shells to fix any issues that prevent the system from booting
normally; for example, a dependency loop between services, or an incorrect entry in **/etc/fstab**.
Exiting from these shells continues with the regular boot process.

**Identifying Stuck Jobs**

During startup, systemd spawns a number of jobs. If some of these jobs cannot complete,
they block other jobs from running. To inspect the current job list, administrators can use the
**systemctl list-jobs** command. Any jobs listed as running must complete before the jobs
listed as waiting can continue.

### REPAIRING FILE SYSTEM ISSUES AT BOOT

#### DIAGNOSING AND FIXING FILE SYSTEM ISSUES

Errors in **/etc/fstab** and corrupt file systems can stop a system from booting. In most cases,
systemd drops to an emergency repair shell that requires the root password.

The following table lists some common errors and their results.

**Common File System Issues**


PROBLEM | RESULT |
---------|----------|
 Corrupt file system | systemd attempts to repair the file system. If the problem is too severe for an automatic fix, the system drops the user to an emergency shell. |
 Nonexistent device or UUID referenced in /etc/fstab | systemd waits for a set amount of time, waiting for the device to become available. If the device does not become available, the system drops the user to an emergency shell after the timeout. |
 Nonexistent mount point in /etc/fstab | The system drops the user to an emergency shell. |
 Incorrect mount option specified in /etc/fstab  | The system drops the user to an emergency shell.  |

!!! info "NOTE"

    When using the emergency shell to address file-system issues, do not forget to run
    **systemctl daemon-reload** after editing **/etc/fstab**. Without this reload,
    systemd may continue using the old version.

The **nofail** option in an entry in the **/etc/fstab** file permits the system to boot even if the
mount of that file system is not successful. Do not use this option under normal circumstances.
With **nofail**, an application can start with its storage missing, with possibly severe consequences.

### SUMMARY

In this chapter, you learned:

- **systemctl reboot** and **systemctl poweroff** reboot and power down a system,
respectively.

- **systemctl isolate target-name.target** switches to a new target at runtime.

- **systemctl get-default** and **systemctl set-default** can be used to query and set the
default target.

- Use **rd.break** on the kernel command line to interrupt the boot process before control is
handed over from the **initramfs**. The root file system is mounted read-only under **/sysroot**.

- The emergency target can be used to diagnose and fix file-system issues.

### LAB

pdf - 307