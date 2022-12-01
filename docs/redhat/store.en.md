---
title: "Analyzing And Storing Logs"
---

### DESCRIBING SYSTEM LOG ARCHITECTURE ###

#### SYSTEM LOGGING ####

Processes and the operating system kernel record a log of events that happen. These logs are used
to audit the system and troubleshoot problems.

Many systems record logs of events in text files which are kept in the **/var/log** directory. These
logs can be inspected using normal text utilities such as **less** and **tail**.

A standard logging system based on the Syslog protocol is built into Red Hat Enterprise Linux.
Many programs use this system to record events and organize them into log files. The systemd-
journald and rsyslog services handle the syslog messages in Red Hat Enterprise Linux 8.

The *systemd-journald* service is at the heart of the operating system event logging
architecture. It collects event messages from many sources including the kernel, output from the
early stages of the boot process, standard output and standard error from daemons as they start
up and run, and syslog events. It then restructures them into a standard format, and writes them
into a structured, indexed system journal. By default, this journal is stored on a file system that
does not persist across reboots.

However, the rsyslog service reads syslog messages received by systemd-journald from
the journal as they arrive. It then processes the syslog events, recording them to its log files or
forwarding them to other services according to its own configuration.

The rsyslog service sorts and writes syslog messages to the log files that do persist across
reboots in **/var/log**. The rsyslog service sorts the log messages to specific log files based on
the type of program that sent each message, or facility, and the priority of each syslog message.

In addition to syslog message files, the **/var/log** directory contains log files from other services
on the system. The following table lists some useful files in the **/var/log** directory.

**Selected System Log Files**

| LOG FILE              | TYPE OF MESSAGES STORED                                                                                                                                                                 |
|:----------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **/var/log/messages** | Most syslog messages are logged here. Exceptions include messages related to authentication and email processing,scheduled job execution, and those which are purely debugging-related. |
| **/var/log/secure**   | Syslog messages related to security and authentication events.                                                                                                                          |
| **/var/log/maillog**  | Syslog messages related to the mail server.                                                                                                                                             |
| **/var/log/cron**     | Syslog messages related to scheduled job execution.                                                                                                                                     |
| **/var/log/boot.log** | Non-syslog console messages related to system startup.                                                                                                                                  |

!!! info "NOTE"

	Some applications do not use syslog to manage their log messages, although
	typically, they do place their log files in a subdirectory of /var/log. For example, the
	Apache Web Server saves log messages to files in a subddirectory of the **/var/log**
	directory.

### REVIEWING SYSLOG FILES ###

#### LOGGING EVENTS TO THE SYSTEM ####

Many programs use the syslog protocol to log events to the system. Each log message is
categorized by a facility (the type of message) and a priority (the severity of the message).
Available facilities are documented in the rsyslog.conf(5) man page.

The following table lists the standard eight syslog priorities from highest to lowest.

**Overview of Syslog Priorities**

| CODE | PRIORITY | SEVERITY                         |
|:----:|:---------|----------------------------------|
| 0    | emerg    | System is unusable               |
| 1    | alert    | Action must be taken immediately |
| 2    | crit     | Critical condition               |
| 3    | err      | Non-critical error condition     |
| 4    | warning  | Warning condition                |
| 5    | notice   | Normal but significant event     |
| 6    | info     | Informational event              |
| 7    | debug    | Debugging-level message          |

The rsyslog service uses the facility and priority of log messages to determine how to handle
them. This is configured by rules in the **/etc/rsyslog.conf** file and any file in the **/etc/
rsyslog.d** directory that has a file name extension of .conf. Software packages can easily add
rules by installing an appropriate file in the **/etc/rsyslog.d** directory.

Each rule that controls how to sort syslog messages is a line in one of the configuration files. The
left side of each line indicates the facility and severity of the syslog messages the rule matches.
The right side of each line indicates what file to save the log message in (or where else to deliver
the message). An asterisk (*****) is a wildcard that matches all values.

For example, the following line would record messages sent to the authpriv facility at any priority
to the file /var/log/secure:

