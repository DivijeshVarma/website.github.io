---
title: "Managing Local Users And Groups"
---

### DESCRIBING USER AND GROUP CONCEPTS ###

#### WHAT IS A USER? ####

A *user* account is used to provide security boundaries between different people and programs that
can run commands.

Users have user names to identify them to human users and make them easier to work with.
Internally, the system distinguishes user accounts by the unique identification number assigned to
them, the user ID or UID. If a user account is used by humans, it will generally be assigned a secret
password that the user will use to prove that they are the actual authorized user when logging in.

User accounts are fundamental to system security. Every process (running program) on the system
runs as a particular user. Every file has a particular user as its owner. File ownership helps the
system enforce access control for users of the files. The user associated with a running process
determines the files and directories accessible to that process.

There are three main types of user account: the *superuser, system users*, and *regular users*.

- The superuser account is for administration of the system. The name of the superuser is root
and the account has UID 0. The superuser has full access to the system.

- The system has system user accounts which are used by processes that provide supporting
services. These processes, or daemons, usually do not need to run as the superuser. They are
assiged non-privileged accounts that allow them to secure their files and other resources from
each other and from regular users on the system. Users do not interactively log in using a system
user account.

- Most users have regular user accounts which they use for their day-to-day work. Like system

You can use the **id** command to show information about the currently logged-in user.

``` bash
[user01@host ~]$ id
uid=1000(user01) gid=1000(user01) groups=1000(user01)
 context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

To view basic information about another user, pass the username to the id command as an
argument.

``` bash
[user01@host]$ id user02
uid=1002(user02) gid=1001(user02) groups=1001(user02)
 context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

To view the owner of a file use the **ls -l** command. To view the owner of a directory use the **ls -
ld** command. In the following output, the third column shows the username.


``` bash
[user01@host ~]$ ls -l file1
-rw-rw-r--. 1 user01 user01 0 Feb  5 11:10 file1
[user01@host]$ ls -ld dir1
drwxrwxr-x. 2 user01 user01 6 Feb  5 11:10 dir1
```

To view process information, use the **ps** command. The default is to show only processes in the
current shell. Add the a option to view all processes with a terminal. To view the user associated
with a process, include the u option. In the following output, the first column shows the username.

``` bash
[user01@host]$ ps -au
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       777  0.0  0.0 225752  1496 tty1     Ss+  11:03   0:00 /sbin/agetty -o -
p -- \u --noclear tty1 linux
root       780  0.0  0.1 225392  2064 ttyS0    Ss+  11:03   0:00 /sbin/agetty -o -
p -- \u --keep-baud 115200,38400,9600
user01      1207  0.0  0.2 234044  5104 pts/0    Ss   11:09   0:00 -bash
user01      1319  0.0  0.2 266904  3876 pts/0    R+   11:33   0:00 ps au
```

The output of the preceding command displays users by name, but internally the operating system
uses the UIDs to track users. The mapping of usernames to UIDs is defined in databases of account
information. By default, systems use the **/etc/passwd** file to store information about local users.

Each line in the **/etc/passwd** file contains information about one user. It is divided up into seven
colon-separated fields. Here is an example of a line from **/etc/passwd**:

`user01:x:1000:1000:User One:/home/user01:/bin/bash`

> user01 --> Username for this user (user01).

> x --> The user's password used to be stored here in encrypted format. That has been moved to the
**/etc/shadow** file, which will be covered later. This field should always be x.

> 1000 --> The UID number for this user account (1000).

> 1000 --> The GID number for this user account's primary group (1000). Groups will be discussed later
in this section.

> User One --> The real name for this user (User One).

> /home/user01 --> The home directory for this user (**/home/user01**). This is the initial working directory when
the shell starts and contains the user's data and configuration settings.

> /bin/bash --> The default shell program for this user, which runs on login (**/bin/bash**). For a regular user,
this is normally the program that provides the user's command-line prompt. A system user
might use **/sbin/nologin** if interactive logins are not allowed for that user.


