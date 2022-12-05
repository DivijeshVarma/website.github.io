---
title: "Tuning System Performance"
---

### ADJUSTING TUNING PROFILES

#### TUNING SYSTEMS

System administrators can optimize the performance of a system by adjusting various device
settings based on a variety of use case workloads. The tuned daemon applies tuning adjustments
both statically and dynamically, using tuning profiles that reflect particular workload requirements.

**Configuring Static Tuning**

The tuned daemon applies system settings when the service starts or upon selection of a
new tuning profile. Static tuning configures predefined kernel parameters in profiles that
**tuned** applies at runtime. With static tuning, kernel parameters are set for overall performance
expectations, and are not adjusted as activity levels change.

**Configuring Dynamic Tuning**

With dynamic tuning, the tuned daemon monitors system activity and adjusts settings depending
on runtime behavior changes. Dynamic tuning is continuously adjusting tuning to fit the current
workload, starting with the initial settings declared in the chosen tuning profile.

For example, storage devices experience high use during startup and login, but have minimal
activity when user workloads consist of using web browsers and email clients. Similarly, CPU and
network devices experience activity increases during peak usage throughout a workday. The
tuned daemon monitors the activity of these components and adjusts parameter settings to
maximize performance during high-activity times and reduce settings during low activity. The
tuned daemon uses performance parameters provided in predefined tuning profiles.

#### INSTALLING AND ENABLING TUNED

A minimal Red Hat Enterprise Linux 8 installation includes and enables the tuned package by
default. To install and enable the package manually:

```bash
[root@host ~]$ yum install tuned
[root@host ~]$ systemctl enable --now tuned
Created symlink /etc/systemd/system/multi-user.target.wants/tuned.service → /usr/
lib/systemd/system/tuned.service.

```
#### SELECTING A TUNING PROFILE

The **Tuned** application provides profiles divided into the following categories:

- Power-saving profiles

- Performance-boosting profiles

The performance-boosting profiles include profiles that focus on the following aspects:

- Low latency for storage and network
- High throughput for storage and network
- Virtual machine performance
- Virtualization host performance

**Tuning Profiles Distributed with Red Hat Enterprise Linux 8**

TUNED PROFILE | PURPOSE |
---------|----------|
 balanced | Ideal for systems that require a compromise between power saving and performance. |
 desktop | Derived from the balanced profile. Provides faster response of interactive applications. |
 throughput-performance | Tunes the system for maximum throughput. |
 latency-performance | Ideal for server systems that require low latency at the expense of power consumption. |
 network-latency  | Derived from the latency-performance profile. It enables additional network tuning parameters to provide low network latency. |
 network-throughput | Derived from the throughput-performance profile. Additional network tuning parameters are applied for maximum network throughput. |
 powersave | Tunes the system for maximum power saving. |
 oracle | Optimized for Oracle database loads based on the throughput-performance profile. |
 virtual-guest | Tunes the system for maximum performance if it runs on a virtual machine. |
 virtual-host | Tunes the system for maximum performance if it acts as a host for virtual machines.  |

#### MANAGING PROFILES FROM THE COMMAND LINE

The **tuned-adm** command is used to change settings of the tuned daemon. The **tuned-adm**
command can query current settings, list available profiles, recommend a tuning profile for the
system, change profiles directly, or turn off tuning.

A system administrator identifies the currently active tuning profile with **tuned-adm active**.

```bash
[root@host ~]# tuned-adm active
Current active profile: virtual-guest

```
The **tuned-adm** list command lists all available tuning profiles, including both built-in profiles
and custom tuning profiles created by a system administrator.

```bash
[root@host ~]# tuned-adm list
Available profiles:
- balanced
- desktop
- latency-performance
- network-latency
- network-throughput
- powersave
- sap
- throughput-performance
- virtual-guest
- virtual-host
Current active profile: virtual-guest

```
Use **tuned-adm profile profilename** to switch the active profile to a different one that
better matches the system's current tuning requirements.