`authpriv.*                  /var/log/secure`

Log messages sometimes match more than one rule in **rsyslog.conf**. In such cases, one
message is stored in more than one log file. To limit messages stored, the key word none in the
priority field indicates that no messages for the indicated facility should be stored in the given file.

Instead of logging syslog messages to a file, they can also be printed to the terminals of all logged-
in users. The **rsyslog.conf** file has a setting to print all the syslog messages with the **emerg**
priority to the terminals of all logged-in users.

#### SAMPLE RULES OF RSYSLOG ####

```
#### RULES ####
# Log all kernel messages to the console.
# Logging much else clutters up the screen.
#kern.*                                                 /dev/console
# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure
# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog
# Log cron stuff
cron.*                                                  /var/log/cron
# Everybody gets emergency messages
*.emerg                                                 :omusrmsg:*
# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler
# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log
```

!!! info "NOTE"

	The syslog subsystem has many more features beyond the scope of this course.
	For those who wish to explore further, consult the rsyslog.conf(5) man page
	and the extensive HTML documentation in **/usr/share/doc/rsyslog/html/
	index.html** contained in the rsyslog-doc package, available from the AppStream
	repository in Red Hat Enterprise Linux 8.

### LOG FILE ROTATION ###

The **logrotate** tool rotates log files to keep them from taking up too much space in the file
system containing the **/var/log directory**. When a log file is rotated, it is renamed with an
extension indicating the date it was rotated. For example, the old **/var/log/messages** file may
become **/var/log/messages-20190130** if it is rotated on 2019-01-30. Once the old log file is
rotated, a new log file is created and the service that writes to it is notified.

After a certain number of rotations, typically after four weeks, the oldest log file is discarded to
free disk space. A scheduled job runs the **logrotate** program daily to see if any logs need to be
rotated. Most log files are rotated weekly, but **logrotate** rotates some faster, or slower, or when
they reach a certain size.

Configuration of **logrotate** is not covered in this course. For more information, see the
**logrotate**(8) man page.

#### ANALYZING A SYSLOG ENTRY ####

Log messages start with the oldest message on top and the newest message at the end of the log
file. The rsyslog service uses a standard format while recording entries in log files. The following
example explains the anatomy of a log message in the **/var/log/secure** log file.

Feb 11 20:11:48 localhost sshd[1433]: Failed password for student from
 172.25.0.10 port 59344 ssh2
 
> Feb 11 20:11:48 The time stamp when the log entry was recorded

> localhost The host from which the log message was sent

> sshd[1433]: The program or process name and PID number that sent the log message

>  Failed password The actual message sent

### MONITORING LOGS ###

Monitoring one or more log files for events is helpful to reproduce problems and issues. The **tail
-f /path/to/file** command outputs the last 10 lines of the file specified and continues to
output new lines in the file as they get written.

For example, to monitor for failed login attempts, run the **tail** command in one terminal and
then in another terminal, run the **ssh** command as the root user while a user tries to log in to the
system.

In the first terminal, run the following **tail** command:

``` bash
[root@host ~]# tail -f /var/log/secure
```

In the second terminal, run the following **ssh** command:

``` bash
[root@host ~]# ssh root@localhost
root@localhost's password: redhat
...output omitted...
[root@host ~]# 
```

Return to the first terminal and view the logs.

``` bash
...output omitted...
Feb 10 09:01:13 host sshd[2712]: Accepted password for root from 172.25.254.254
Feb 10 09:01:13 host sshd[2712]: pam_unix(sshd:session): session opened for user
 root by (uid=0)
```

#### SENDING SYSLOG MESSAGES MANUALLY ####

The **logger** command can send messages to the **rsyslog** service. By default, it sends the
message to the user facility with the **notice** priority (user.notice) unless specified otherwise
with the **-p** option. It is useful to test any change to the rsyslog service configuration.

To send a message to the **rsyslog** service that gets recorded in the **/var/log/boot.log** log
file, execute the following **logger** command:

``` bash
[root@host ~]# logger -p local7.notice "Log entry created on host"
```