#### WHAT IS A GROUP? ####

A group is a collection of users that need to share access to files and other system resources.
Groups can be used to grant access to files to a set of users instead of just a single user.
Like users, groups have group names to make them easier to work with. Internally, the system
distinguishes groups by the unique identification number assigned to them, the group ID or GID.
The mapping of group names to GIDs is defined in databases of group account information. By
default, systems use the **/etc/group** file to store information about local groups.

Each line in the /etc/group file contains information about one group. Each group entry is
divided into four colon-separated fields. Here is an example of a line from /etc/group:

`group01:x:10000:user01,user02,user03`

> group01 --> Group name for this group (group01).

> x --> Obsolete group password field. This field should always be x.

> 10000 --> The GID number for this group (10000).

> user01 --> A list of users who are members of this group as a supplementary group (user01, user02,
user03). Primary (or default) and supplementary groups are discussed later in this section.


**Primary Groups and Supplementary Groups**

Every user has exactly one primary group. For local users, this is the group listed by GID number in
the */etc/passwd* file. By default, this is the group that will own new files created by the user.

Normally, when you create a new regular user, a new group with the same name as that user
is created. That group is used as the primary group for the new user, and that user is the
only member of this User Private Group. It turns out that this helps make management of file
permissions simpler, which will be discussed later in this course.

Users may also have supplementary groups. Membership in supplementary groups is determined
by the */etc/group* file. Users are granted access to files based on whether any of their
groups have access. It doesn't matter if the group or groups that have access are primary or
supplementary for the user.

For example, if the user user01 has a primary group user01 and supplementary groups wheel
and webadmin, then that user can read files readable by any of those three groups.

The **id** command can also be used to find out about group membership for a user.

