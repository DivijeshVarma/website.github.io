---
title: "Scheduling Future Tasks"
---

### SCHEDULING A DEFERRED USER JOB

**DESCRIBING DEFERRED USER TASKS**

Sometimes you might need to run a command, or set of commands, at a set point in the future.
Examples include people who want to schedule an email to their boss, or a system administrator
working on a firewall configuration who puts a “safety” job in place to reset the firewall settings in
ten minutes' time, unless they deactivate the job beforehand.

These scheduled commands are often called tasks or jobs, and the term deferred indicates that
these tasks or jobs are going to run in the future.

One of the solutions available to Red Hat Enterprise Linux users for scheduling deferred tasks is
at. The at package provides the (atd) system daemon along with a set of command-line tools to
interact with the daemon (**at, atq**, and more). In a default Red Hat Enterprise Linux installation,
the atd daemon is installed and enabled automatically.

Users (including root) can queue up jobs for the atd daemon using the **at** command. The atd
daemon provides 26 queues, **a** to **z**, with jobs in alphabetically later queues getting lower system
priority (higher nice values, discussed in a later chapter).

**Scheduling Deferred User Tasks**

Use the **at TIMESPEC** command to schedule a new job. The **at** command then reads the
commands to execute from the stdin channel. While manually entering commands, you can finish
your input by pressing **Ctrl+D**. For more complex commands that are prone to typographical
errors, it is often easier to use input redirection from a script file, for example, **at now +5min <
myscript**, rather than typing all the commands manually in a terminal window.

The **TIMESPEC** argument with the **at** command accepts many powerful combinations, allowing
users to describe exactly when a job should run. Typically, they start with a time, for example,
**02:00pm, 15:59**, or even **teatime**, followed by an optional date or number of days in the future.
The following lists some examples of combinations that can be used.

- now +5min

- teatime tomorrow (teatime is 16:00)

- noon +4 days

- 5pm august 3 2021

For a complete list of valid time specifications, refer to the **timespec** definition as listed in the
references.

### INSPECTING AND MANAGING DEFERRED USER JOBS

To get an overview of the pending jobs for the current user, use the command **atq** or the **at -l** commands.

```bash
[user@host ~]$ atq
28   Mon Feb 2 05:13:00 2015   a  user
29   Mon Feb 3 16:00:00 2014   h  user
27   Tue Feb 4 12:00:00 2014   a  user

```

In the preceding output, every line represents a different job scheduled to run in the future.

28    The unique job number for this job.

Mon   The execution date and time for the scheduled job.

a     Indicates that the job is scheduled with the default queue a. Different jobs may be scheduled with different queues.

user  The owner of the job (and the user that the job will run as).

!!! danger "IMPORTANT"

    Unprivileged users can only see and control their own jobs. The root user can see
    and manage all jobs.

To inspect the actual commands that will run when a job is executed, use the **at -c JOBNUMBER**
command. This command shows the environment for the job being set up to reflect the
environment of the user who created the job at the time it was created, followed by the actual
commands to be run.

**Removing Jobs**

The **atrm JOBNUMBER** command removes a scheduled job. Remove the scheduled job when it
is no longer needed, for example, when a remote firewall configuration succeeded, and does not
need to be reset.

### SCHEDULING RECURRING USER JOBS

#### DESCRIBING RECURRING USER JOBS

Jobs scheduled to run repeatedly are called recurring jobs. Red Hat Enterprise Linux systems
ship with the crond daemon, provided by the cronie package, enabled and started by default
specifically for recurring jobs. The crond daemon reads multiple configuration files: one per
user (edited with the **crontab** command), and a set of system-wide files. These configuration
files give users and administrators fine-grained control over when their recurring jobs should be
executed.

If a scheduled command produces any output or error that is not redirected, the crond daemon
attempts to email that output or error to the user who owns that job (unless overridden) using the
mail server configured on the system. Depending on the environment, this may need additional
configuration. The output or error of the scheduled command can be redirected to different files.

#### SCHEDULING RECURRING USER JOBS

Normal users can use the **crontab** command to manage their jobs. This command can be called
in four different ways:

**Crontab Examples**