### REVIEWING SYSTEM JOURNAL ENTRIES ###

#### FINDING EVENTS ####

The systemd-journald service stores logging data in a structured, indexed binary file called the
journal. This data includes extra information about the log event. For example, for syslog events
this includes the facility and the priority of the original message.

!!! danger "IMPORTANT"

	In Red Hat Enterprise Linux 8, the **/run/log** directory stores the system journal by
	default. The contents of the **/run/log** directory get cleared after a reboot. You can
	change this setting, and how to do so is discussed later in this chapter.

To retrieve log messages from the journal, use the **journalctl** command. You can use this
command to view all messages in the journal, or to search for specific events based on a wide
range of options and criteria. If you run the command as root, you have full access to the journal.
Regular users can also use this command, but might be restricted from seeing certain messages.

``` bash
[root@host ~]# journalctl
...output omitted...
Feb 21 17:46:25 host.lab.example.com systemd[24263]: Stopped target Sockets.
Feb 21 17:46:25 host.lab.example.com systemd[24263]: Closed D-Bus User Message Bus
 Socket.
Feb 21 17:46:25 host.lab.example.com systemd[24263]: Closed Multimedia System.
Feb 21 17:46:25 host.lab.example.com systemd[24263]: Reached target Shutdown.
Feb 21 17:46:25 host.lab.example.com systemd[24263]: Starting Exit the Session...
Feb 21 17:46:25 host.lab.example.com systemd[24268]: pam_unix(systemd-
user:session): session c>
Feb 21 17:46:25 host.lab.example.com systemd[1]: Stopped User Manager for UID
 1001.
Feb 21 17:46:25 host.lab.example.com systemd[1]: user-runtime-dir@1001.service:
 Unit not neede>
Feb 21 17:46:25 host.lab.example.com systemd[1]: Stopping /run/user/1001 mount
 wrapper...
Feb 21 17:46:25 host.lab.example.com systemd[1]: Removed slice User Slice of UID
 1001.
Feb 21 17:46:25 host.lab.example.com systemd[1]: Stopped /run/user/1001 mount
 wrapper.
Feb 21 17:46:36 host.lab.example.com sshd[24434]: Accepted publickey for root from
 172.25.250.>
Feb 21 17:46:37 host.lab.example.com systemd[1]: Started Session 20 of user root.
Feb 21 17:46:37 host.lab.example.com systemd-logind[708]: New session 20 of user
 root.
Feb 21 17:46:37 host.lab.example.com sshd[24434]: pam_unix(sshd:session): session
 opened for u>
Feb 21 18:01:01 host.lab.example.com CROND[24468]: (root) CMD (run-parts /etc/
cron.hourly)
Feb 21 18:01:01 host.lab.example.com run-parts[24471]: (/etc/cron.hourly) starting
 0anacron
Feb 21 18:01:01 host.lab.example.com run-parts[24477]: (/etc/cron.hourly) finished
 0anacron
lines 1464-1487/1487 (END) q
```

The **journalctl** command highlights important log messages: messages at **notice** or **warning**
priority are in bold text while messages at the **error** priority or higher are in red text.

The key to successfully using the journal for troubleshooting and auditing is to limit journal
searches to show only relevant output.

By default, **journalctl -n** shows the last 10 log entries. You can adjust this with an optional
argument that specifies how many log entries to display. For the last five log entries, run the
following journalctl command:

``` bash
[root@host ~]# journalctl -n 5
-- Logs begin at Wed 2019-02-20 16:01:17 +07, end at Thu 2019-02-21 18:01:01 +07.
 --
...output omitted...
Feb 21 17:46:37 host.lab.example.com systemd-logind[708]: New session 20 of user
 root.
Feb 21 17:46:37 host.lab.example.com sshd[24434]: pam_unix(sshd:session): session
 opened for u>
Feb 21 18:01:01 host.lab.example.com CROND[24468]: (root) CMD (run-parts /etc/
cron.hourly)
Feb 21 18:01:01 host.lab.example.com run-parts[24471]: (/etc/cron.hourly) starting
 0anacron
Feb 21 18:01:01 host.lab.example.com run-parts[24477]: (/etc/cron.hourly) finished
 0anacron
lines 1-6/6 (END) q
```