```bash
[root@host ~]$ tuned-adm profile throughput-performance
[root@host ~]$ tuned-adm active
Current active profile: throughput-performance

```
The **tuned-adm** command can recommend a tuning profile for the system. This mechanism is
used to determine the default profile of a system after installation.

```bash
[root@host ~]$ tuned-adm recommend
virtual-guest

```
!!! info "NOTE"

    The **tuned-adm recommend** output is based on various system characteristics,
    including whether the system is a virtual machine and other predefined categories
    selected during system installation.

To revert the setting changes made by the current profile, either switch to another profile or
deactivate the tuned daemon. Turn off tuned tuning activity with **tuned-adm off**.

```bash
[root@host ~]$ tuned-adm off
[root@host ~]$ tuned-adm active
No current active profile.

```
### MANAGING PROFILES WITH WEB CONSOLE

To manage system performance profiles with Web Console, log in with privileged access. Click the
**Reuse my password for privileged tasks** option. This permits the user to execute commands,
with sudo privileges, that modify system performance profiles.

As a privileged user, click the **Systems** menu option in the left navigation bar. The current active
profile is displayed in the **Performance Profile** field. To select a different profile, click the active
profile link.

In the **Change Performance Profile** user interface, scroll through the profile list to select one that
best suits the system purpose.

To verify changes, return to the main **System** page and confirm that it displays the active profile in
the **Performance Profile** field.

### INFLUENCING PROCESS SCHEDULING

#### LINUX PROCESS SCHEDULING AND MULTITASKING

Modern computer systems range from low-end systems that have single CPUs that can only
execute a single instruction at any instance of time, to high-performing supercomputers with
hundreds of CPUs each and dozens or even hundreds of processing cores on each CPU, allowing
the execution of huge numbers of instructions in parallel. All these systems still have one thing in
common: the need to run more process threads than they have CPUs.

Linux and other operating systems run more processes than there are processing units using a
technique called time-slicing or multitasking. The operating system process scheduler rapidly
switches between processes on a single core, giving the impression that there are multiple
processes running at the same time.

#### RELATIVE PRIORITIES
Different processes have different levels of importance. The process scheduler can be configured
to use different scheduling policies for different processes. The scheduling policy used for most
processes running on a regular system is called SCHED_OTHER (also called SCHED_NORMAL), but
other policies exist for various workload needs.

Since not all processes are equally important, processes running with the SCHED_NORMAL
policy can be given a relative priority. This priority is called the nice value of a process, which are
organized as **40** different levels of niceness for any process.

The nice level values range from -20 (highest priority) to 19 (lowest priority). By default, processes
inherit their nice level from their parent, which is usually 0. Higher nice levels indicate less priority
(the process easily gives up its CPU usage), while lower nice levels indicate a higher priority (the
process is less inclined to give up the CPU). If there is no contention for resources, for example,
when there are fewer active processes than available CPU cores, even processes with a high nice
level will still use all available CPU resources they can. However, when there are more processes
requesting CPU time than available cores, the processes with a higher nice level will receive less
CPU time than those with a lower nice level.

#### SETTING NICE LEVELS AND PERMISSIONS

Since setting a low nice level on a CPU-hungry process might negatively impact the performance
of other processes running on the same system, only the root user may reduce a process nice
level.

Unprivileged users are only permitted to increase nice levels on their own processes. They
cannot lower the nice levels on their processes, nor can they modify the nice level of other users'
processes.

#### REPORTING NICE LEVELS

Several tools display the nice levels of running processes. Process management tools, such as top,
display the nice level by default. Other tools, such as the ps command, display nice levels when
using the proper options.

**Displaying Nice Levels with Top**

Use the **top** command to interactively view and manage processes. The default configuration
displays two columns of interest about nice levels and priorities. The **NI** column displays the
process nice value and the **PR** column displays its scheduled priority. In the **top** interface, the nice
level maps to an internal system priority queue as displayed in the following graphic. For example,
a nice level of -20 maps to 0 in the **PR** column. A nice level of 19 maps to a priority of 39 in the **PR**
column.

