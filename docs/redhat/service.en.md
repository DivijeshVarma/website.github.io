---
title: "Controlling Services And Daemons"
---

### IDENTIFYING AUTOMATICALLY STARTED SYSTEM PROCESSES ###

#### INTRODUCTION TO systemd ####

The systemd daemon manages startup for Linux, including service startup and service
management in general. It activates system resources, server daemons, and other processes both
at boot time and on a running system.

Daemons are processes that either wait or run in the background, performing various tasks.
Generally, daemons start automatically at boot time and continue to run until shutdown or until
they are manually stopped. It is a convention for names of many daemon programs to end in the
letter **d**.

A service in the systemd sense often refers to one or more daemons, but starting or stopping a
service may instead make a one-time change to the state of the system, which does not involve
leaving a daemon process running afterward (called oneshot).

In Red Hat Enterprise Linux, the first process that starts (PID 1) is systemd. A few of the features
provided by systemd include:

- Parallelization capabilities (starting multiple services simultaneously), which increase the boot
speed of a system.

- On-demand starting of daemons without requiring a separate service.

- Automatic service dependency management, which can prevent long timeouts. For example, a
network-dependent service will not attempt to start up until the network is available.

- A method of tracking related processes together by using Linux control groups.

#### DESCRIBING SERVICE UNITS ####

systemd uses units to manage different types of objects. Some common unit types are listed
below:

- Service units have a **.service** extension and represent system services. This type of unit is
used to start frequently accessed daemons, such as a web server.

- Socket units have a **.socket** extension and represent inter-process communication (IPC)
sockets that systemd should monitor. If a client connects to the socket, systemd will start a
daemon and pass the connection to it. Socket units are used to delay the start of a service at
boot time and to start less frequently used services on demand.

- Path units have a **.path** extension and are used to delay the activation of a service until
a specific file system change occurs. This is commonly used for services which use spool
directories such as a printing system.

The **systemctl** command is used to manage units. For example, display available unit types with
the **systemctl -t help** command.

!!! danger "IMPORTANT"

	When using **systemctl**, you can abbreviate unit names, process tree entries, and
	unit descriptions.

#### LISTING SERVICE UNITS ####

You use the **systemctl** command to explore the current state of the system. For example, the
following command lists all currently loaded service units, paginating the output using **less**.

``` bash
[root@host ~]# systemctl list-units --type=service
UNIT                               LOAD   ACTIVE SUB     DESCRIPTION
atd.service                        loaded active running Job spooling tools
auditd.service                     loaded active running Security Auditing Service
chronyd.service                    loaded active running NTP client/server
crond.service                      loaded active running Command Scheduler
dbus.service                       loaded active running D-Bus System Message Bus
...output omitted...
```

The above output limits the type of unit listed to service units with the --type=service option.
The output has the following columns:

**Columns in the systemctl list-units Command Output**

**UNIT**

	The service unit name.

**LOAD**

	Whether systemd properly parsed the unit's configuration and loaded the unit into memory.

**ACTIVE**

	The high-level activation state of the unit. This information indicates whether the unit has
	started successfully or not.

**SUB**

	The low-level activation state of the unit. This information indicates more detailed information
	about the unit. The information varies based on unit type, state, and how the unit is executed.
	
**DESCRIPTION**

	The short description of the unit.

By default, the **systemctl list-units --type=service** command lists only the service
units with active activation states. The **--all** option lists all service units regardless of the
activation states. Use the **--state=** option to filter by the values in the **LOAD, ACTIVE**, or **SUB**
fields.

``` bash
[root@host ~]# systemctl list-units --type=service --all
UNIT                          LOAD      ACTIVE   SUB     DESCRIPTION
  atd.service                 loaded    active   running Job spooling tools
  auditd.service              loaded    active   running Security Auditing ...
  auth-rpcgss-module.service  loaded    inactive dead    Kernel Module ...
  chronyd.service             loaded    active   running NTP client/server
  cpupower.service            loaded    inactive dead    Configure CPU power ...
  crond.service               loaded    active   running Command Scheduler
  dbus.service                loaded    active   running D-Bus System Message Bus
●display-manager.service     not-found inactive dead    display-manager.service
...output omitted...
```