Similar to the **tail -f** command, the **journalctl -f** command outputs the last 10 lines of the
system journal and continues to output new journal entries as they get written to the journal. To
exit the **journalctl -f** process, use the **Ctrl+C** key combination.

``` bash
[root@host ~]# journalctl -f
-- Logs begin at Wed 2019-02-20 16:01:17 +07. --
...output omitted...
Feb 21 18:01:01 host.lab.example.com run-parts[24477]: (/etc/cron.hourly) finished
 0anacron
Feb 21 18:22:42 host.lab.example.com sshd[24437]: Received disconnect from
 172.25.250.250 port 48710:11: disconnected by user
Feb 21 18:22:42 host.lab.example.com sshd[24437]: Disconnected from user root
 172.25.250.250 port 48710
Feb 21 18:22:42 host.lab.example.com sshd[24434]: pam_unix(sshd:session): session
 closed for user root
Feb 21 18:22:42 host.lab.example.com systemd-logind[708]: Session 20 logged out.
 Waiting for processes to exit.
Feb 21 18:22:42 host.lab.example.com systemd-logind[708]: Removed session 20.
Feb 21 18:22:43 host.lab.example.com sshd[24499]: Accepted
 publickey for root from 172.25.250.250 port 48714 ssh2: RSA
 SHA256:1UGybTe52L2jzEJa1HLVKn9QUCKrTv3ZzxnMJol1Fro
Feb 21 18:22:44 host.lab.example.com systemd-logind[708]: New session 21 of user
 root.
Feb 21 18:22:44 host.lab.example.com systemd[1]: Started Session 21 of user root.
Feb 21 18:22:44 host.lab.example.com sshd[24499]: pam_unix(sshd:session): session
 opened for user root by (uid=0)
^C
[root@host ~]# 
```

To help troubleshoot problems, you might want to filter the output of the journal based on the
priority of the journal entries. The **journalctl -p** takes either the name or the number of a
priority level and shows the journal entries for entries at that priority and above. The **journalctl**
command understands the **debug, info, notice, warning, err, crit, alert**, and **emerg**
priority levels.

Run the following **journalctl** command to list journal entries at the err priority or higher:

``` bash
[root@host ~]# journalctl -p err
-- Logs begin at Wed 2019-02-20 16:01:17 +07, end at Thu 2019-02-21 18:01:01 +07.
 --
..output omitted...
Feb 20 16:01:17 host.lab.example.com kernel: Detected CPU family 6 model 13
 stepping 3
Feb 20 16:01:17 host.lab.example.com kernel: Warning: Intel Processor - this
 hardware has not undergone testing by Red Hat and might not be certif>
Feb 20 16:01:20 host.lab.example.com smartd[669]: DEVICESCAN failed: glob(3)
 aborted matching pattern /dev/discs/disc*
Feb 20 16:01:20 host.lab.example.com smartd[669]: In the system's table of devices
 NO devices found to scan
lines 1-5/5 (END) q
```

When looking for specific events, you can limit the output to a specific time frame. The
**journalctl** command has two options to limit the output to a specific time range, the **--since**
and **--until** options. Both options take a time argument in the format "*YYYY-MM-DD hh:mm:ss*"
(the double-quotes are required to preserve the space in the option). If the date is omitted,
the command assumes the current day, and if the time is omitted, the command assumes the
whole day starting at 00:00:00. Both options take **yesterday, today**, and **tomorrow** as valid
arguments in addition to the date and time field.

Run the following **journalctl** command to list all journal entries from today's records.

