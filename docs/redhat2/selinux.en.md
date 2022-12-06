---
title: "Managing Selinux Security"
---

### CHANGING THE SELINUX ENFORCEMENT MODE

#### HOW SELINUX PROTECTS RESOURCES

SELinux provides a critical security purpose in Linux, permitting or denying access to files and
other resources that are significantly more precise than user permissions.

File permissions control which users or groups of users can access which specific files. However, a
user given read or write access to any specific file can use that file in any way that user chooses,
even if that use is not how the file should be used.

For example, with write access to a file, should a structured data file designed to be written to
using only a particular program, be allowed to be opened and modified by other editors that could
result in corruption?

File permissions cannot stop such undesired access. They were never designed to control how a
file is used, but only who is allowed to read, write, or run a file.

SELinux consists of sets of policies, defined by the application developers, that declare exactly
what actions and accesses are proper and allowed for each binary executable, configuration
file, and data file used by an application. This is known as a targeted policy because one policy is
written to cover the activities of a single application. Policies declare predefined labels that are
placed on individual programs, files, and network ports.

#### WHY USE SECURITY ENHANCED LINUX?

Not all security issues can be predicted in advance. SELinux enforces a set of access rules
preventing a weakness in one application from affecting other applications or the underlying
system. SELinux provides an extra layer of security; it also adds a layer of complexity which can
be off-putting to people new to this subsystem. Learning to work with SELinux may take time but
the enforcement policy means that a weakness in one part of the system does not spread to other
parts. If SELinux works poorly with a particular subsystem, you can turn off enforcement for that
specific service until you find a solution to the underlying problem.

SELinux has three modes:

- Enforcing: SELinux is enforcing access control rules. Computers generally run in this mode.

- Permissive: SELinux is active but instead of enforcing access control rules, it records warnings of
rules that have been violated. This mode is used primarily for testing and troubleshooting.

- Disabled: SELinux is turned off entirely: no SELinux violations are denied, nor even recorded.
Discouraged!

#### BASIC SELINUX SECURITY CONCEPTS

Security Enhanced Linux (SELinux) is an additional layer of system security. The primary goal of
SELinux is to protect user data from system services that have been compromised. Most Linux
administrators are familiar with the standard user/group/other permission security model. This
is a user and group based model known as discretionary access control. SELinux provides an
additional layer of security that is object-based and controlled by more sophisticated rules, known
as mandatory access control.

To allow remote anonymous access to a web server, firewall ports must be opened. However,
this gives malicious people an opportunity to crack the system through a security exploit. If they
succeed in compromising the web server process they gain its permissions. Specifically, the
permissions of the apache user and the apache group. That user and group has read access to
the document root, **/var/www/html**. It also has access to **/tmp**, and **/var/tmp**, and any other
files and directories that are world writable.

SELinux is a set of security rules that determine which process can access which files, directories,
and ports. Every file, process, directory, and port has a special security label called an SELinux
context. A context is a name used by the SELinux policy to determine whether a process can
access a file, directory, or port. By default, the policy does not allow any interaction unless an
explicit rule grants access. If there is no allow rule, no access is allowed.

SELinux labels have several contexts: user, role, type, and sensitivity. The targeted policy,
which is the default policy enabled in Red Hat Enterprise Linux, bases its rules on the third context:
the type context. Type context names usually end with **_t**.