![Nice levels are reported by top](https://www.thegeeksearch.com/images/post/_huc6ce938d20c7aa0584ebcd2ff9e55678_9171_7bfe127a835cfcab340438496fb8f8a0.webp)

**Displaying Nice Levels from the Command Line**

The **ps** command displays process nice levels, but only by including the correct formatting options.

The following **ps** command lists all processes with their PID, process name, nice level, and
scheduling class, sorted in descending order by nice level. Processes that display **TS** in the **CLS**
scheduling class column, run under the SCHED_NORMAL scheduling policy. Processes with a dash
(-) as their nice level, run under other scheduling policies and are interpreted as a higher priority
by the scheduler. Details of the additional scheduling policies are beyond the scope of this course.

```bash
[user@host ~]$ ps axo pid,comm,nice,cls --sort=-nice
PID COMMAND     NI CLS
30  khugepaged  19 TS
29  ksmd         5 TS
1   systemd      0 TS
2   kthreadd     0 TS
9   ksoftirqd/0  0 TS
10  rcu_sched    0 TS
11  migration/0  - FF
12 watchdog/0    - FF
...output omitted...

```
#### STARTING PROCESSES WITH DIFFERENT NICE LEVELS

During process creation, a process inherit its parent's nice level. When a process is started from the
command line, it will inherit its nice level from the shell process where it was started. Typically, this
results in new processes running with a nice level of 0.

The following example starts a process from the shell, and displays the process's nice value. Note
the use of the PID option in the **ps** to specify the output requested.

```bash
[user@host ~]$ sha1sum /dev/zero &
[1] 3480
[user@host ~]$ ps -o pid,comm,nice 3480
PID COMMAND
 NI
3480 sha1sum
 0

```
The **nice** command can be used by all users to start commands with a default or higher nice level.
Without options, the **nice** command starts a process with the default nice value of 10.

The following example starts the **sha1sum** command as a background job with the default nice
level and displays the process's nice level:

```bash
[user@host ~]$ nice sha1sum /dev/zero &
[1] 3517
[user@host ~]$ ps -o pid,comm,nice 3517
PID COMMAND    NI
3517 sha1sum   10

```
Use the **-n** option to apply a user-defined nice level to the starting process. The default is to add
10 to the process' current nice level. The following example starts a command as a background job
with a user-defined nice value and displays the process's nice level:

[user@host ~]$ nice -n 15 sha1sum &
[1] 3521
[user@host ~]$ ps -o pid,comm,nice 3521
PID COMMAND    NI
3521 sha1sum   15

!!! danger "IMPORTANT"

    Unprivileged users may only increase the nice level from its current value, to a
    maximum of 19. Once increased, unprivileged users cannot reduce the value to
    return to the previous nice level. The root use may reduce the nice level from any
    current level, to a minimum of -20.

#### CHANGING THE NICE LEVEL OF AN EXISTING PROCESS

The nice level of an existing process can be changed using the **renice** command. This example
uses the PID identifier from the previous example to change from the current nice level of 15 to the
desired nice level of 19.

```bash
[user@host ~]$ renice -n 19 3521
3521 (process ID) old priority 15, new priority 19

```
The **top** command can also be used to change the nice level on a process. From within the **top**
interactive interface, press the r option to access the **renice** command, followed by the PID to
be changed and the new nice level.

### SUMMARY

In this chapter, you learned:

- The tuned service automatically modifies device settings to meet specific system needs based
on a pre-defined selected tuning profile.

- To revert all changes made to system settings by a selected profile, either switch to another
profile or deactivate the tuned service.

- The system assigns a relative priority to a process to determine its CPU access. This priority is
called the **nice** value of a process.

- The **nice** command assigns a priority to a process when it starts. The **renice** command
modifies the priority of a running process.

### LAB

pdf - 86