``` bash
[root@host ~]# journalctl --since today
-- Logs begin at Wed 2019-02-20 16:01:17 +07, end at Thu 2019-02-21 18:31:14 +07.
 --
...output omitted...
Feb 21 18:22:44 host.lab.example.com systemd-logind[708]: New session 21 of user
 root.
Feb 21 18:22:44 host.lab.example.com systemd[1]: Started Session 21 of user root.
Feb 21 18:22:44 host.lab.example.com sshd[24499]: pam_unix(sshd:session): session
Feb 21 18:31:14 host.lab.example.com dnf[24533]: Red Hat Enterprise Linux 8.0
 AppStream (dvd)    637 kB/s | 2.8 kB     00:00
Feb 21 18:31:14 host.lab.example.com dnf[24533]: Red Hat Enterprise Linux 8.0
 BaseOS (dvd)       795 kB/s | 2.7 kB     00:00
Feb 21 18:31:14 host.lab.example.com dnf[24533]: Metadata cache created.
Feb 21 18:31:14 host.lab.example.com systemd[1]: Started dnf makecache.
lines 533-569/569 (END) q
```

Run the following **journalctl** command to list all journal entries ranging from **2019-02-10
20:30:00 to 2019-02-13 12:00:00**.

``` bash
[root@host ~]# journalctl --since "2014-02-10 20:30:00" --until "2014-02-13
 12:00:00"
...output omitted...
```

You can also specify all entries since a time relative to the present. For example, to specify all
entries in the last hour, you can use the following command:

``` bash
[root@host ~]# journalctl --since "-1 hour"
...output omitted...
```

!!! info "NOTE"

	You can use other, more sophisticated time specifications with the **--since** and **--
	until** options. For some examples, see the systemd.time(7) man page.

In addition to the visible content of the journal, there are fields attached to the log entries that can
only be seen when verbose output is turned on. Any displayed extra field can be used to filter the
output of a journal query. This is useful to reduce the output of complex searches for certain events
in the journal.

``` bash
[root@host ~]# journalctl -o verbose
-- Logs begin at Wed 2019-02-20 16:01:17 +07, end at Thu 2019-02-21 18:31:14 +07.
 --
...output omitted...
Thu 2019-02-21 18:31:14.509128 +07... 
    PRIORITY=6
    _BOOT_ID=4409bbf54680496d94e090de9e4a9e23
    _MACHINE_ID=73ab164e278e48be9bf80e80714a8cd5
    SYSLOG_FACILITY=3
    SYSLOG_IDENTIFIER=systemd
    _UID=0
    _GID=0
    CODE_FILE=../src/core/job.c
    CODE_LINE=826
    CODE_FUNC=job_log_status_message
    JOB_TYPE=start
    JOB_RESULT=done
    MESSAGE_ID=39f53479d3a045ac8e11786248231fbf
    _TRANSPORT=journal
    _PID=1
    _COMM=systemd
    _EXE=/usr/lib/systemd/systemd
    _CMDLINE=/usr/lib/systemd/systemd --switched-root --system --deserialize 18
    _CAP_EFFECTIVE=3fffffffff
    _SELINUX_CONTEXT=system_u:system_r:init_t:s0
    _SYSTEMD_CGROUP=/init.scope
    _SYSTEMD_UNIT=init.scope
    _SYSTEMD_SLICE=-.slice
    UNIT=dnf-makecache.service
    MESSAGE=Started dnf makecache.
    _HOSTNAME=host.lab.example.com
    INVOCATION_ID=d6f90184663f4309835a3e8ab647cb0e
    _SOURCE_REALTIME_TIMESTAMP=1550748674509128
lines 32239-32275/32275 (END) q
```

The following list gives the common fields of the system journal that can be used to search for lines
relevant to a particular process or event.

- _COMM The name of the command

- _EXE The path to the executable for the process

- _PID The PID of the process

- _UID The UID of the user running the process

- _SYSTEMD_UNIT The systemd unit that started the process

More than one of the system journal fields can be combined to form a granular search query with
the **journalctl** command. For example, the following **journalctl** command shows all journal
entries related to the **sshd.service** systemd unit from a process with PID 1182.