![SElinux file context](https://www.thegeeksearch.com/images/post/wp-content-uploads-2020-10-SELinux-file-context_hu2c9107b5657b0003c389b5a17363ab6e_11135_750x0_resize_q100_h2_box.webp)

The type context for a web server is **httpd_t**. The type context for files and directories normally
found in **/var/www/html** is **httpd_sys_content_t**. The contexts for files and directories
normally found in **/tmp** and **/var/tmp** is **tmp_t**. The type context for web server ports is
**http_port_t**.

Apache has a type context of **httpd_t**. There is a policy rule that permits Apache access to files
and directories with the **httpd_sys_content_t** type context. By default files found in **/var/
www/html** and other web server directories have the **httpd_sys_content_t** type context.
There is no **allow** rule in the policy for files normally found in **/tmp** and **/var/tmp**, so access
is not permitted. With SELinux enabled, a malicious user who had compromised the web server
process could not access the **/tmp** directory.

The MariaDB server has a type context of **mysqld_t**. By default, files found in **/data/mysql**
have the **mysqld_db_t** type context. This type context allows MariaDB access to those files but
disables access by other services, such as the Apache web service.

![SElinux access](https://access.redhat.com/webassets/avalon/d/Red_Hat_Enterprise_Linux-7-SELinux_Users_and_Administrators_Guide-en-US/images/f7b4d4a7ee54ec0153a3422060470895/selinux-intro-apache-mariadb.png)

Many commands that deal with files use the **-Z** option to display or set SELinux contexts. For
instance, **ps, ls, cp**, and **mkdir** all use the **-Z** option to display or set SELinux contexts.

```bash
[root@host ~]# ps axZ
LABEL                           PID TTY     STAT    TIME  COMMAND
system_u:system_r:init_t:s0       1  ?        Ss     0:09  /usr/lib/systemd/...
system_u:system_r:kernel_t:s0     2  ?        S      0:00  [kthreadd]
system_u:system_r:kernel_t:s0     3  ?        S      0:00  [ksoftirqd/0]
...output omitted...
[root@host ~]# systemctl start httpd
[root@host ~]# ps -ZC httpd
LABEL                           PID  TTY           TIME    CMD
system_u:system_r:httpd_t:s0    1608  ?         00:00:05   httpd
system_u:system_r:httpd_t:s0    1609  ?         00:00:00   httpd
...output omitted...
[root@host ~]# ls -Z /home
drwx------. root   root     system_u:object_r:lost_found_t:s0 lost+found
drwx------. student student unconfined_u:object_r:user_home_dir_t:s0 student
drwx------. visitor visitor unconfined_u:object_r:user_home_dir_t:s0 visitor
[root@host ~]# ls -Z /var/www
drwxr-xr-x. root root system_u:object_r:httpd_sys_script_exec_t:s0 cgi-bin
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 error
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 html
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 icons

```

#### CHANGING THE CURRENT SELINUX MODE

The SELinux subsystem provides tools to display and change modes. To determine the current
SELinux mode, run the **getenforce** command. To set SELinux to a different mode, use the
**setenforce** command:

```bash
[user@host ~]# getenforce
Enforcing
[user@host ~]# setenforce
usage:
 setenforce [ Enforcing | Permissive | 1 | 0 ]
[user@host ~]# setenforce 0
[user@host ~]# getenforce
Permissive
[user@host ~]# setenforce Enforcing
[user@host ~]# getenforce
Enforcing
```
Alternatively, you can set the SELinux mode at boot time by passing a parameter to the kernel:
the kernel argument of **enforcing=0** boots the system into permissive mode; a value of
**enforcing=1** sets enforcing mode. You can also disable SELinux completely by passing on the
kernel parameter **selinux=0**. A value of **selinux=1** enables SELinux.

#### SETTING THE DEFAULT SELINUX MODE

You can also configure SELinux persistently using the **/etc/selinux/config** file. In the
example below (the default configuration), the configuration file sets SELinux to **enforcing**. The
comments also show the other valid values: **permissive** and **disabled**.

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#
 enforcing - SELinux security policy is enforced.
#
 permissive - SELinux prints warnings instead of enforcing.
#
 disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of these two values:
#
 targeted - Targeted processes are protected,
#
 minimum - Modification of targeted policy. Only selected processes
#
 are protected.
#
 mls - Multi Level Security protection.
SELINUXTYPE=targeted

```
The system reads this file at boot time and configures SELinux as shown. Kernel arguments
(**selinux=0|1** and **enforcing=0|1**) override this configuration.

### CONTROLLING SELINUX FILE CONTEXTS

#### INITIAL SELINUX CONTEXT

On systems running SELinux, all processes and files are labeled. The label represents the security
relevant information, known as the SELinux context.

New files typically inherit their SELinux context from the parent directory, thus ensuring that they
have the proper context.

But this inheritance procedure can be undermined in two different ways. First, if you create a file
in a different location from the ultimate intended location and then move the file, the file still has
the SELinux context of the directory where it was created, not the destination directory. Second, if
you copy a file preserving the SELinux context, as with the **cp -a** command, the SELinux context
reflects the location of the original file.

The following example demonstrates inheritance and its pitfalls. Consider these two files created
in **/tmp**, one moved to **/var/www/html** and the second one copied to the same directory. Note
the SELinux contexts on the files. The file that was moved to the **/var/www/html** directory
retains the file context for the **/tmp** directory. The file that was copied to the **/var/www/html**
directory inherited SELinux context from the **/var/www/html** directory.

The **ls -Z** command displays the SELinux context of a file. Note the label of the file.

```bash
[root@host ~]# ls -Z /var/www/html/index.html
-rw-r--r--. root root unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/
index.html
```

And the ls -Zd command displays the SELinux context of a directory:

```bash
[root@host ~]# ls -Zd /var/www/html/
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/html/
```

Note that the **/var/www/html/index.html** has the same label as the parent directory **/
var/www/html/**. Now, create files outside of the **/var/www/html** directory and note their file
context:

```bash
[root@host ~]# touch /tmp/file1 /tmp/file2
[root@host ~]# ls -Z /tmp/file*
unconfined_u:object_r:user_tmp_t:s0 /tmp/file1
unconfined_u:object_r:user_tmp_t:s0 /tmp/file2

```
Move one of these files to the **/var/www/html** directory, copy another, and note the label of
each:

```bash
[root@host ~]# mv /tmp/file1 /var/www/html/
[root@host ~]# cp /tmp/file2 /var/www/html/

[root@host ~]# ls -Z /var/www/html/file*
unconfined_u:object_r:user_tmp_t:s0 /var/www/html/file1
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2

```
The moved file maintains its original label while the copied file inherits the label from the **/var/
www/html** directory. **unconfined_u:** is the user, **object_r:** denotes the role, and **s0** is the
level. A sensitivity level of 0 is the lowest possible sensitivity level.

#### CHANGING THE SELINUX CONTEXT OF A FILE

Commands to change the SELinux context on files include **semanage fcontext, restorecon**,
and **chcon**.

The preferred method to set the SELinux context for a file is to declare the default labeling for a
file using the **semanage fcontext** command and then applying that context to the file using the
**restorecon** command. This ensures that the labeling will be as desired even after a complete
relabeling of the file system.

The **chcon** command changes SELinux contexts. **chcon** sets the security context on the file,
stored in the file system. It is useful for testing and experimenting. However, it does not save
context changes in the SELinux context database. When a **restorecon** command runs, changes
made by the **chcon** command also do not survive. Also, if the entire file system is relabeled, the
SELinux context for files changed using **chcon** are reverted.

The following screen shows a directory being created. The directory has a type value of
**default_t**.

```bash
[root@host ~]# mkdir /virtual
[root@host ~]# ls -Zd /virtual
drwxr-xr-x. root root unconfined_u:object_r:default_t:s0 /virtual

```
The **chcon** command changes the file context of the **/virtual** directory: the type value changes
to **httpd_sys_content_t**.

```bash
[root@host ~]# chcon -t httpd_sys_content_t /virtual
[root@host ~]# ls -Zd /virtual
drwxr-xr-x. root root unconfined_u:object_r:httpd_sys_content_t:s0 /virtual

```
The **restorecon** command runs and the type value returns to the value of **default_t**. Note the
**Relabeled** message.

```bash
[root@host ~]# restorecon -v /virtual
Relabeled /virtual from unconfined_u:object_r:httpd_sys_content_t:s0 to
unconfined_u:object_r:default_t:s0
[root@host ~]# ls -Zd /virtual
drwxr-xr-x. root root unconfined_u:object_r:default_t:s0 /virtual

```
#### DEFINING SELINUX DEFAULT FILE CONTEXT RULES

The **semanage fcontext** command displays and modifies the rules that **restorecon** uses to
set default file contexts. It uses extended regular expressions to specify the path and file names.
The most common extended regular expression used in **fcontext** rules is **(/.*)?**, which means
“optionally, match a / followed by any number of characters”. It matches the directory listed before
the expression and everything in that directory recursively.

**Basic File Context Operations**

The following table is a reference for **semanage fcontext** options to add, remove or list SELinux
file contexts.

**semanage fcontext commands**


OPTION | DESCRIPTION |
---------|----------|
 -a, --add | Add a record of the specified object type |
 -d, --delete | Delete a record of the specified object type |
 -l, --list | List records of the specified object type |


To ensure that you have the tools to manage SELinux contexts, install the **policycoreutil**
package and the **policycoreutil-python** package if needed. These contain the **restorecon**
command and **semanage** command, respectively.

To ensure that all files in a directory have the correct file context run the **semanage fcontext -**
**l** followed by the **restorecon** command. In the following example, note the file context of each
file before and after the **semanage** and **restorecon** commands run.

```bash
[root@host ~]# ls -Z /var/www/html/file*
unconfined_u:object_r:user_tmp_t:s0 /var/www/html/file1
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2

[root@host ~]# semanage fcontext -l
...output omitted...
/var/www(/.*)?
 all files
 system_u:object_r:httpd_sys_content_t:s0
...output omitted...
[root@host; ~]# restorecon -Rv /var/www/
Relabeled /var/www/html/file1 from unconfined_u:object_r:user_tmp_t:s0 to
unconfined_u:object_r:httpd_sys_content_t:s0
[root@host ~]# ls -Z /var/www/html/file*
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file1
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2
```

The following example shows how to use semanage to add a context for a new directory.

```bash
[root@host ~]# mkdir /virtual
[root@host ~]# touch /virtual/index.html
[root@host ~]# ls -Zd /virtual/
drwxr-xr-x. root root unconfined_u:object_r:default_t:s0 /virtual/

[root@host ~]# ls -Z /virtual/
-rw-r--r--. root root unconfined_u:object_r:default_t:s0 index.html
[root@host ~]# semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'
[root@host ~]# restorecon -RFvv /virtual
[root@host ~]# ls -Zd /virtual/
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /virtual/
[root@host ~]# ls -Z /virtual/
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 index.html
```

### ADJUSTING SELINUX POLICY WITH BOOLEANS

#### SELINUX BOOLEANS

SELinux booleans are switches that change the behavior of the SELinux policy. SELinux booleans
are rules that can be enabled or disabled. They can be used by security administrators to tune the
policy to make selective adjustments.

The SELinux man pages, provided with the selinux-policy-doc package, describe the purpose of
the available booleans. The **man -k '_selinux'** command lists these man pages.

Commands useful for managing SELinux booleans include **getsebool**, which lists booleans and
their state, and **setsebool** which modifies booleans. **setsebool -P** modifies the SELinux policy
to make the modification persistent. And **semanage boolean -l** reports on whether or not a
boolean is persistent, along with a short description of the boolean.

Non-privileged users can run the **getsebool** command, but you must be a superuser to run
**semanage boolean -l** and **setsebool -P**.

```bash
[user@host ~]$ getsebool -a
abrt_anon_write --> off
abrt_handle_event --> off
abrt_upload_watch_anon_write --> on
antivirus_can_scan_system --> off
antivirus_use_jit --> off
...output omitted...
[user@host ~]$ getsebool httpd_enable_homedirs
httpd_enable_homedirs --> off

[user@host ~]$ setsebool httpd_enable_homedirs on
Could not change active booleans. Please try as root: Permission denied
[user@host ~]$ sudo setsebool httpd_enable_homedirs on
[user@host ~]$ sudo semanage boolean -l | grep httpd_enable_homedirs
httpd_enable_homedirs  (on , off)  Allow httpd to enable homedirs
[user@host ~]$ getsebool httpd_enable_homedirs
httpd_enable_homedirs --> on

```
The **-P** option writes all pending values to the policy, making them persistent across reboots. In
the example that follows, note the values in parentheses: both are now set to **on**.

```bash
[user@host ~]$ setsebool -P httpd_enable_homedirs on
[user@host ~]$ sudo semanage boolean -l | grep httpd_enable_homedirs
httpd_enable_homedirs    (on , on)   Allow httpd to enable homedirs

```
To list booleans in which the current state differs from the default state, run **semanage boolean
-l -C**.

```bash
[user@host ~]$ sudo semanage boolean -l -C
SELinux boolean    State        Default Description

cron_can_relabel   (off , on)    Allow cron to can relabel
```

### INVESTIGATING AND RESOLVING SELINUX ISSUES

#### TROUBLESHOOTING SELINUX ISSUES

It is important to understand what actions you must take when SELinux prevents access to files on
a server that you know should be accessible. Use the following steps as a guide to troubleshooting
these issues:

1. Before thinking of making any adjustments, consider that SELinux may be doing its job
correctly by prohibiting the attempted access. If a web server tries to access files in **/home**,
this could signal a compromise of the service if web content is not published by users.
If access should have been granted, then additional steps need to be taken to solve the
problem.

2. The most common SELinux issue is an incorrect file context. This can occur when a file is
created in a location with one file context and moved into a place where a different context is
expected. In most cases, running **restorecon** will correct the issue. Correcting issues in this
way has a very narrow impact on the security of the rest of the system.

3. Another remedy for overly restrictive access could be the adjustment of a Boolean. For
example, the **ftpd_anon_write** boolean controls whether anonymous FTP users can upload
files. You must turn this boolean on to permit anonymous FTP users to upload files to a server.
Adjusting booleans requires more care because they can have a broad impact on system
security.

4. It is possible that the SELinux policy has a bug that prevents a legitimate access. Since
SELinux has matured, this is a rare occurrence. When it is clear that a policy bug has been
identified, contact Red Hat support to report the bug so it can be resolved.

#### MONITORING SELINUX VIOLATIONS

Install the setroubleshoot-server package to send SELinux messages to **/var/log/messages**.
**setroubleshoot-server** listens for audit messages in **/var/log/audit/audit.log** and
sends a short summary to **/var/log/messages**. This summary includes unique identifiers (UUID)
for SELinux violations that can be used to gather further information. The **sealert -l UUID**
command is used to produce a report for a specific incident. Use **sealert -a /var/log/**
**audit/audit.log** to produce reports for all incidents in that file.

Consider the following sample sequence of commands on a standard Apache web server:

```bash
[root@host ~]# touch /root/file3
[root@host ~]# mv /root/file3 /var/www/html
[root@host ~]# systemctl start httpd
[root@host ~]# curl http://localhost/file3
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access /file3
on this server.</p>
</body></html>
```

You expect the web server to deliver the contents of **file3** but instead it returns a **permission
denied** error. Inspecting both **/var/log/audit/audit.log** and **/var/log/messages**
reveals extra information about this error.

```bash
[root@host ~]# tail /var/log/audit/audit.log
...output omitted...
type=AVC msg=audit(1392944135.482:429): avc: denied { getattr } for
  pid=1609 comm="httpd" path="/var/www/html/file3" dev="vda1" ino=8980981
  scontext=system_u:system_r:httpd_t:s0
  tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file
...output omitted...
[root@host ~]# tail /var/log/messages
...output omitted...
Feb 20 19:55:42 host setroubleshoot: SELinux is preventing /usr/sbin/httpd
  from getattr access on the file . For complete SELinux messages. run
  sealert -l 613ca624-248d-48a2-a7d9-d28f5bbe2763

```
Both log files indicate that an SELinux denial is the culprit. The **sealert** command that is part of
the output in **/var/log/messages** provides extra information, including a possible fix.

```bash
[root@host ~]# sealert -l 613ca624-248d-48a2-a7d9-d28f5bbe2763
SELinux is preventing /usr/sbin/httpd from getattr access on the file .

*****  Plugin catchall (100. confidence) suggests  **************************

If you believe that httpd should be allowed getattr access on the
file by default.
Then you should report this as a bug.
You can generate a local policy module to allow this access.
Do
allow this access for now by executing:
# grep httpd /var/log/audit/audit.log | audit2allow -M mypol
# semodule -i mypol.pp
Additional Information:
Source Context          system_u:system_r:httpd_t:s0
Target Context          unconfined_u:object_r:admin_home_t:s0
Target Objects            [ file ]
Source                  httpd
Source Path             /usr/sbin/httpd
Port                    <Unknown>
Host                    servera
Source RPM Packages     httpd-2.4.6-14.el7.x86_64
Target RPM Packages     
Policy RPM              selinux-policy-3.12.1-124.el7.noarch
Selinux Enabled         True
Policy Type             targeted
Enforcing Mode          Enforcing
Host Name               servera
Platform                Linux servera 3.10.0-84.el7.x86_64 #1
                        SMP Tue Feb 4 16:28:19 EST 2014 x86_64 x86_64
Alert Count             2
First Seen              2014-02-20 19:55:35 EST
Last Seen               2014-02-20 19:55:35 EST
Local ID                613ca624-248d-48a2-a7d9-d28f5bbe2763

Raw Audit Messages
type=AVC msg=audit(1392944135.482:429): avc: denied { getattr } for
  pid=1609 comm="httpd" path="/var/www/html/file3" dev="vda1" ino=8980981
  scontext=system_u:system_r:httpd_t:s0
  tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file

type=SYSCALL msg=audit(1392944135.482:429): arch=x86_64 syscall=lstat
  success=no exit=EACCES a0=7f9fed0edea8 a1=7fff7bffc770 a2=7fff7bffc770
  a3=0 items=0 ppid=1608 pid=1609 auid=4294967295 uid=48 gid=48 euid=48
  suid=48 fsuid=48 egid=48 sgid=48 fsgid=48 tty=(none) ses=4294967295
  comm=httpd exe=/usr/sbin/httpd subj=system_u:system_r:httpd_t:s0 key=(null)

Hash: httpd,httpd_t,admin_home_t,file,getattr
```
!!! info "NOTE"

    The **Raw Audit Messages** section reveals the target file that is the problem, **/
    var/www/html/file3**. Also, the target context, **tcontext**, does not look like
    it belongs with a web server. Use the **restorecon ****/var/www/html/file3**
    command to fix the file context. If there are other files that need to be adjusted,
    **restorecon** can recursively reset the context: **restorecon -R /var/www/**.

The **Raw Audit Messages** section of the **sealert** command contains information from **/var/
log/audit.log**. To search the **/var/log/audit.log** file use the **ausearch** command. The **-
m** searches on the message type. The **-ts** option searches based on time.

```bash
[root@host ~]# ausearch -m AVC -ts recent
----
time->Tue Apr 9 13:13:07 2019
type=PROCTITLE msg=audit(1554808387.778:4002):
  proctitle=2F7573722F7362696E2F6874747064002D44464F524547524F554E44
type=SYSCALL msg=audit(1554808387.778:4002): arch=c000003e syscall=49
  success=no exit=-13 a0=3 a1=55620b8c9280 a2=10 a3=7ffed967661c items=0
  ppid=1 pid=9340 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0
  sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="httpd" exe="/usr/sbin/httpd"
  subj=system_u:system_r:httpd_t:s0 key=(null)
type=AVC msg=audit(1554808387.778:4002): avc: denied { name_bind }
for pid=9340 comm="httpd" src=82 scontext=system_u:system_r:httpd_t:s0
tcontext=system_u:object_r:reserved_port_t:s0 tclass=tcp_socket permissive=0

```

#### WEB CONSOLE

If Web Console is installed it can also be used for troubleshooting SELinux issues. Log on to Web
Console and select **SELinux** from the menu on the left. The SELinux Policy window informs you of
the current enforce policy. Any issues are detailed in the **SELinux Access Control Errors** section.

Click on the **>** character to show error details. Click on **solution details** to show all details and
possible solution.

![SElinux policy solution in web console](https://cockpit-project.org/images/selinux-apply-this-solution.png)

Once the problem has been solved, the ***SELinux Access Control Errors*** section should no longer
show the error. If the message **No SELinux alerts** appears, then all issues have been fixed.

![SElinux policy in web console](https://access.redhat.com/webassets/avalon/d/Red_Hat_Enterprise_Linux-8-Configuring_basic_system_settings-en-US/images/f6c74a13352b57c4346b399a6ae8c8b8/cs_getting_started-selinux-on.png)

### SUMMARY

In this chapter, you learned:

- The **getenforce** and **setenforce** commands are used to manage the SELinux mode of a
system.

- The **semanage** command is used to manage SELinux policy rules. The **restorecon** command
applies the context defined by the policy.

- Booleans are switches that change the behavior of the SELinux policy. They can be enabled or
disabled and are used to tune the policy.

- The **sealert** displays useful information to help with SELinux troubleshooting.

### LAB

pdf - 151