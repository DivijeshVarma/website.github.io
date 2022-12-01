---
title: "Monitoring And Managing Linux Processes"
---

### LISTING PROCESSES ###

#### DEFINITION OF A PROCESS ####

A process is a running instance of a launched, executable program. A process consists of:

- An address space of allocated memory

- Security properties including ownership credentials and privileges

- One or more execution threads of program code

- Process state

The *environment* of a process includes:

- Local and global variables

- A current scheduling context

- Allocated system resources, such as file descriptors and network ports

An existing (*parent*) process duplicates its own address space (**fork**) to create a new (*child*)
process structure. Every new process is assigned a unique process ID (PID) for tracking and
security. The PID and the parent's process ID (PPID) are elements of the new process environment.
Any process may create a child process. All processes are descendants of the first system process,
**systemd** on a Red Hat Enterprise Linux 8 system).

![Process life cycle](https://www.linuxprobe.com/links/RH124/images/processes/lifecycle.png "Process life cycle")

Through the *fork* routine, a child process inherits security identities, previous and current file
descriptors, port and resource privileges, environment variables, and program code. A child
process may then *exec* its own program code. Normally, a parent process *sleeps* while the child
process runs, setting a request (*wait*) to be signaled when the child completes. Upon exit, the
child process has already closed or discarded its resources and environment. The only remaining
resource, called a *zombie*, is an entry in the process table. The parent, signaled awake when the
child exited, cleans the process table of the child's entry, thus freeing the last resource of the child
process. The parent process then continues with its own program code execution.

### DESCRIBING PROCESS STATES ###

In a multitasking operating system, each CPU (or CPU core) can be working on one process at a
single point in time. As a process runs, its immediate requirements for CPU time and resource
allocation change. Processes are assigned a *state*, which changes as circumstances dictate.

![Linux process states](https://www.linuxprobe.com/links/RH124/images/processes/states.png "Linux process states")

Linux process states are illustrated in the previous diagram and described in the following table:

**Linux Process States**

| NAME     | FLAG  | KERNEL-DEFINED STATE NAME AND DESCRIPTION                                                                                                                                                                                                                                          |
|:--------:|:------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Running  | **R** | TASK_RUNNING: The process is either executing on a CPU or waiting to run.Process can be executing user routines or kernel routines (system calls), or be queued and ready when in the Running (or Runnable) state.                                                                 |
| Sleeping | **S** | TASK_INTERRUPTIBLE: The process is waiting for some condition: a hardware request, system resource access, or signal. When an event or signal satisfies the condition, the process returns to *Running*.                                                                           |
| Sleeping | **D** | TASK_UNINTERRUPTIBLE: This process is also Sleeping, but unlike **S** state, does not respond to signals. Used only when process interruption may cause an unpredictable device state.                                                                                             |
| Sleeping | **K** | TASK_KILLABLE: Identical to the uninterruptible **D** state, but modified to allow a waiting task to respond to the signal that it should be killed (exit completely). Utilities frequently display Killable processes as **D** state.                                             |
| Sleeping | **I** | TASK_REPORT_IDLE: A subset of state **D**. The kernel does not count these processes when calculating load average. Used for kernel threads. Flags TASK_UNINTERRUPTABLE and TASK_NOLOAD are set. Similar to TASK_KILLABLE, also a subset of state **D**. It accepts fatal signals. |
| Stopped  | **T** | TASK_STOPPED: The process has been *Stopped* (suspended), usually by being signaled by a user or another process. The process can be continued (resumed) by another signal to return to *Running*.                                                                                 |
| Stopped  | **T** | TASK_TRACED: A process that is being debugged is also temporarily Stopped and shares the same **T** state flag.                                                                                                                                                                    |
| Zombie   | **Z** | EXIT_ZOMBIE: A child process signals its parent as it exits. All resources except for the process identity (PID) are released.                                                                                                                                                     |
| Zombie   | **X** | EXIT_DEAD: When the parent cleans up (reaps) the remaining child process structure, the process is now released completely. This state will never be observed in process-listing utilities.                                                                                        |

**Why Process States are Important**

When troubleshooting a system, it is important to understand how the kernel communicates with
processes and how processes communicate with each other. At process creation, the system
assigns the process a state. The *S* column of the top command or the *STAT* column of the *ps* show
the state of each process. On a single CPU system, only one process can run at a time. It is possible
to see several processes with a state of *R*. However, not all of them will be running consecutively,
some of them will be in status *waiting*.

``` bash
[user@host ~]$ top
PID USER  PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
  1 root  20   0  244344  13684   9024 S   0.0   0.7   0:02.46 systemd
  2 root  20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd
...output omitted... 
```

``` bash
[user@host ~]$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
...output omitted...
root         2  0.0  0.0      0     0 ?        S    11:57   0:00 [kthreadd]
student   3448  0.0  0.2 266904  3836 pts/0    R+   18:07   0:00 ps aux
...output omitted... 
```

Process can be suspended, stopped, resumed, terminated, and interrupted using signals. Signals
are discussed in more detail later in this chapter. Signals can be used by other processes, by the
kernel itself, or by users logged into the system.

### LISTING PROCESSES ###

The **ps** command is used for listing current processes. It can provide detailed process information,
including:

- User identification (UID), which determines process privileges

- Unique process identification (PID)

- CPU and real time already expended

- How much memory the process has allocated in various locations

- The location of process **stdout**, known as the controlling terminal

- The current process state

!!! danger "IMPORTANT"

	The Linux version of **ps** supports three option formats:
	
	- UNIX (POSIX) options, which may be grouped and must be preceded by a dash
	
	- BSD options, which may be grouped and must not be used with a dash
	
	- GNU long options, which are preceded by two dashes
	
	For example, **ps -aux** is not the same as **ps aux**.
	
Perhaps the most common set of options, **aux**, displays all processes including processes without
a controlling terminal. A long listing (options **lax**) provides more technical detail, but may display
faster by avoiding user name lookups. The similar UNIX syntax uses the options **-ef** to display all
processes.

``` bash
[user@host ~]$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.1  51648  7504 ?        Ss   17:45   0:03 /usr/lib/systemd/
syst
root         2  0.0  0.0      0     0 ?        S    17:45   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    17:45   0:00 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   17:45   0:00 [kworker/0:0H]
root         7  0.0  0.0      0     0 ?        S    17:45   0:00 [migration/0]
...output omitted...
[user@host ~]$ ps lax
F   UID   PID  PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY     TIME COMMAND
4     0     1     0  20   0  51648  7504 ep_pol Ss   ?       0:03 /usr/lib/
systemd/
1     0     2     0  20   0      0     0 kthrea S    ?       0:00 [kthreadd]
1     0     3     2  20   0      0     0 smpboo S    ?       0:00 [ksoftirqd/0]
1     0     5     2   0 -20      0     0 worker S<   ?       0:00 [kworker/0:0H]
1     0     7     2 -100  -      0     0 smpboo S    ?       0:00 [migration/0]
...output omitted...
[user@host ~]$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 17:45 ?        00:00:03 /usr/lib/systemd/systemd --
switched-ro
root         2     0  0 17:45 ?        00:00:00 [kthreadd]
root         3     2  0 17:45 ?        00:00:00 [ksoftirqd/0]
root         5     2  0 17:45 ?        00:00:00 [kworker/0:0H]
root         7     2  0 17:45 ?        00:00:00 [migration/0]
...output omitted...
```

By default, **ps** with no options selects all processes with the same effective user ID (EUID) as the
current user, and which are associated with the same terminal where **ps** was invoked.

- Processes in brackets (usually at the top of the list) are scheduled kernel threads.

- Zombies are listed as exiting or defunct.

- The output of **ps** displays once. Use top for a process display that dynamically updates.

- **ps** can display in tree format so you can view relationships between parent and child processes.

- The default output is sorted by process ID number. At first glance, this may appear to be
chronological order. However, the kernel reuses process IDs, so the order is less structured than
it appears. To sort, use the **-O** or **--sort** options. Display order matches that of the system
process table, which reuses table rows as processes die and new ones are created. Output may
appear chronological, but is not guaranteed unless explicit **-O** or **--sort** options are used.

### CONTROLLING JOBS ###

#### DESCRIBING JOBS AND SESSIONS ####

*Job control* is a feature of the shell which allows a single shell instance to run and manage multiple
commands.

A job is associated with each pipeline entered at a shell prompt. All processes in that pipeline are
part of the job and are members of the same process group. If only one command is entered at a
shell prompt, that can be considered to be a minimal “pipeline” of one command, creating a job
with only one member.

Only one job can read input and keyboard generated signals from a particular terminal window at a
time. Processes that are part of that job are *foreground* processes of that *controlling terminal*.

A background process of that controlling terminal is a member of any other job associated with that
terminal. Background processes of a terminal cannot read input or receive keyboard generated
interrupts from the terminal, but may be able to write to the terminal. A job in the background may
be stopped (suspended) or it may be running. If a running background job tries to read from the
terminal, it will be automatically suspended.

Each terminal is its own session, and can have a foreground process and any number of
independent background processes. A job is part of exactly one session: the one belonging to its
controlling terminal.

The **ps** command shows the device name of the controlling terminal of a process in the **TTY**
column. Some processes, such as system daemons, are started by the system and not from a shell
prompt. These processes do not have a controlling terminal, are not members of a job, and cannot
be brought to the foreground. The **ps** command displays a question mark (**?**) in the **TTY** column for
these processes.

#### RUNNING JOBS IN THE BACKGROUND ####

Any command or pipeline can be started in the background by appending an ampersand (**&**) to
the end of the command line. The Bash shell displays a job number (unique to the session) and the
PID of the new child process. The shell does not wait for the child process to terminate, but rather
displays the shell prompt.

``` bash
[user@host ~]$ sleep 10000 &
[1] 5947
[user@host ~]$ 
```

!!! info "NOTE"

	When a command line containing a pipe is sent to the background using an
	ampersand, the PID of the last command in the pipeline is used as output. All
	processes in the pipeline are still members of that job.
	
	``` bash
	[user@host ~]$  example_command | sort | mail -s "Sort output" &
	[1] 5998
	```

You can display the list of jobs that Bash is tracking for a particular session with the **jobs**
command.

``` bash
[user@host ~]$ jobs
[1]+  Running                 sleep 10000 &
[user@host ~]$ 
```

A background job can be brought to the foreground by using the **fg** command with its job ID (%job
number).

``` bash
[user@host ~]$ fg %1
sleep 10000
```

In the preceding example, the **sleep** command is now running in the foreground on the controlling
terminal. The shell itself is again asleep, waiting for this child process to exit.

To send a foreground process to the background, first press the keyboard generated suspend
request (**Ctrl+Z**) in the terminal.

``` bash
sleep 10000
^Z
[1]+  Stopped                 sleep 10000
[user@host ~]$ 
```

The job is immediately placed in the background and is suspended.

The **ps j** command displays information relating to jobs. The PID is the unique process ID of the
process. THe PPID is the PID of the parent process of this process, the process that started (forked)
it. The PGID is the PID of the process group leader, normally the first process in the job's pipeline.
The SID is the PID of the session leader, which (for a job) is normally the interactive shell that is
running on its controlling terminal. Since the example **sleep** command is currently suspended, its
process state is **T**.

``` bash
[user@host ~]$ ps j
 PPID   PID  PGID   SID TTY      TPGID STAT   UID   TIME COMMAND
 2764  2768  2768  2768 pts/0     6377 Ss    1000   0:00 /bin/bash
 2768  5947  5947  2768 pts/0     6377 T     1000   0:00 sleep 10000
 2768  6377  6377  2768 pts/0     6377 R+    1000   0:00 ps j
```

To start the suspended process running in the background, use the **bg** command with the same job
ID.

``` bash
[user@host ~]$ bg %1
[1]+ sleep 10000 &
```

The shell will warn a user who attempts to exit a terminal window (session) with suspended jobs. If
the user tries exiting again immediately, the suspended jobs are killed.

!!! info "NOTE"

	Note the + sign after the [1] in the examples above. The + sign indicates that this
	job is the current default job. That is, if a command is used that expects a %job
	number argument and a job number is not provided, then the action is taken on the
	job with the + indicator.

### KILLING PROCESSES ###

#### PROCESS CONTROL USING SIGNALS ####

A signal is a software interrupt delivered to a process. Signals report events to an executing
program. Events that generate a signal can be an error, external event (such as an I/O request or
an expired timer), or by explicit request (the use of a signal-sending command or by a keyboard
sequence).

The following table lists the fundamental signals used by system administrators for routine process
management. Refer to signals by either their short (HUP) or proper (SIGHUP) name.

**Fundamental Process Management Signals**

| SIGNAL NUMBER | SHORT NAME | DEFINITION         | PURPOSE                                                                                                                                                        |
|:-------------:|:-----------|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1             | HUP        | Hangup             | Used to report termination of the controlling process of a terminal. Also used to request process reinitialization (configuration reload) without termination. |
| 2             | INT        | Keyboard interrupt | Causes program termination. Can be blocked or handled. Sent by pressing INTR key combination (**Ctrl+C**).                                                     |
| 3             | QUIT       | Keyboard quit      | Similar to SIGINT, but also produces a process dump at termination. Sent by pressing QUIT key combination (**'Ctrl+\'**).                                      |
| 9             | KILL       | Kill, unblockable  | Causes abrupt program termination. Cannot be blocked, ignored, or handled; always fatal.                                                                       |
| 15            | TERM       | Terminate          | Causes program termination. Unlike SIGKILL, can be blocked, ignored, or handled. The “polite” way to ask a program to terminate; allows self-cleanup.          |
| 18            | CONT       | Continue           | Sent to a process to resume, if stopped. Cannot be blocked. Even if handled, always resumes the process.                                                       |
| 19            | STOP       | Stop, unblockable  | Suspends the process. Cannot be blocked or handled.                                                                                                            |
| 20            | TSTP       | Keyboard stop      | Unlike SIGSTOP, can be blocked, ignored, or handled. Sent by pressing SUSP key combination (**Ctrl+Z**).                                                                                                                                                               |

!!! info "NOTE"

	Signal numbers vary on different Linux hardware platforms, but signal names and
	meanings are standardized. For command use, it is advised to use signal names
	instead of numbers. The numbers discussed in this section are for x86_64 systems.

Each signal has a default action, usually one of the following:

- Term - Cause a program to terminate (exit) at once.

- Core - Cause a program to save a memory image (core dump), then terminate.

- Stop - Cause a program to stop executing (suspend) and wait to continue (resume).

Programs can be prepared to react to expected event signals by implementing handler routines to
ignore, replace, or extend a signal's default action.

**Commands for Sending Signals by Explicit Request**

You signal their current foreground process by pressing a keyboard control sequence to suspend
(**Ctrl+Z**), kill (**Ctrl+C**), or core dump (**'Ctrl+\'**) the process. However, you will use signal-sending
commands to send signals to a background process or to processes in a different session.

Signals can be specified as options either by name (for example, **-HUP** or **-SIGHUP**) or by number
(the related **-1**). Users may kill their own processes, but root privilege is required to kill processes
owned by others.

The **kill** command sends a signal to a process by PID number. Despite its name, the kill command
can be used for sending any signal, not just those for terminating programs. You can use the **kill
-l** command to list the names and numbers of all available signals.

``` bash
[user@host ~]$ kill -l
 1) SIGHUP      2) SIGINT      3) SIGQUIT     4) SIGILL      5) SIGTRAP
 6) SIGABRT     7) SIGBUS      8) SIGFPE      9) SIGKILL    10) SIGUSR1
11) SIGSEGV    12) SIGUSR2    13) SIGPIPE    14) SIGALRM    15) SIGTERM
16) SIGSTKFLT  17) SIGCHLD    18) SIGCONT    19) SIGSTOP    20) SIGTSTP
...output omitted...
[user@host ~]$ ps aux | grep job
5194  0.0  0.1 222448  2980 pts/1    S    16:39   0:00 /bin/bash /home/user/bin/
control job1
5199  0.0  0.1 222448  3132 pts/1    S    16:39   0:00 /bin/bash /home/user/bin/
control job2
5205  0.0  0.1 222448  3124 pts/1    S    16:39   0:00 /bin/bash /home/user/bin/
control job3
5430  0.0  0.0 221860  1096 pts/1    S+   16:41   0:00 grep --color=auto job
```

``` bash
[user@host ~]$ kill 5194
[user@host ~]$ ps aux | grep job
user   5199  0.0  0.1 222448  3132 pts/1    S    16:39   0:00 /bin/bash /home/
user/bin/control job2
user   5205  0.0  0.1 222448  3124 pts/1    S    16:39   0:00 /bin/bash /home/
user/bin/control job3
user   5783  0.0  0.0 221860   964 pts/1    S+   16:43   0:00 grep --color=auto
 job
[1]   Terminated              control job1
[user@host ~]$ kill -9 5199
[user@host ~]$ ps aux | grep job
user   5205  0.0  0.1 222448  3124 pts/1    S    16:39   0:00 /bin/bash /home/
user/bin/control job3
user   5930  0.0  0.0 221860  1048 pts/1    S+   16:44   0:00 grep --color=auto
 job
[2]-  Killed                  control job2
[user@host ~]$ kill -SIGTERM 5205
user   5986  0.0  0.0 221860  1048 pts/1    S+   16:45   0:00 grep --color=auto
 job
[3]+  Terminated              control job3
```

The **killall** command can signal multiple processes, based on their command name.

``` bash
[user@host ~]$ ps aux | grep job
5194  0.0  0.1 222448  2980 pts/1    S    16:39   0:00 /bin/bash /home/user/bin/
control job1
5199  0.0  0.1 222448  3132 pts/1    S    16:39   0:00 /bin/bash /home/user/bin/
control job2
5205  0.0  0.1 222448  3124 pts/1    S    16:39   0:00 /bin/bash /home/user/bin/
control job3
5430  0.0  0.0 221860  1096 pts/1    S+   16:41   0:00 grep --color=auto job
[user@host ~]$ killall control
[1]   Terminated              control job1
[2]-  Terminated              control job2
[3]+  Terminated              control job3
[user@host ~]$ 
```

Use **pkill** to send a signal to one or more processes which match selection criteria. Selection
criteria can be a command name, a processes owned by a specific user, or all system-wide
processes. The **pkill** command includes advanced selection criteria:

- Command - Processes with a pattern-matched command name.

- UID - Processes owned by a Linux user account, effective or real.

- GID - Processes owned by a Linux group account, effective or real.

- Parent - Child processes of a specific parent process.

- Terminal - Processes running on a specific controlling terminal.

``` bash
[user@host ~]$ ps aux | grep pkill
user   5992  0.0  0.1 222448  3040 pts/1    S    16:59   0:00 /bin/bash /home/
user/bin/control pkill1
user   5996  0.0  0.1 222448  3048 pts/1    S    16:59   0:00 /bin/bash /home/
user/bin/control pkill2
user   6004  0.0  0.1 222448  3048 pts/1    S    16:59   0:00 /bin/bash /home/
user/bin/control pkill3
```

``` bash
[user@host ~]$ pkill control
[1]   Terminated              control pkill1
[2]-  Terminated              control pkill2
[user@host ~]$ ps aux | grep pkill
user   6219  0.0  0.0 221860  1052 pts/1    S+   17:00   0:00 grep --color=auto
 pkill
[3]+  Terminated              control pkill3
[user@host ~]$ ps aux | grep test
user   6281  0.0  0.1 222448  3012 pts/1    S    17:04   0:00 /bin/bash /home/
user/bin/control test1
user   6285  0.0  0.1 222448  3128 pts/1    S    17:04   0:00 /bin/bash /home/
user/bin/control test2
user   6292  0.0  0.1 222448  3064 pts/1    S    17:04   0:00 /bin/bash /home/
user/bin/control test3
user   6318  0.0  0.0 221860  1080 pts/1    S+   17:04   0:00 grep --color=auto
 test 
[user@host ~]$ pkill -U user
[user@host ~]$ ps aux | grep test
user   6870  0.0  0.0 221860  1048 pts/0    S+   17:07   0:00 grep --color=auto
 test
[user@host ~]$ 
```

### LOGGING USERS OUT ADMINISTRATIVELY ###

You may need to log other users off for any of a variety of reasons. To name a few of the many
possibilities: the user committed a security violation; the user may have overused resources; the
user may have an unresponsive system; or the user has improper access to materials. In these
cases, you may need to administratively terminate their session using signals.

To log off a user, first identify the login session to be terminated. Use the w command to list user
logins and current running processes. Note the **TTY** and FROM columns to determine the sessions
to close.

All user login sessions are associated with a terminal device (**TTY**). If the device name is of the
form **pts/N**, it is a pseudo-terminal associated with a graphical terminal window or remote login
session. If it is of the form **ttyN**, the user is on a system console, alternate console, or other
directly connected terminal device.

``` bash
[user@host ~]$ w
 12:43:06 up 27 min,  5 users,  load average: 0.03, 0.17, 0.66
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty2                      12:26   14:58   0.04s  0.04s -bash
bob      tty3                      12:28   14:42   0.02s  0.02s -bash
user     pts/1    desk.example.com 12:41    2.00s  0.03s  0.03s w
[user@host ~]$ 
```

Discover how long a user has been on the system by viewing the session login time. For each
session, CPU resources consumed by current jobs, including background tasks and child processes,
are in the **JCPU** column. Current foreground process CPU consumption is in the PCPU column.

Processes and sessions can be individually or collectively signaled. To terminate all processes for
one user, use the **pkill** command. Because the initial process in a login session (session leader) is
designed to handle session termination requests and ignore unintended keyboard signals, killing
all of a user's processes and login shells requires using the SIGKILL signal.

!!! danger "IMPORTANT"

	SIGKILL is commonly used too quickly by administrators.
	
	Since the SIGKILL signal cannot be handled or ignored, it is always fatal. However, it
	forces termination without allowing the killed process to run self-cleanup routines.
	It is recommended to send SIGTERM first, then try SIGINT, and only if both fail retry
	with SIGKILL.
	
First identify the PID numbers to be killed using **pgrep**, which operates much like **pkill**, including
using the same options, except that **pgrep** lists processes rather than killing them.

``` bash
[root@host ~]# pgrep -l -u bob
6964 bash
6998 sleep
6999 sleep
7000 sleep
[root@host ~]# pkill -SIGKILL -u bob
[root@host ~]# pgrep -l -u bob
[root@host ~]# 
```

When processes requiring attention are in the same login session, it may not be necessary to kill
all of a user's processes. Determine the controlling terminal for the session using the **w** command,
then kill only processes referencing the same terminal ID. Unless **SIGKILL** is specified, the session
leader (here, the Bash login shell) successfully handles and survives the termination request, but
all other session processes are terminated.

``` bash
[root@host ~]# pgrep -l -u bob
7391 bash
7426 sleep
7427 sleep
7428 sleep
[root@host ~]# w -h -u bob
bob      tty3      18:37    5:04   0.03s  0.03s -bash
[root@host ~]# pkill -t tty3
[root@host ~]# pgrep -l -u bob
7391 bash
[root@host ~]# pkill -SIGKILL -t tty3
[root@host ~]# pgrep -l -u bob
[root@host ~]# 
```

The same selective process termination can be applied using parent and child process
relationships. Use the **pstree** command to view a process tree for the system or a single user. Use
the parent process's PID to kill all children they have created. This time, the parent Bash login shell
survives because the signal is directed only at its child processes.

``` bash
[root@host ~]# pstree -p bob
bash(8391)─┬─sleep(8425)
           ├─sleep(8426)
           └─sleep(8427)
[root@host ~]# pkill -P 8391
[root@host ~]# pgrep -l -u bob
bash(8391)
[root@host ~]# pkill -SIGKILL -P 8391
[root@host ~]# pgrep -l -u bob
bash(8391)
[root@host ~]# 
```

### MONITORING PROCESS ACTIVITY ###

#### DESCRIBING LOAD AVERAGE ####

*Load average* is a measurement provided by the Linux kernel that is a simple way to represent the
perceived system load over time. It can be used as a rough gauge of how many system resource
requests are pending, and to determine whether system load is increasing or decreasing over time.

Every five seconds, the kernel collects the current load number, based on the number of processes
in runnable and uninterruptible states. This number is accumulated and reported as an exponential
moving average over the most recent 1, 5, and 15 minutes.

**Understanding the Linux Load Average Calculation**

The load average represents the perceived system load over a time period. Linux determines this
by reporting how many processes are ready to run on a CPU, and how many processes are waiting
for disk or network I/O to complete.

- The load number is essentially based on the number of processes that are ready to run (in
process state **R**) and are waiting for I/O to complete (in process state **D**).

- Some UNIX systems only consider CPU utilization or run queue length to indicate system load.
Linux also includes disk or network utilization because that can have as significant an impact
on system performance as CPU load. When experiencing high load averages with minimal CPU
activity, examine disk and network activity.

Load average is a rough measurement of how many processes are currently waiting for a request
to complete before they can do anything else. The request might be for CPU time to run the
process. Alternatively, the request might be for a critical disk I/O operation to complete, and the
process cannot be run on the CPU until the request completes, even if the CPU is idle. Either way,
system load is impacted and the system appears to run more slowly because processes are waiting
to run.

**Interpreting Displayed Load Average Values**

The **uptime** command is one way to display the current load average. It prints the current time,
how long the machine has been up, how many user sessions are running, and the current load
average.

``` bash
[user@host ~]$ uptime
 15:29:03 up 14 min,  2 users,  load average: 2.92, 4.48, 5.20
```

The three values for the load average represent the load over the last 1, 5, and 15 minutes. A quick
glance indicates whether system load appears to be increasing or decreasing.

If the main contribution to load average is from processes waiting for the CPU, you can calculate
the approximate per CPU load value to determine whether the system is experiencing significant
waiting.

The **lscpu** command can help you determine how many CPUs a system has.

In the following example, the system is a dual-core single socket system with two hyperthreads per
core. Roughly speaking, Linux will treat this as a four CPU system for scheduling purposes.

``` bash
[user@host ~]$ lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              4
On-line CPU(s) list: 0-3
Thread(s) per core:  2
Core(s) per socket:  2
Socket(s):           1
NUMA node(s):        1
...output omitted...
```


For a moment, imagine that the only contribution to the load number is from processes that need
CPU time. Then you can divide the displayed load average values by the number of logical CPUs
in the system. A value below 1 indicates satisfactory resource utilization and minimal wait times. A
value above 1 indicates resource saturation and some amount of processing delay.

```
# From lscpu, the system has four logical CPUs, so divide by 4:
#                               load average: 2.92, 4.48, 5.20
#           divide by number of logical CPUs:    4     4     4
#                                             ----  ----  ----
#                       per-CPU load average: 0.73  1.12  1.30
#
# This system's load average appears to be decreasing.
# With a load average of 2.92 on four CPUs, all CPUs were in use ~73% of the time.
# During the last 5 minutes, the system was overloaded by ~12%.
# During the last 15 minutes, the system was overloaded by ~30%.
```

An idle CPU queue has a load number of 0. Each process waiting for a CPU adds a count of 1 to the
load number. If one process is running on a CPU, the load number is one, the resource (the CPU) is
in use, but there are no requests waiting. If that process is running for a full minute, its contribution
to the one-minute load average will be 1.

However, processes uninterruptibly sleeping for critical I/O due to a busy disk or network resource
are also included in the count and increase the load average. While not an indication of CPU
utilization, these processes are added to the queue count because they are waiting for resources
and cannot run on a CPU until they get them. This is still system load due to resource limitations
that is causing processes not to run.

Until resource saturation, a load average remains below 1, since tasks are seldom found waiting in
queue. Load average only increases when resource saturation causes requests to remain queued
and are counted by the load calculation routine. When resource utilization approaches 100%, each
additional request starts experiencing service wait time.

A number of additional tools report load average, including **w** and **top**.

#### REAL-TIME PROCESS MONITORING ####

The top program is a dynamic view of the system's processes, displaying a summary header
followed by a process or thread list similar to ps information. Unlike the static ps output, top
continuously refreshes at a configurable interval, and provides capabilities for column reordering,
sorting, and highlighting. User configurations can be saved and made persistent.

Default output columns are recognizable from other resource tools:

- The process ID (**PID**).

- User name (**USER**) is the process owner.

- Virtual memory (**VIRT**) is all memory the process is using, including the resident set, shared
libraries, and any mapped or swapped memory pages. (Labeled VSZ in the ps command.)

- Resident memory (**RES**) is the physical memory used by the process, including any resident
shared objects. (Labeled **RSS** in the **ps** command.)

- Process state (**S**) displays as:
	
	- D = Uninterruptible Sleeping

	- R = Running or Runnable
   
	- S = Sleeping
   
	- T = Stopped or Traced
	
	- Z = Zombie

- CPU time (**TIME**) is the total processing time since the process started. May be toggled to
include cumulative time of all previous children.

- The process command name (**COMMAND**).

**Fundamental Keystrokes in top**

| KEY            | PURPOSE                                                                         |
|:--------------:|:--------------------------------------------------------------------------------|
| **? or H**     | Help for interactive keystrokes.                                                |
| **L, T, M**    | Toggles for load, threads, and memory header lines.                             |
| **1**          | Toggle showing individual CPUs or a summary for all CPUs in header.             |
| **S(1)**       | Change the refresh (screen) rate, in decimal seconds (e.g., 0.5, 1, 5).         |
| **B**          | Toggle reverse highlighting for Running processes; default is bold only.        |
| **B**          | Enables use of bold in display, in the header, and for Running processes.       |
| **Shift+H**    | Toggle threads; show process summary or individual threads.                     |
| **U, Shift+U** | Filter for any user name (effective, real).                                     |
| **Shift+M**    | Sorts process listing by memory usage, in descending order.                     |
| **Shift+P**    | Sorts process listing by processor utilization, in descending order.            |
| **K(1)**       | Kill a process. When prompted, enter PID, then signal.                          |
| **R(1)**       | Renice a process. When prompted, enter PID, then nice_value.                    |
| **Shift+W**    | Write (save) the current display configuration for use at the next top restart. |
| **Q**          | Quit.                                                                           |

### SUMMARY ###

In this chapter, you learned:

- A process is a running instance of an executable program. Processes are assigned a state, which
can be running, sleeping, stopped, or zombie. The **ps** command is used to list processes.

- Each terminal is its own session and can have foreground process and independent background
processes. The **jobs** command displays processes within a terminal session.

- A signal is a software interrupt that reports events to an executing program. The **kill, pkill**,
and **killall** commands use signals to control processes.

- Load average is an estimate of how busy the system is. To display load average values, you can
use the **top, uptime**, or **w** command.

### LAB

pdf - 297