``` bash
[root@host ~]# journalctl _SYSTEMD_UNIT=sshd.service _PID=1182
Apr 03 19:34:27 host.lab.example.com sshd[1182]: Accepted password for root
 from ::1 port 52778 ssh2
Apr 03 19:34:28 host.lab.example.com sshd[1182]: pam_unix(sshd:session): session
 opened for user root by (uid=0)
...output omitted...
```

!!! info "NOTE"

	For a list of commonly used journal fields, consult the systemd.journal-
	fields(7) man page.

### PRESERVING THE SYSTEM JOURNAL ###

#### STORING THE SYSTEM JOURNAL PERMANENTLY ####

By default, the system journals are kept in the **/run/log/journal** directory, which means the
journals are cleared when the system reboots. You can change the configuration settings of the
systemd-journald service in the **/etc/systemd/journald.conf** file to make the journals
persist across reboot.

The Storage parameter in the **/etc/systemd/journald.conf** file defines whether to
store system journals in a volatile manner or persistently across reboot. Set this parameter to
**persistent**, **volatile**, or **auto** as follows:

- **persistent**: stores journals in the **/var/log/journal** directory which persists across
reboots.

If the **/var/log/journal** directory does not exist, the systemd-journald service creates it.

- **volatile**: stores journals in the volatile **/run/log/journal** directory.

As the **/run** file system is temporary and exists only in the runtime memory, data stored in it,
including system journals, do not persist across reboot.

- **auto**: rsyslog determines whether to use persistent or volatile storage. If the **/var/log/
journal** directory exists, then rsyslog uses persistent storage, otherwise it uses volatile
storage.

	This is the default action if the Storage parameter is not set.

The advantage of persistent system journals is that the historic data is available immediately
at boot. However, even with a persistent journal, not all data is kept forever. The journal has a
built-in log rotation mechanism that triggers monthly. In addition, by default, the journals are not
allowed to get larger than 10% of the file system it is on, or leave less than 15% of the file system
free. These values can be tuned for both the runtime and persistent journals in **/etc/systemd/
journald.conf**. The current limits on the size of the journal are logged when the systemd-
journald process starts. The following command output shows the journal entries that reflect the
current size limits:

``` bash
[user@host ~]$ journalctl | grep -E 'Runtime|System journal'
Feb 25 13:01:46 localhost systemd-journald[147]: Runtime journal (/run/log/
journal/ae06db7da89142138408d77efea9229c) is 8.0M, max 91.4M, 83.4M free.
Feb 25 13:01:48 remotehost.lab.example.com systemd-journald[548]: Runtime journal
 (/run/log/journal/73ab164e278e48be9bf80e80714a8cd5) is 8.0M, max 91.4M, 83.4M
 free.
Feb 25 13:01:48 remotehost.lab.example.com systemd-journald[548]: System journal
 (/var/log/journal/73ab164e278e48be9bf80e80714a8cd5) is 8.0M, max 3.7G, 3.7G free.
Feb 25 13:01:48 remotehost.lab.example.com systemd[1]: Starting Tell Plymouth To
 Write Out Runtime Data...
Feb 25 13:01:48 remotehost.lab.example.com systemd[1]: Started Tell Plymouth To
 Write Out Runtime Data.
```

!!! info "NOTE"

	In the **grep** above, the pipe (**|**) symbol acts as an or indicator. That is, grep matches
	any line containing either the **Runtime** string or the **System** string from the
	**journalctl** output. This fetches the current size limits on the volatile (Runtime)
	journal store as well the persistent (System) journal store.

**Configuring Persistent System Journals**

To configure the systemd-journald service to preserve system journals persistently across
reboot, set Storage to persistent in the **/etc/systemd/journald.conf** file. Run the text
editor of your choice as the superuser to edit the **/etc/systemd/journald.conf** file.

``` bash
[Journal]
Storage=persistent
...output omitted...
```

After editing the configuration file, restart the systemd-journald service to bring the
configuration changes into effect.

``` bash
[root@host ~]# systemctl restart systemd-journald
```