The **systemctl** command without any arguments lists units that are both loaded and active.

``` bash
[root@host ~]# systemctl
UNIT                                 LOAD   ACTIVE SUB       DESCRIPTION
proc-sys-fs-binfmt_misc.automount    loaded active waiting   Arbitrary...
sys-devices-....device               loaded active plugged   Virtio network...
sys-subsystem-net-devices-ens3.deviceloaded active plugged   Virtio network...
...
-.mount                              loaded active mounted   Root Mount
boot.mount                           loaded active mounted   /boot
...
systemd-ask-password-plymouth.path   loaded active waiting   Forward Password...
systemd-ask-password-wall.path       loaded active waiting   Forward Password...
init.scope                           loaded active running   System and Servi...
session-1.scope                      loaded active running   Session 1 of...
atd.service                          loaded active running   Job spooling tools
auditd.service                       loaded active running   Security Auditing...
chronyd.service                      loaded active running   NTP client/server
crond.service                        loaded active running   Command Scheduler
...output omitted...
```

The **systemctl list-units** command displays units that the systemd service attempts to
parse and load into memory; it does not display installed, but not enabled, services. To see the
state of all unit files installed, use the **systemctl list-unit-files** command. For example:

``` bash
[root@host ~]# systemctl list-unit-files --type=service
UNIT FILE                                   STATE
arp-ethers.service                          disabled
atd.service                                 enabled
auditd.service                              enabled
auth-rpcgss-module.service                  static
autovt@.service                             enabled
blk-availability.service                    disabled
...output omitted...
```

In the output of the **systemctl list-units-files** command, valid entries for the **STATE** field
are **enabled, disabled, static**, and **masked**.

#### VIEWING SERVICE STATES ####

View the status of a specific unit with **systemctl status name.type**. If the unit type is not
provided, **systemctl** will show the status of a service unit, if one exists.

``` bash
[root@host ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset:
 enabled)
   Active: active (running) since Thu 2019-02-14 12:07:45 IST; 7h ago
 Main PID: 1073 (sshd)
    CGroup: /system.slice/sshd.service
           └─1073 /usr/sbin/sshd -D ...
Feb 14 11:51:39 host.example.com systemd[1]: Started OpenSSH server daemon.
Feb 14 11:51:39 host.example.com sshd[1073]: Could not load host key: /et...y
Feb 14 11:51:39 host.example.com sshd[1073]: Server listening on 0.0.0.0 ....
Feb 14 11:51:39 host.example.com sshd[1073]: Server listening on :: port 22.
Feb 14 11:53:21 host.example.com sshd[1270]: error: Could not load host k...y
Feb 14 11:53:22 host.example.com sshd[1270]: Accepted password for root f...2
...output omitted...
```

This command displays the current status of the service. The meaning of the fields are:

**Service Unit Information**

| FIELD    | DESCRIPTION                                                                  |
|:--------:|:-----------------------------------------------------------------------------|
| Loaded   | Whether the service unit is loaded into memory.                              |
| Active   | Whether the service unit is running and if so, how long it has been running. |
| Main PID | The main process ID of the service, including the command name.              |
| Status   | Additional information about the service.                                    |

Several keywords indicating the state of the service can be found in the status output:

**Service States in the Output of systemctl**

| KEYWORD          | DESCRIPTION                                                             |
|:----------------:|:------------------------------------------------------------------------|
| loaded           | Unit configuration file has been processed.                             |
| active (running) | Running with one or more continuing processes.                          |
| active (exited)  | Successfully completed a one-time configuration.                        |
| active (waiting) | Running but waiting for an event.                                       |
| inactive         | Not running.                                                            |
| enabled          | Is started at boot time.                                                |
| disabled         | Is not set to be started at boot time.                                  |
| static           | Cannot be enabled, but may be started by an enabled unit automatically. |

!!! info "NOTE"

	The **systemctl status NAME** command replaces the **service NAME status**
	command used in Red Hat Enterprise Linux 6 and earlier.