``` bash
[user03@host ~]$ id
uid=1003(user03) gid=1003(user03) groups=1003(user03),10(wheel),10000(group01)
 context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

In the preceding example, user03 has the group user03 as their primary group (**gid**). The
**groups** item lists all groups for this user, and other than the primary group user03, the user has
groups wheel and group01 as supplementary groups.

### GAINING SUPERUSER ACCESS ###

#### THE SUPERUSER ####

Most operating systems have some sort of superuser, a user that has all power over the system.
In Red Hat Enterprise Linux this is the root user. This user has the power to override normal
privileges on the file system, and is used to manage and administer the system. To perform tasks
such as installing or removing software and to manage system files and directories, users must
escalate their privileges to the root user.

The root user only among normal users can control most devices, but there are a few exceptions.
For example, normal users can control removable devices, such as USB devices. Thus, normal users
can add and remove files and otherwise manage a removable device, but only root can manage
"fixed" hard drives by default.

This unlimited privilege, however, comes with responsibility. The root user has unlimited power
to damage the system: remove files and directories, remove user accounts, add back doors, and so
on. If the root user's account is compromised, someone else would have administrative control of
the system. Throughout this course, administrators are encouraged to log in as a normal user and
escalate privileges to root only when needed.

The root account on Linux is roughly equivalent to the local Administrator account on Microsoft
Windows. In Linux, most system administrators log in to the system as an unprivileged user and
use various tools to temporarily gain root privileges.

!!! danger "WARNING"

	One common practice on Microsoft Windows in the past was for the local
	Administrator user to log in directly to perform system administrator duties.
	Although this is possible on Linux, Red Hat recommends that system administrators
	do not log in directly as root. Instead, system administrators should log in as a
	normal user and use other mechanisms (**su, sudo**, or PolicyKit, for example) to
	temporarily gain superuser privileges.
	
	By logging in as the superuser, the entire desktop environment unnecessarily runs
	with administrative privileges. In that situation, any security vulnerability which
	would normally only compromise the user account has the potential to compromise
	the entire system.

#### SWITCHING USERS ####

The **su** command allows users to switch to a different user account. If you run **su** from a regular
user account, you will be prompted for the password of the account to which you want to switch.
When root runs **su**, you do not need to enter the user's password.

``` bash
[user01@host ~]$ su - user02
Password: 
[user02@host ~]$ 
```

If you omit the user name, the su or su - command attempts to switch to root by default.

``` bash
[user01@host ~]$ su -
Password: 
[root@host ~]# 
```

The command **su** starts a non-login shell, while the command **su -** (with the dash option) starts
a login shell. The main distinction between the two commands is that **su -** sets up the shell
environment as if it were a new login as that user, while **su** just starts a shell as that user, but uses
the original user's environment settings.

In most cases, administrators should run su - to get a shell with the target user's normal
environment settings. For more information, see the bash(1) man page.

!!! info "NOTE"

	The **su** command is most frequently used to get a command-line interface (shell
	prompt) which is running as another user, typically root. However, with the **-c**
	option, it can be used like the Windows utility runas to run an arbitrary program as
	another user. Run **info su** to view more details.


#### RUNNING COMMANDS WITH SUDO ####

In some cases, the root user's account may not have a valid password at all for security reasons.
In this case, users cannot log in to the system as root directly with a password, and **su** cannot be
used to get an interactive shell. One tool that can be used to get root access in this case is **sudo**.

Unlike **su, sudo** normally requires users to enter their own password for authentication, not
the password of the user account they are trying to access. That is, users who use sudo to
run commands as root do not need to know the root password. Instead, they use their own
passwords to authenticate access.

Additionally, **sudo** can be configured to allow specific users to run any command as some other
user, or only some commands as that user.

For example, when **sudo** is configured to allow the user01 user to run the command **usermod** as
root, user01 could run the following command to lock or unlock a user account:

``` bash
[user01@host ~]$ sudo usermod -L user02
[sudo] password for user01: 
[user01@host ~]$ su - user02
Password: 
su: Authentication failure
[user01@host ~]$ 
```

If a user tries to run a command as another user, and the **sudo** configuration does not permit it,
the command will be blocked, the attempt will be logged, and by default an email will be sent to the
root user.

``` bash
[user02@host ~]$ sudo tail /var/log/secure
[sudo] password for user02: 
user02 is not in the sudoers file.  This incident will be reported.
[user02@host ~]$ 
```

One additional benefit to using **sudo** is that all commands executed are logged by default to **/
var/log/secure**.

``` bash
[user01@host ~]$ sudo tail /var/log/secure
...output omitted...
Feb  6 20:45:46 host sudo[2577]:  user01 : TTY=pts/0 ; PWD=/home/user01 ;
 USER=root ; COMMAND=/sbin/usermod -L user02