If the systemd-journald service successfully restarts, you can see that the **/var/log/
journal** directory is created and contains one or more subdirectories. These subdirectories have
hexadecimal characters in their long names and contain ***.journal** files. The ***.journal** files
are the binary files that store the structured and indexed journal entries.

``` bash
[root@host ~]# ls /var/log/journal
73ab164e278e48be9bf80e80714a8cd5
[root@host ~]# ls /var/log/journal/73ab164e278e48be9bf80e80714a8cd5
system.journal  user-1000.journal
```

While the system journals persist across reboot, you get an extensive number of entries in the
output of the **journalctl** command that includes entries from the current system boot as well
as the previous ones. To limit the output to a specific system boot, use the **-b** option with the
**journalctl** command. The following **journalctl** command retrieves the entries limited to the
first system boot:

``` bash
[root@host ~]# journalctl -b 1
...output omitted...
```

The following **journalctl** command retrieves the entries limited to the second system boot. The
following argument is meaningful only if the system has been rebooted for more than twice:

``` bash
[root@host ~]# journalctl -b 2
```

The following **journalctl** command retrieves the entries limited to the current system boot:

``` bash
[root@host ~]# journalctl -b
```

!!! info "NOTE"

	When debugging a system crash with a persistent journal, it is usually required to
	limit the journal query to the reboot before the crash happened. The **-b** option can
	be accompanied by a negative number indicating how many prior system boots the
	output should include. For example, **journalctl -b -1** limits the output to only
	the previous boot.

### MAINTAINING ACCURATE TIME ###

#### SETTING LOCAL CLOCKS AND TIME ZONES ####

Correct synchronized system time is critical for log file analysis across multiple systems. The
*Network Time Protocol* (NTP) is a standard way for machines to provide and obtain correct time
information on the Internet. A machine may get accurate time information from public NTP
services on the Internet, such as the NTP Pool Project. A high-quality hardware clock to serve
accurate time to local clients is another option.

The **timedatectl** command shows an overview of the current time-related system settings,
including current time, time zone, and NTP synchronization settings of the system.

``` bash
[user@host ~]$ timedatectl
               Local time: Fri 2019-04-05 16:10:29 CDT
           Universal time: Fri 2019-04-05 21:10:29 UTC
                 RTC time: Fri 2019-04-05 21:10:29
                Time zone: America/Chicago (CDT, -0500)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

A database of time zones is available and can be listed with the **timedatectl list-timezones**
command.

``` bash
[user@host ~]$ timedatectl list-timezones
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
Africa/Algiers
Africa/Asmara
Africa/Bamako
...
```

Time zone names are based on the public time zone database that IANA maintains. Time zones are
named based on continent or ocean, then typically but not always the largest city within the time
zone region. For example, most of the US Mountain time zone is America/Denver.

Selecting the correct name can be non-intuitive in cases where localities inside the time zone
have different daylight saving time rules. For example, in the USA, much of the state of Arizona
(US Mountain time) does not have a daylight saving time adjustment at all and is in the time zone
America/Phoenix.

The command **tzselect** is useful for identifying correct zoneinfo time zone names. It interactively
prompts the user with questions about the system's location, and outputs the name of the correct
time zone. It does not make any change to the time zone setting of the system.

The superuser can change the system setting to update the current time zone using the
**timedatectl set-timezone** command. The following **timedatectl** command updates the
current time zone to **America/Phoenix**.

``` bash
[root@host ~]# timedatectl set-timezone America/Phoenix
[root@host ~]# timedatectl
               Local time: Fri 2019-04-05 14:12:39 MST
           Universal time: Fri 2019-04-05 21:12:39 UTC
                 RTC time: Fri 2019-04-05 21:12:39
                Time zone: America/Phoenix (MST, -0700)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

!!! info "NOTE"

	Should you need to use the Coordinated Universal Time (UTC) on a particular server,
	set its time zone to UTC. The **tzselect** command does not include the name of the
	UTC time zone. Use the **timedatectl set-timezone UTC** command to set the
	system's current time zone to **UTC**.