#### VERIFYING THE STATUS OF A SERVICE ####

The **systemctl** command provides methods for verifying the specific states of a service. For
example, use the following command to verify that the a service unit is currently active (running):

``` bash
[root@host ~]# systemctl is-active sshd.service
active
```

The command returns state of the service unit, which is usually **active** or **inactive**.

Run the following command to verify whether a service unit is enabled to start automatically
during system boot:

``` bash
[root@host ~]# systemctl is-enabled sshd.service
enabled
```

The command returns whether the service unit is enabled to start at boot time, which is usually
**enabled** or **disabled**.

To verify whether the unit failed during startup, run the following command:

``` bash
[root@host ~]# systemctl is-failed sshd.service
active
```

The command either returns **active** if it is properly running or **failed** if an error has occurred
during startup. In case the unit was stopped, it returns **unknown** or **inactive**.

To list all the failed units, run the **systemctl --failed --type=service** command.

### CONTROLLING SYSTEM SERVICES ###

#### STARTING AND STOPPING SERVICES ####

Services need to be stopped or started manually for a number of reasons: perhaps the service
needs to be updated; the configuration file may need to be changed; or a service may need to be
uninstalled; or an administrator may manually start an infrequently used service.

To start a service, first verify that it is not running with **systemctl status**. Then, use the
**systemctl start** command as the root user (using **sudo** if necessary). The example below
shows how to start the sshd.service service:

``` bash
[root@host ~]# systemctl start sshd.service
```

The systemd service looks for .service files for service management in commands in the
absence of the service type with the service name. Thus the above command can be executed as:

``` bash
[root@host ~]# systemctl start sshd
```

To stop a currently running service, use the stop argument with the systemctl command. The
example below shows how to stop the sshd.service service:

``` bash
[root@host ~]# systemctl stop sshd.service
```

#### RESTARTING AND RELOADING SERVICES ####

During a restart of a running service, the service is stopped and then started. On the restart of
service, the process ID changes and a new process ID gets associated during the startup. To restart
a running service, use the **restart** argument with the **systemctl** command. The example below
shows how to restart the sshd.service service:

``` bash
[root@host ~]# systemctl restart sshd.service
```

Some services have the ability to reload their configuration files without requiring a restart. This
process is called a service reload. Reloading a service does not change the process ID associated
with various service processes. To reload a running service, use the **reload** argument with the
**systemctl** command. The example below shows how to reload the sshd.service service after
configuration changes:

``` bash
[root@host ~]# systemctl reload sshd.service
```

In case you are not sure whether the service has the functionality to reload the configuration
file changes, use the **reload-or-restart** argument with the **systemctl** command. The
command reloads the configuration changes if the reloading functionality is available. Otherwise,
the command restarts the service to implements the new configuration changes:

``` bash
[root@host ~]# systemctl reload-or-restart sshd.service
```

#### LISTING UNIT DEPENDENCIES ####

Some services require that other services be running first, creating dependencies on the other
services. Other services are not started at boot time but rather only on demand. In both cases,
systemd and **systemctl** start services as needed whether to resolve the dependency or to
start an infrequently used service. For example, if the CUPS print service is not running and a
file is placed into the print spool directory, then the system will start CUPS-related daemons or
commands to satisfy the print request.

``` bash
[root@host ~]# systemctl stop cups.service
Warning: Stopping cups, but it can still be activated by:
  cups.path
  cups.socket
```

To completely stop printing services on a system, stop all three units. Disabling the service disables
the dependencies.

The **systemctl list-dependencies UNIT** command displays a hierarchy mapping of
dependencies to start the service unit. To list reverse dependencies (units that depend on the
specified unit), use the **--reverse** option with the command.

``` bash
[root@host ~]# systemctl list-dependencies sshd.service
sshd.service
● ├─system.slice
● ├─sshd-keygen.target
● │ ├─sshd-keygen@ecdsa.service
● │ ├─sshd-keygen@ed25519.service
● │ └─sshd-keygen@rsa.service
● └─sysinit.target
...output omitted...
```