...output omitted...
```

In Red Hat Enterprise Linux 7 and Red Hat Enterprise Linux 8, all members of the wheel group
can use sudo to run commands as any user, including root. The user is prompted for their own
password. This is a change from Red Hat Enterprise Linux 6 and earlier, where users who were
members of the wheel group did not get this administrative access by default.

!!! danger "WARNING"

	RHEL 6 did not grant the wheel group any special privileges by default. Sites that
	have been using this group for a non-standard purpose might be surprised when
	RHEL 7 and RHEL 8 automatically grants all members of wheel full **sudo** privileges.
	This could lead to unauthorized users getting administrative access to RHEL 7 and
	RHEL 8 systems.
	
	Historically, UNIX-like systems use membership in the wheel group to grant or
	control superuser access.

**Getting an Interactive Root Shell with Sudo**

If there is a nonadministrative user account on the system that can use *sudo* to run the *su*
command, you can run *sudo su -* from that account to get an interactive root user shell. This
works because *sudo* will run *su -* as root, and root does not need to enter a password to use
*su*.

Another way to access the root account with *sudo* is to use the *sudo -i* command. This will
switch to the root account and run that user's default shell (usually *bash*) and associated shell
login scripts. If you just want to run the shell, you can use the *sudo -s* command.

For example, an administrator might get an interactive shell as root on an AWS EC2 instance by
using SSH public-key authentication to log in as the normal user ec2-user, and then by running
*sudo -i* to get the root user's shell.

``` bash
[ec2-user@host ~]$ sudo -i
[sudo] password for ec2-user: 
[root@host ~]# 
```

The *sudo su -* command and *sudo -i* do not behave exactly the same. This will be discussed
briefly at the end of the section.

**Configuring Sudo**

The main configuration file for sudo is */etc/sudoers*. To avoid problems if multiple
administrators try to edit it at the same time, it should only be edited with the special *visudo*
command.

For example, the following line from the */etc/sudoers* file enables sudo access for members of
group *wheel*.

`%wheel        ALL=(ALL)       ALL`

In this line, *%wheel* is the user or group to whom the rule applies. A % specifies that this is a group,
group wheel. The *ALL=(ALL)* specifies that on any host that might have this file, wheel can run
any command. The final ALL specifies that wheel can run those commands as any user on the
system.

By default, */etc/sudoers* also includes the contents of any files in the */etc/sudoers.d*
directory as part of the configuration file. This allows an administrator to add sudo access for a
user simply by putting an appropriate file in that directory.

!!! info "NOTE"

	Using supplementary files under the **/etc/sudoers.d** directory is convenient
	and simple. You can enable or disable **sudo** access simply by copying a file into the
	directory or removing it from the directory.
	
	In this course, you will create and remove files in the **/etc/sudoers.d** directory to
	configure **sudo** access for users and groups.

To enable full **sudo** access for the user user01, you could create **/etc/sudoers.d/user01** with
the following content:

`user01  ALL=(ALL)  ALL`

To enable full **sudo** access for the group group01, you could create **/etc/sudoers.d/group01**
with the following content:

`%group01  ALL=(ALL)  ALL`

It is also possible to set up **sudo** to allow a user to run commands as another user without entering
their password:

`ansible  ALL=(ALL)  NOPASSWD:ALL`

While there are obvious security risks to granting this level of access to a user or group, it is
frequently used with cloud instances, virtual machines, and provisioning systems to help configure
servers. The account with this access must be carefully protected and might require SSH public-
key authentication in order for a user on a remote system to access it at all.

For example, the official AMI for Red Hat Enterprise Linux in the Amazon Web Services Marketplace
ships with the root and the ec2-user users' passwords locked. The ec2-user user account is
set up to allow remote interactive access through SSH public-key authentication. The user ec2-
user can also run any command as root without a password because the last line of the AMI's **/
etc/sudoers** file is set up as follows:

`ec2-user  ALL=(ALL)  NOPASSWD: ALL`

The requirement to enter a password for **sudo** can be re-enabled or other changes may be made to
tighten security as part of the process of configuring the system.

!!! info "NOTE"

	In this course, you will frequently see **sudo su -** used instead of **sudo -i**. Both
	commands work, but there are some subtle differences between them.
	
	The **sudo su -** command sets up the root environment exactly like a normal login
	because the **su -** command ignores the settings made by sudo and sets up the
	environment from scratch.
	
	The default configuration of the **sudo -i** command actually sets up some details of
	the root user's environment differently than a normal login. For example, it sets the
	PATH environment variable slightly differently. This affects where the shell will look
	to find commands.
	
	You can make **sudo -i** behave more like **su -** by editing **/etc/sudoers** with
	**visudo**. Find the line
	
	`Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin`
	
	and replace it with the following two lines:
	
	`Defaults      secure_path = /usr/local/bin:/usr/bin`
	
	`Defaults>root secure_path = /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin`
	
	For most purposes, this is not a major difference. However, for consistency of PATH
	settings on systems with the default **/etc/sudoers** file, the authors of this course
	mostly use **sudo su -** in examples.

### MANAGING LOCAL USER ACCOUNTS ###

#### MANAGING LOCAL USERS ####

A number of command-line tools can be used to manage local user accounts.

**Creating Users from the Command Line**

- The *useradd username* command creates a new user named username. It sets up the
user's home directory and account information, and creates a private group for the user named
username. At this point the account does not have a valid password set, and the user cannot log
in until a password is set.

- The *useradd --help* command displays the basic options that can be used to override the
defaults. In most cases, the same options can be used with the *usermod* command to modify an
existing user.

- Some defaults, such as the range of valid UID numbers and default password aging rules, are
read from the */etc/login.defs* file. Values in this file are only used when creating new users.
A change to this file does not affect existing users.

**Modifying Existing Users from the Command Line**

- The *usermod --help* command displays the basic options that can be used to modify an
account. Some common options include:

| USERMOD OPTIONS:          | USAGE                                                                                                                                                                       |
|:--------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **-c, --comment COMMENT** | Add the user's real name to the comment field.                                                                                                                              |
| **-g, --gid GROUP**       | Specify the primary group for the user account.                                                                                                                             |
| **-G, --groups GROUPS**   | Specify a comma-separated list of supplementary groups for the user account.                                                                                                |
| **-a, --append**          | Used with the -G option to add the supplementary groups to the user's current set of group memberships instead of replacing the set of supplementary groups with a new set. |
| **-d, --home HOME_DIR**   | Specify a particular home directory for the user account.                                                                                                                   |
| **-m, --move-home**       | Move the user's home directory to a new location. Must be used with the -d option.                                                                                          |
| **-s, --shell SHELL**     | Specify a particular login shell for the user account.                                                                                                                      |
| **-L, --lock**            | Lock the user account.                                                                                                                                                      |
| **-U, --unlock**          | Unlock the user account.                                                                                                                                                    |

**Deleting Users from the Command Line**

- The userdel username command removes the details of username from /etc/passwd, but
leaves the user's home directory intact.

- The userdel -r username command removes the details of username from /etc/passwd
and also deletes the user's home directory.

!!! danger "WARNING"

	When a user is removed with **userdel** without the **-r** option specified, the system
	will have files that are owned by an unassigned UID. This can also happen when a
	file, having a deleted user as its owner, exists outside that user's home directory.
	This situation can lead to information leakage and other security issues.
	
	In Red Hat Enterprise Linux 7 and Red Hat Enterprise Linux 8, the **useradd**
	command assigns new users the first free UID greater than or equal to 1000, unless
	you explicitly specify one using the **-u** option.
	
	This is how information leakage can occur. If the first free UID had been previously
	assigned to a user account which has since been removed from the system, the old
	user's UID will get reassigned to the new user, giving the new user ownership of the
	old user's remaining files.
	
	The following scenario demonstrates this situation.
	
	``` bash
	[root@host ~]# useradd user01
	[root@host ~]# ls -l /home
	drwx------. 3 user01  user01    74 Feb  4 15:22 user01
	[root@host ~]# userdel user01
	[root@host ~]# ls -l /home
	drwx------. 3    1000    1000   74 Feb  4 15:22 user01
	[root@host ~]# useradd user02
	[root@host ~]# ls -l /home
	drwx------. 3 user02     user02       74 Feb  4 15:23 user02
	drwx------. 3 user02     user02       74 Feb  4 15:22 user01
	```

	Notice that user02 now owns all files that user01 previously owned.
	
	Depending on the situation, one solution to this problem is to remove all unowned
	files from the system when the user that created them is deleted. Another solution
	is to manually assign the unowned files to a different user. The root user can use
	the **find / -nouser -o -nogroup** command to find all unowned files and
	directories.
	
**Setting Passwords from the Command Line**

- The **passwd username** command sets the initial password or changes the existing password of
username.

- The root user can set a password to any value. A message is displayed if the password does
not meet the minimum recommended criteria, but is followed by a prompt to retype the new
password and all tokens are updated successfully.

``` bash
[root@host ~]# passwd user01
Changing password for user user01.
New password: redhat
BAD PASSWORD: The password fails the dictionary check - it is based on a
 dictionary word