COMMAND | INTENDED USE |
:---------|----------|
 crontab -l | List the jobs for the current user. |
 crontab -r | Remove all jobs for the current user. |
 crontab -e | Edit jobs for the current user. |
 crontab filename | Remove all jobs, and replace with the jobs read from filename. If no file is specified, stdin is used.  |

!!! info "NOTE"

    The superuser can use the -u option with the crontab command to manage jobs
    for another user. You should not use the crontab command to manage system
    jobs; instead, use the methods described in the next section.

### DESCRIBING USER JOB FORMAT

The **crontab -e** command invokes Vim by default, unless the EDITOR environment variable has
been set to something different. Enter one job per line. Other valid entries include: empty lines,
typically for ease of reading; comments, identified by lines starting with the number sign (#); and
environment variables using the format **NAME=value**, which affects all lines below the line where
they are declared. Common variable settings include the SHELL variable, which declares which
shell to use to interpret the remaining lines of the crontab file; and the MAILTO variable, which
determines who should receive any emailed output.

!!! danger "IMPORTANT"

    Sending email may require additional configuration of the local mail server or SMTP
    relay on a system.

Fields in the **crontab** file appear in the following order:

- Minutes

- Hours

- Day of month

- Month

- Day of week

- Command

!!! danger "IMPORTANT"

    When the **Day of month** and **Day of week** fields are both other than *, the
    command is executed when either of these two fields are satisfied. For example,
    to run a command on the 15th of every month, and every Friday at **12:15**, use the
    following job format:

    `15 12 15 * Fri command`

The first five fields all use the same syntax rules:

- '*' for “Do not Care”/always.

- A number to specify a number of minutes or hours, a date, or a weekday. For weekdays, 0 equals
Sunday, 1 equals Monday, 2 equals Tuesday, and so on. 7 also equals Sunday.

- **x-y** for a range, x to y inclusive.

- **x,y** for lists. Lists can include ranges as well, for example, **5,10-13,17** in the **Minutes** column
to indicate that a job should run at 5, 10, 11, 12, 13, and 17 minutes past the hour.

- `*/x` to indicate an interval of x, for example, `*/7` in the **Minutes** column runs a job every seven
minutes.

Additionally, 3-letter English abbreviations can be used for both months and weekdays, for
example, Jan, Feb, and Mon, Tue.

The last field contains the command to execute using the default shell. The **SHELL** environment
variable can used to change the shell for the scheduled command. If the command contains an
unescaped percentage sign (%), then that percentage sign is treated as a newline character, and
everything after the percentage sign is passed to the command on **stdin**.

**Example Recurring User Jobs**

This section describes some examples of recurring jobs.

- The following job executes the command **/usr/local/bin/yearly_backup** at exactly
9 a.m. on February 2nd, every year.

`0 9 2 2 * /usr/local/bin/yearly_backup`

- The following job sends an email containing the word **Chime** to the owner of this job, every five
minutes between 9 a.m. and 5 p.m., on every Friday in July.

`*/5 9-16 * Jul 5 echo "Chime"`

The preceding **9-16** range of hours means that the job timer starts at the ninth hour (09:00)
and continues until the end of the sixteenth hour (16:59). The job starts executing at **09:00** with
the last execution at **16:55** because five minutes from **16:55** is **17:00** which is beyond the
given scope of hours.

- The following job runs the command **/usr/local/bin/daily_report** every weekday at two
minutes before midnight.

`58 23 * * 1-5 /usr/local/bin/daily_report`

- The following job executes the **mutt** command to send the mail message **Checking in** to the
recipient **boss@example.com** on every workday (Monday to Friday), at 9 a.m.

`0 9 * * 1-5 mutt -s "Checking in" boss@example.com % Hi there boss, just checking in.`

### SCHEDULING RECURRING SYSTEM JOBS

#### DESCRIBING RECURRING SYSTEM JOBS

System administrators often need to run recurring jobs. Best practice is to run these jobs from
system accounts rather than from user accounts. That is, do not schedule to run these jobs using
the **crontab** command, but instead use system-wide crontab files. Job entries in the system-wide
crontab files are similar to those of the users' crontab entries, excepting only that the system-wide
crontab files have an extra field before the command field; the user under whose authority the
command should run.

The **/etc/crontab** file has a useful syntax diagram in the included comments.

```
# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# | .------------- hour (0 - 23)
# | | .---------- day of month (1 - 31)
# | | | .------- month (1 - 12) OR jan,feb,mar,apr ...
# | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue ...
# | | | | |
# * * * * * user-name command to be executed

```

Recurring system jobs are defined in two locations: the **/etc/crontab** file, and files within the
**/etc/cron.d/** directory. You should always create your custom crontab files under the **/etc/
cron.d** directory to schedule recurring system jobs. Place the custom crontab file in **/etc/
cron.d** to protect it from being overwritten if any package update occurs to the provider of **/
etc/crontab**, which may overwrite the existing contents in **/etc/crontab**. Packages that
require recurring system jobs place their crontab files in **/etc/cron.d/** containing the job
entries. Administrators also use this location to group related jobs into a single file.

The crontab system also includes repositories for scripts that need to run every hour, day,
week, and month. These repositories are directories called **/etc/cron.hourly/, /etc/
cron.daily/, /etc/cron.weekly/**, and **/etc/cron.monthly/**. Again, these directories
contain executable shell scripts, not crontab files.

!!! danger "IMPORTANT"

    Remember to make any script you place in these directories executable. If a script
    is not executable, it will not run. To make a script executable, use the **chmod +x
    script_name**.

A command called **run-parts** called from the **/etc/cron.d/0hourly** file runs the **/etc/
cron.hourly/*** scripts. The **run-parts** command also runs the daily, weekly, and monthly jobs,
but it is called from a different configuration file called **/etc/anacrontab**.

!!! info "NOTE"

    In the past, a separate service called anacron used to handle the /etc/
    anacrontab file, but in Red Hat Enterprise Linux 7 and later, the regular crond
    service parses this file.

The purpose of **/etc/anacrontab** is to make sure that important jobs always run, and not
skipped accidentally because the system was turned off or hibernating when the job should have
been executed. For example, if a system job that runs daily was not executed last time it was due
because the system was rebooting, the job is executed when the system becomes ready. However,
there may be a delay of several minutes in starting the job depending on the value of the **Delay
in minutes** parameter specified for the job in **/etc/anacrontab**.

There are different files in **/var/spool/anacron/** for each of the daily, weekly, and monthly
jobs to determine if a particular job has run. When crond starts a job from **/etc/anacrontab**, it
updates the time stamps of those files. The same time stamp is used to determine when a job was
last run. The syntax of **/etc/anacrontab** is different from the regular **crontab** configuration
files. It contains exactly four fields per line, as follows.

- **Period in days**

The interval in days for the job that runs on a repeating schedule. This field accepts an integer
or a macro as its value. For example, the macro @**daily** is equivalent to the integer **1**, which
means that the job is executed on a daily basis. Similarly, the macro @**weekly** is equivalent to
the integer **7**, which means that the job is executed on a weekly basis.

- Delay in minutes

The amount of time the crond daemon should wait before starting this job.

- Job identifier

The unique name the job is identified as in the log messages.

- Command

The command to be executed.

The **/etc/anacrontab** file also contains environment variable declarations using the syntax
**NAME=value**. Of special interest is the variable START_HOURS_RANGE, which specifies the time
interval for the jobs to run. Jobs are not started outside of this range. If on a particular day, a job
does not run within this time interval, the job has to wait until the next day for execution.

### INTRODUCING SYSTEMD TIMER

With the advent of systemd in Red Hat Enterprise Linux 7, a new scheduling function is now
available: systemd timer units. A systemd timer unit activates another unit of a different type
(such as a service) whose unit name matches the timer unit name. The timer unit allows timer-
based activation of other units. For easier debugging, systemd logs timer events in system
journals.

**Sample Timer Unit**

The sysstat package provides a systemd timer unit called sysstat-collect.timer to collect
system statistics every 10 minutes. The following output shows the configuration lines of **/usr/
lib/systemd/system/sysstat-collect.timer**.

```
...output omitted...
[Unit]
Description=Run system activity accounting tool every 10 minutes
[Timer]
OnCalendar=*:00/10
[Install]
WantedBy=sysstat.service

```

The parameter **OnCalendar=*:00/10** signifies that this timer unit activates the corresponding
unit (sysstat-collect.service) every 10 minutes. However, you can specify more complex
time intervals. For example, a value of **2019-03-* 12:35,37,39:16** against the **OnCalendar**
parameter causes the timer unit to activate the corresponding service unit at **12:35:16,
12:37:16**, and **12:39:16** every day throughout the entire month of March, 2019. You can
also specify relative timers using parameters such as **OnUnitActiveSec**. For example, the
**OnUnitActiveSec=15min** option causes the timer unit to trigger the corresponding unit 15
minutes after the last time the timer unit activated its corresponding unit.

!!! danger "IMPORTANT"

    Do not modify any unit configuration file under the **/usr/lib/systemd/system**
    directory because any update to the provider package of the configuration file
    may override the configuration changes you made in that file. So, make a copy
    of the unit configuration file you intend to change under the **/etc/systemd/**
    **system** directory and then modify the copy so that the configuration changes you
    make with respect to a unit does not get overridden by any update to the provider
    package. If two files exist with the same name under the **/usr/lib/systemd/**
    **system** and **/etc/systemd/system** directories, systemd parses the file under
    the **/etc/systemd/system** directory.

After you change the timer unit configuration file, use the **systemctl daemon-reload**
command to ensure that systemd is aware of the changes. This command reloads the systemd
manager configuration.

```bash
[root@host ~]# systemctl daemon-reload

```
After you reload the systemd manager configuration, use the following **systemctl** command to
activate the timer unit.

```bash
[root@host ~]# systemctl enable --now <unitname>.timer
```
### MANAGING TEMPORARY FILES

#### MANAGING TEMPORARY FILES

A modern system requires a large number of temporary files and directories. Some applications
(and users) use the **/tmp** directory to hold temporary data, while others use a more task-specific
location such as daemon and user-specific volatile directories under /run. In this context, volatile
means that the file system storing these files only exists in memory. When the system reboots or
loses power, all the contents of volatile storage will be gone.

To keep a system running cleanly, it is necessary for these directories and files to be created when
they do not exist, because daemons and scripts might rely on them being there, and for old files to
be purged so that they do not fill up disk space or provide faulty information.

Red Hat Enterprise Linux 7 and later include a new tool called **systemd-tmpfiles**, which
provides a structured and configurable method to manage temporary directories and files.

When systemd starts a system, one of the first service units launched is systemd-tmpfiles-
setup. This service runs the command **systemd-tmpfiles --create --remove**. This
command reads configuration files from **/usr/lib/tmpfiles.d/*.conf, /run/tmpfiles.d/
*.conf, and /etc/tmpfiles.d/*.conf**. Any files and directories marked for deletion in those
configuration files is removed, and any files and directories marked for creation (or permission
fixes) will be created with the correct permissions if necessary.

**Cleaning Temporary Files with a Systemd Timer**

To ensure that long-running systems do not fill up their disks with stale data, a systemd timer unit
called systemd-tmpfiles-clean.timer triggers systemd-tmpfiles-clean.service on
a regular interval, which executes the **systemd-tmpfiles --clean** command.

The **systemd** timer unit configuration files have a **[Timer]** section that indicates how often the
service with the same name should be started.

Use the following **systemctl** command to view the contents of the **systemd-tmpfiles-
clean.timer** unit configuration file.

```bash
[user@host ~]$ systemctl cat systemd-tmpfiles-clean.timer
# /usr/lib/systemd/system/systemd-tmpfiles-clean.timer
# SPDX-License-Identifier: LGPL-2.1+
#
# This file is part of systemd.
#
# systemd is free software; you can redistribute it and/or modify it
# under the terms of the GNU Lesser General Public License as published
# by
# the Free Software Foundation; either version 2.1 of the License, or
# (at your option) any later version.

[Unit]
Description=Daily Cleanup of Temporary Directories
Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)
[Timer]
OnBootSec=15min
```
In the preceding configuration the parameter **OnBootSec=15min** indicates that the service
unit called systemd-tmpfiles-clean.service gets triggered 15 minutes after the system
has booted up. The parameter **OnUnitActiveSec=1d** indicates that any further trigger to the
systemd-tmpfiles-clean.service service unit happens 24 hours after the service unit was
activated last.

Based on your requirement, you can change the parameters in the **systemd-tmpfiles-
clean.timer** timer unit configuration file. For example, the value **30min** for the parameter
**OnUnitActiveSec** triggers the systemd-tmpfiles-clean.service service unit 30 minutes
after the service unit was last activated. As a result, systemd-tmpfiles-clean.service gets
triggered every 30 minutes after bringing the changes into effect.

After changing the timer unit configuration file, use the **systemctl daemon-reload** command
to ensure that systemd is aware of the change. This command reloads the systemd manager
configuration.

```bash
[root@host ~]# systemctl daemon-reload

```
After you reload the systemd manager configuration, use the following **systemctl** command to
activate the systemd-tmpfiles-clean.timer unit.

```bash
[root@host ~]# systemctl enable --now systemd-tmpfiles-clean.timer

```
**Cleaning Temporary Files Manually**

The command **systemd-tmpfiles --clean** parses the same configuration files as the
**systemd-tmpfiles --create** command, but instead of creating files and directories, it
will purge all files which have not been accessed, changed, or modified more recently than the
maximum age defined in the configuration file.

The format of the configuration files for **systemd-tmpfiles** is detailed in the **tmpfiles.d**(5)
manual page. The basic syntax consists of seven columns: Type, Path, Mode, UID, GID, Age, and
Argument. Type refers to the action that **systemd-tmpfiles** should take; for example, d to
create a directory if it does not yet exist, or Z to recursively restore SELinux contexts and file
permissions and ownership.

The following are some examples with explanations.

`d /run/systemd/seats 0755 root root -`

When creating files and directories, create the **/run/systemd/seats** directory if it does not
yet exist, owned by the user root and the group root, with permissions set to **rwxr-xr-x**. This
directory will not be automatically purged.

`D /home/student 0700 student student 1d`

Create the **/home/student** directory if it does not yet exist. If it does exist, empty it of all
contents. When **systemd-tmpfiles --clean** is run, remove all files which have not been
accessed, changed, or modified in more than one day.

`L /run/fstablink - root root - /etc/fstab`

Create the symbolic link **/run/fstablink** pointing to **/etc/fstab**. Never automatically purge
this line.

**Configuration File Precedence**

Configuration files can exist in three places:

- /etc/tmpfiles.d/*.conf

- /run/tmpfiles.d/*.conf

- /usr/lib/tmpfiles.d/*.conf

The files in **/usr/lib/tmpfiles.d/** are provided by the relevant RPM packages, and you
should not edit these files. The files under **/run/tmpfiles.d/** are themselves volatile files,
normally used by daemons to manage their own runtime temporary files. The files under **/etc/
tmpfiles.d/** are meant for administrators to configure custom temporary locations, and to
override vendor-provided defaults.

If a file in **/run/tmpfiles.d/** has the same file name as a file in **/usr/lib/tmpfiles.d/**,
then the file in **/run/tmpfiles.d/** is used. If a file in **/etc/tmpfiles.d/** has the same file
name as a file in either **/run/tmpfiles.d/** or **/usr/lib/tmpfiles.d/**, then the file in **/etc/
tmpfiles.d/** is used.

Given these precedence rules, you can easily override vendor-provided settings by copying the
relevant file to **/etc/tmpfiles.d/**, and then editing it. Working in this fashion ensures that
administrator-provided settings can be easily managed from a central configuration management
system, and not be overwritten by an update to a package.

!!! info "NOTE"

    When testing new or modified configurations, it can be useful to only apply the
    commands from one configuration file. This can be achieved by specifying the name
    of the configuration file on the command line.

### SUMMARY

In this chapter, you learned:

- Jobs that are scheduled to run once in the future are called deferred jobs or tasks.

- Recurring user jobs execute the user's tasks on a repeating schedule.

- Recurring system jobs accomplish administrative tasks on a repeating schedule that have
system-wide impact.

- The systemd timer units can execute both the deferred or recurring jobs.

### LAB

pdf - 66