Use the **timedatectl set-time** command to change the system's current time. The time is
specified in the "*YYYY-MM-DD hh:mm:ss*" format, where either date or time can be omitted. The
following **timedatectl** command changes the time to **09:00:00**.

``` bash
[root@host ~]# timedatectl set-time 9:00:00
[root@serverX ~]$ timedatectl
               Local time: Fri 2019-04-05 09:00:27 MST
           Universal time: Fri 2019-04-05 16:00:27 UTC
                 RTC time: Fri 2019-04-05 16:00:27
                Time zone: America/Phoenix (MST, -0700)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

The **timedatectl set-ntp** command enables or disables NTP synchronization for automatic
time adjustment. The option requires either a **true or false** argument to turn it on or off. The
following **timedatectl** command turns on NTP synchronization.

``` bash
[root@host ~]# timedatectl set-ntp true
```

!!! info "NOTE"

	In Red Hat Enterprise Linux 8, the timedatectl set-ntp command will adjust
	whether or not chronyd NTP service is operating. Other Linux distributions might
	use this setting to adjust a different NTP or SNTP service.
	
	Enabling or disabling NTP using other utilities in Red Hat Enterprise Linux, such as in
	the graphical GNOME Settings application, also updates this setting.

### CONFIGURING AND MONITORING CHRONYD ###

The chronyd service keeps the usually-inaccurate local hardware clock (RTC) on track by
synchronizing it to the configured NTP servers. If no network connectivity is available, chronyd
calculates the RTC clock drift, which is recorded in the **driftfile** specified in the **/etc/
chrony.conf** configuration file.

By default, the chronyd service uses servers from the NTP Pool Project for the time
synchronization and does not need additional configuration. It may be useful to change the NTP
servers when the machine in question is on an isolated network.

The stratum of the NTP time source determines its quality. The stratum determines the number
of hops the machine is away from a high-performance reference clock. The reference clock is a
**stratum 0** time source. An NTP server directly attached to it is a **stratum 1**, while a machine
synchronizing time from the NTP server is a **stratum 2** time source.

The server and peer are the two categories of time sources that you can in the **/etc/
chrony.conf** configuration file. The server is one stratum above the local NTP server, and the
peer is at the same stratum level. More than one server and more than one peer can be specified,
one per line.

The first argument of the **server** line is the IP address or DNS name of the NTP server. Following
the server IP address or name, a series of options for the server can be listed. It is recommended to
use the **iburst** option, because after the service starts, four measurements are taken in a short
time period for a more accurate initial clock synchronization.

The following **server classroom.example.com iburst** line in the **/etc/chrony.conf** file
causes the chronyd service to use the classroom.example.com NTP time source.

``` bash
# Use public servers from the pool.ntp.org project.
...output omitted...
server classroom.example.com iburst
...output omitted...
```

After pointing **chronyd** to the local time source, classroom.example.com, you should restart
the service.

``` bash
[root@host ~]# systemctl restart chronyd
```

The **chronyc** command acts as a client to the chronyd service. After setting up NTP
synchronization, you should verify that the local system is seamlessly using the NTP server to
synchronize the system clock using the **chrony sources** command. For more verbose output
with additional explanations about the output, use the **chronyc sources -v** command.

``` bash
[root@host ~]# chronyc sources -v
210 Number of sources = 1
  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||                                                /   xxxx = adjusted offset,
||         Log2(Polling interval) -.             |    yyyy = measured offset,
||                                  \            |    zzzz = estimated error.
||                                   |           |
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* classroom.example.com         8   6    17    23   -497ns[-7000ns] +/-  956us
```

The ***** character in the **S** (Source state) field indicates that the classroom.example.com server
has been used as a time source and is the NTP server the machine is currently synchronized to.

### SUMMARY ###

In this chapter, you learned:

- The systemd-journald and rsyslog services capture and write log messages to the
appropriate files.

- The **/var/log** directory contains log files.

- Periodic rotation of log files prevent them from filling up the file system space.

- The systemd journals are temporary and do not persist across reboot.

- The chronyd service helps to synchronize time settings with a time source.

### LAB

pdf - 404