#### MASKING AND UNMASKING SERVICES ####

At times, a system may have different services installed that are conflicting with each other.
For example, there are multiple methods to manage mail servers (postfix and sendmail, for
example). Masking a service prevents an administrator from accidentally starting a service that
conflicts with others. Masking creates a link in the configuration directories to the **/dev/null** file
which prevents the service from starting.

``` bash
[root@host ~]# systemctl mask sendmail.service
Created symlink /etc/systemd/system/sendmail.service → /dev/null.
[root@host ~]# systemctl list-unit-files --type=service
UNIT FILE                                   STATE
...output omitted...
sendmail.service                            masked
...output omitted...
```

Attempting to start a masked service unit fails with the following output:


``` bash
[root@host ~]# systemctl start sendmail.service
```

Failed to start sendmail.service: Unit sendmail.service is masked.
Use the systemctl unmask command to unmask the service unit.

``` bash
[root@host ~]# systemctl unmask sendmail
Removed /etc/systemd/system/sendmail.service.
```

!!! danger "IMPORTANT"

	A disabled service can be started manually or by other unit files but it does not start
	automatically at boot. A masked service does not start manually or automatically.

#### ENABLING SERVICES TO START OR STOP AT BOOT ####

Starting a service on a running system does not guarantee that the service automatically starts
when the system reboots. Similarly, stopping a service on a running system does not keep it from
starting again when the system reboots. Creating links in the systemd configuration directories
enables the service to start at boot. The **systemctl** commands creates and removes these links.
To start a service at boot, use the **systemctl enable** command.

``` bash
[root@root ~]# systemctl enable sshd.service
Created symlink /etc/systemd/system/multi-user.target.wants/sshd.service → /usr/
lib/systemd/system/sshd.service.
```

The above command creates a symbolic link from the service unit file, usually in the **/usr/lib/
systemd/system** directory, to the location on disk where systemd looks for files, which is in the
**/etc/systemd/system/TARGETNAME.target.wants** directory. Enabling a service does not
start the service in the current session. To start the service and enable it to start automatically
during boot, execute both the **systemctl start** and **systemctl enable** commands.

To disable the service from starting automatically, use the following command, which removes
the symbolic link created while enabling a service. Note that disabling a service does not stop the
service.

``` bash
[root@host ~]# systemctl disable sshd.service
Removed /etc/systemd/system/multi-user.target.wants/sshd.service.
```

To verify whether the service is enabled or disable, use the **systemctl is-enabled** command.

#### SUMMARY OF *systemctl* COMMANDS ####

Services can be started and stopped on a running system and enabled or disabled for an automatic
start at boot time.

**Useful Service Management Commands**

| TASK                                                                        | COMMAND                              |
|:----------------------------------------------------------------------------|:-------------------------------------|
| View detailed information about a unit state.                               | **systemctl status UNIT**            |
| Stop a service on a running system.                                         | **systemctl stop UNIT**              |
| Start a service on a running system.                                        | **systemctl start UNIT**             |
| Restart a service on a running system.                                      | **systemctl restart UNIT**           |
| Reload the configuration file of a running service.                         | **systemctl reload UNIT**            |
| Completely disable a service from being started, both manually and at boot. | **systemctl mask UNIT**              |
| Make a masked service available.                                            | **systemctl unmask UNIT**            |
| Configure a service to start at boot time.                                  | **systemctl enable UNIT**            |
| Disable a service from starting at boot time.                               | **systemctl disable UNIT**           |
| List units required and wanted by the specified unit.                       | **systemctl list-dependencies UNIT** |

### SUMMARY ###

In this chapter, you learned:

- systemd provides a method for activating system resources, server daemons, and other
processes, both at boot time and on a running system.

- Use the **systemctl** to start, stop, reload, enable, and disable services.

- Use the **systemctl status** command to determine the status of system daemons and
network services started by systemd.

- The **systemctl list-dependencies** command lists all service units upon which a specific
service unit depends.

- systemd can mask a service unit so that it does not run even to satisfy dependencies.

### LAB

pdf - 326