Retype new password: redhat
passwd: all authentication tokens updated successfully.
[root@host ~]# 
```

- A regular user must choose a password at least eight characters long and is also not based on a
dictionary word, the username, or the previous password.

**UID Ranges**

Specific UID numbers and ranges of numbers are used for specific purposes by Red Hat
Enterprise Linux.

- UID 0 is always assigned to the superuser account, **root**.

- UID 1-200 is a range of "system users" assigned statically to system processes by Red Hat.

- UID 201-999 is a range of "system users" used by system processes that do not own files on the
file system. They are typically assigned dynamically from the available pool when the software
that needs them is installed. Programs run as these "unprivileged" system users in order to limit
their access to only the resources they need to function.

• UID 1000+ is the range available for assignment to regular users.

!!! info "NOTE"

	Prior to RHEL 7, the convention was that UID 1-499 was used for system users and
	UID 500+ for regular users. Default ranges used by **useradd** and **groupadd** can be
	changed in the **/etc/login.defs** file.
	
### MANAGING LOCAL GROUP ACCOUNTS ###

#### MANAGING LOCAL GROUPS ####

A group must exist before a user can be added to that group. Several command-line tools are used
to manage local group accounts.

**Creating Groups from the Command Line**

- The *groupadd* command creates groups. Without options the *groupadd* command uses the
next available GID from the range specified in the */etc/login.defs* file while creating the
groups.

- The -g option specifies a particular GID for the group to use.

``` bash
[user01@host ~]$ sudo groupadd -g 10000 group01
[user01@host ~]$ tail /etc/group
...output omitted...
group01:x:10000:
```

!!! info "NOTE"

	Given the automatic creation of user private groups (GID 1000+), it is generally
	recommended to set aside a range of GIDs to be used for supplementary groups. A
	higher range will avoid a collision with a system group (GID 0-999).

- The **-r** option creates a system group using a GID from the range of valid system GIDs listed in
the **/etc/login.defs** file. The *SYSGIDMIN* and *SYSGIDMAX* configuration items in **/
etc/login.defs** define the range of system GIDs.

``` bash
[user01@host ~]$ sudo groupadd -r group02
[user01@host ~]$ tail /etc/group
...output omitted...
group01:x:10000:
group02:x:988:
```

**Modifying Existing Groups from the Command Line**

- The *groupmod* command changes the properties of an existing group. The -n option specifies a
new name for the group.

``` bash
[user01@host ~]$ sudo groupmod -n group0022 group02
[user01@host ~]$ tail /etc/group
...output omitted...
group0022:x:988:
```

Notice that the group name is updated to group0022 from group02.

- The **-g** option specifies a new GID.

``` bash
[user01@host ~]$ sudo groupmod -g 20000 group0022
[user01@host ~]$ tail /etc/group
...output omitted...
group0022:x:20000:
```

Notice that the GID is updated to 20000 from 988.

Deleting Groups from the Command Line

- The groupdel command removes groups.

``` bash
[user01@host ~]$ sudo groupdel group0022
```

!!! info "NOTE"

	You cannot remove a group if it is the primary group of any existing user. As with
	**userdel**, check all file systems to ensure that no files remain on the system that are
	owned by the group.

**Changing Group Membership from the Command Line**

- The membership of a group is controlled with user management. Use the *usermod -g*
command to change a user's primary group.

``` bash
[user01@host ~]$ id user02
uid=1006(user02) gid=1008(user02) groups=1008(user02)
[user01@host ~]$ sudo usermod -g group01 user02
[user01@host ~]$ id user02
uid=1006(user02) gid=10000(group01) groups=10000(group01)
```

- Use the *usermod -aG* command to add a user to a supplementary group.

``` bash
[user01@host ~]$ id user03
uid=1007(user03) gid=1009(user03) groups=1009(user03)
[user01@host ~]$ sudo usermod -aG group01 user03
[user01@host ~]$ id user03
uid=1007(user03) gid=1009(user03) groups=1009(user03),10000(group01)
```

!!! danger "IMPORTANT"

	The use of the **-a** option makes **usermod** function in append mode. Without **-a**, the
	user will be removed from any of their current supplementary groups that are not
	included in the **-G** option's list.

### MANAGING USER PASSWORDS OBJECTIVES ###

#### SHADOW PASSWORDS AND PASSWORD POLICY ####

At one time, encrypted passwords were stored in the world-readable **/etc/passwd** file. This was
thought to be reasonably secure until dictionary attacks on encrypted passwords became common.
At that point, the encrypted passwords were moved to a separate **/etc/shadow** file which is
readable only by root. This new file also allowed password aging and expiration features to be
implemented.

Like **/etc/passwd**, each user has a line in the **/etc/shadow** file. A sample line from **/etc/
shadow** with its nine colon-separated fields is shown below.

`user03:$6$CSsX...output omitted...:17933:0:99999:7:2:18113:`

> user03 --> Username of the account this password belongs to.

> $6$CSsX...output omitted... --> The *encrypted password* of the user. The format of encrypted passwords is discussed later in
this section.

> 17933 --> The day on which the password was last changed. This is set in days since 1970-01-01, and is
calculated in the UTC time zone.

> 0 --> The minimum number of days that have to elapse since the last password change before the
user can change it again.

> 99999 --> The maximum number of days that can pass without a password change before the password
expires. An empty field means it does not expire based on time since the last change.

> 7 --> Warning period. The user will be warned about an expiring password when they login for this
number of days before the deadline.

> 2 --> Inactivity period. Once the password has expired, it will still be accepted for login for this
many days. After this period has elapsed, the account will be locked.

> 18113 --> The day on which the password expires. This is set in days since 1970-01-01, and is calculated 
in the UTC time zone. An empty field means it does not expire on a particular date.

> :   --> The last field is usually empty and is reserved for future use.

**Format of an Encrypted Password**

The encrypted password field stores three pieces of information: the *hashing algorithm* used, the
*salt*, and the *encrypted hash*. Each piece of information is delimited by the *$* sign.

`$6$CSsXcYG1L/4ZfHr/$2W6evvJahUfzfHpc9X.45Jc6H30E...output omitted...`

> 6 --> The hashing algorithm used for this password. The number 6 indicates it is a SHA-512 hash,
which is the default in Red Hat Enterprise Linux 8. A 1 would indicate MD5, a 5 SHA-256.

> CSsXcYG1L/4ZfHr/ --> The salt used to encrypt the password. This is originally chosen at random.

> 2W6evvJahUfzfHpc9X.45Jc6H30E...output omitted... -->  The encrypted hash of the user's password. The salt and the unencrypted password are combined and encrypted to generate the encrypted hash of the password.

The use of a salt prevents two users with the same password from having identical entries in the /
**etc/shadow** file. For example, even if user01 and user02 both use redhat as their passwords,
their encrypted passwords in **/etc/shadow** will be different if their salts are different.

**Password Verification**

When a user tries to log in, the system looks up the entry for the user in **/etc/shadow**, combines
the salt for the user with the unencrypted password that was typed in, and encrypts them using the
hashing algorithm specified. If the result matches the encrypted hash, the user typed in the right
password. If the result does not match the encrypted hash, the user typed in the wrong password
and the login attempt fails. This method allows the system to determine if the user typed in the
correct password without storing that password in a form usable for logging in.

#### CONFIGURING PASSWORD AGING ####

The following diagram relates the relevant password aging parameters, which can be adjusted
using the **chage** command to implement a password aging policy.

![password](https://computingforgeeks.com/wp-content/uploads/2020/02/linux-password-aging.png "password")

``` bash
[user01@host ~]$ sudo chage -m 0 -M 90 -W 7 -I 14 user03
```

The preceding **chage** command uses the **-m, -M, -W**, and **-I** options to set the minimum age,
maximum age, warning period, and inactivity period of the user's password, respectively.

The **chage -d 0 user03** command forces the user03 user to update its password on the next
login.

The **chage -l user03** command displays the password aging details of user03.

The **chage -E 2019-08-05 user03** command causes the user03 user's account to expire on
2019-08-05 (in YYYY-MM-DD format).

!!! info "NOTE"

	The **date** command can be used to calculate a date in the future. The **-u** option
	reports the time in UTC.
	
	``` bash
	[user01@host ~]$ date -d "+45 days" -u
	Thu May 23 17:01:20 UTC 2019
	```

Edit the password aging configuration items in the **/etc/login.defs** file to set the default
password aging policies. The *PASSMAXDAYS* sets the default maximum age of the password. The
*PASSMINDAYS* sets the default minimum age of the password. The *PASSWARNAGE* sets the
default warning period of the password. Any change in the default password aging policies will be
effective for new users only. The existing users will continue to use the old password aging settings
rather than the new ones.

### RESTRICTING ACCESS ###

You can use the **chage** command to set account expiration dates. When that date is reached, the
user cannot log in to the system interactively. The **usermod** command can lock an account with
the **-L** option.

``` bash
[user01@host ~]$ sudo usermod -L user03
[user01@host ~]$ su - user03
Password: redhat
su: Authentication failure
```

If a user leaves the company, the administrator may lock and expire an account with a single
**usermod** command. The date must be given as the number of days since 1970-01-01, or in the
YYYY-MM-DD format.

``` bash
[user01@host ~]$ sudo usermod -L -e 2019-10-05 user03
```

The preceding **usermod** command uses the **-e** option to set the account expiry date for the given
user account. The **-L** option locks the user's password.

Locking the account prevents the user from authenticating with a password to the system. It is
the recommended method of preventing access to an account by an employee who has left the
company. If the employee returns, the account can later be unlocked with **usermod -U**. If the
account was also expired, be sure to also change the expiration date.

**The nologin Shell**

The *nologin* shell acts as a replacement shell for the user accounts not intended to interactively
log into the system. It is wise from the security standpoint to disable the user account from logging
into the system when the user acount serves a responsibility that does not require the user to log
into the system. For example, a mail server may require an account to store mail and a password
for the user to authenticate with a mail client used to retrieve mail. That user does not need to log
directly into the system.

A common solution to this situation is to set the user's login shell to */sbin/nologin*. If the user
attempts to log in to the system directly, the *nologin* shell closes the connection.

``` bash
[user01@host ~]$ usermod -s /sbin/nologin user03
[user01@host ~]$ su - user03
Last login: Wed Feb  6 17:03:06 IST 2019 on pts/0
This account is currently not available.
```

!!! danger "IMPORTANT"

	The **nologin** shell prevents interactive use of the system, but does not prevent
	all access. Users might be able to authenticate and upload or retrieve files through
	applications such as web applications, file transfer programs, or mail readers if they
	use the user's password for authentication.

### SUMMARY ###

In this chapter, you learned:

- There are three main types of user account: the superuser, system users, and regular users.

- A user must have a primary group and may be a member of one or more supplementary groups.

- The three critical files containing user and group information are **/etc/passwd, /etc/group**,
and **/etc/shadow**.

- The **su** and **sudo** commands can be used to run commands as the superuser.

- The **useradd, usermod**, and **userdel** commands can be used to manage users.

- The **groupadd, groupmod**, and **groupdel** commands can be used to manage groups.

- The **chage** command can be used to configure and view password expiration settings for users.

### LAB

pdf - 221