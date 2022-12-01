---
title: "Controlling Access To Files"
---

### INTERPRETING LINUX FILE SYSTEM PERMISSIONS ###

#### LINUX FILE-SYSTEM PERMISSIONS ####

*File permissions* control access to files. The Linux file permissions system is simple but flexible,
which makes it easy to understand and apply, yet still able to handle most normal permission cases
easily.

Files have three categories of user to which permissions apply. The file is owned by a user, normally
the one who created the file. The file is also owned by a single group, usually the primary group
of the user who created the file, but this can be changed. Different permissions can be set for the
owning user, the owning group, and for all other users on the system that are not the user or a
member of the owning group.

The most specific permissions take precedence. User permissions override group permissions,
which override other permissions.

In Figure, joshua is a member of the groups joshua and web, and allison is a member of
allison, wheel, and web. When joshua and allison need to collaborate, the files should be
associated with the group web and group permissions should allow the desired access.

![Example group membership to facilitate collaboration](https://www.linuxprobe.com/links/RH124/images/perms/usersandgroups.png "Example group membership to facilitate collaboration")

Three categories of permissions apply: read, write, and execute. The following table explains how
these permissions affect access to files and directories.

**Effects of Permissions on Files and Directories**

| PERMISSION  | EFFECT ON FILES                      | EFFECT ON DIRECTORIES                                                                                                                                                     |
|:-----------:|:-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| r (read)    | Contents of the file can be read.    | Contents of the directory (the filenames) can be listed.                                                                                                                  |
| w (write)   | Contents of the file can be changed. | Any file in the directory can be created or deleted.                                                                                                                      |
| x (execute) | Files can be executed as commands.   | Contents of the directory can be accessed. (You can change into the directory, read information about its files, and access its files if the files'permissions allow it.) |
|             |                                      |                                                                                                                                                                           |

Note that users normally have both read and execute permissions on read-only directories so that
they can list the directory and have full read-only access to its contents. If a user only has read
access on a directory, the names of the files in it can be listed, but no other information, including
permissions or time stamps, are available, nor can they be accessed. If a user only has execute
access on a directory, they cannot list the names of the files in the directory, but if they already
know the name of a file that they have permission to read, then they can access the contents of
that file by explicitly specifying the file name.

A file may be removed by anyone who has write permission to the directory in which the file
resides, regardless of the ownership or permissions on the file itself. This can be overridden with a
special permission, the *sticky bit*, discussed later in this chapter.

!!! info "NOTE"

	Linux file permissions work differently than the permissions system used by the
	NTFS file system for Microsoft Windows.

	On Linux, permissions apply only to the file or directory on which they are set. That
	is, permissions on a directory are not inherited automatically by the subdirectories
	and files within it. However, permissions on a directory can block access to the
	contents of the directory depending on how restrictive they are.

	The read permission on a directory in Linux is roughly equivalent to List folder
	contents in Windows.

	The write permission on a directory in Linux is equivalent to Modify in Windows; it
	implies the ability to delete files and subdirectories. In Linux, if write and the sticky
	bit are both set on a directory, then only the user that owns a file or subdirectory
	in the directory may delete it, which is close to the behavior of the Windows Write
	permission.

	The root user on Linux has the equivalent of the Windows Full Control permission
	on all files. However, root may still have access restricted by the system's SELinux
	policy and the security context of the process and files in question. SELinux will be
	discussed in a later course.

### VIEWING FILE AND DIRECTORY PERMISSIONS AND OWNERSHIP ###

The **-l** option of the **ls** command shows more detailed information about file permissions and
ownership:

``` bash
[user@host~]$ ls -l test
-rw-rw-r--. 1 student student 0 Feb  8 17:36 test
```

You can use the **-d** option to to show detailed information about a directory itself, and not its
contents.

``` bash
[user@host ~]$ ls -ld /home
drwxr-xr-x. 5 root root 4096 Jan 31 22:00 /home
```

The first character of the long listing is the file type. You interpret it like this:

- `- is a regular file.`

- `d is a directory.`

- `l is a soft link.`

- `Other characters represent hardware devices (b and c) or other special-purpose files (p and s).`

The next nine characters are the file permissions. These are in three sets of three characters:
permissions that apply to the user that owns the file, the group that owns the file, and all other
users. If the set shows **rwx**, that category has all three permissions, read, write, and execute. If a
letter has been replaced by -, then that category does not have that permission.

After the link count, the first name specifies the user that owns the file, and the second name the
group that owns the file.

So in the example above, the permissions for user student are specified by the first set of three
characters. User student has read and write on test, but not execute.

Group student is specified by the second set of three characters: it also has read and write on
test, but not execute.

Any other user's permissions are specified by the third set of three characters: they only have read
permission on test.

The most specific set of permissions apply. So if user student has different permissions than
group student, and user student is also a member of that group, then the user permissions will
be the ones that apply.

### EXAMPLES OF PERMISSION EFFECTS ###

The following examples will help illustrate how file permissions interact. For these examples, we
have four users with the following group memberships:

| USER        | GROUP MEMBERSHIPS      |
|:-----------:|:-----------------------|
| operator1   | operator1, consultant1 |
| database1   | database1, consultant1 |
| database2   | database2, operator2   |
| contractor1 | contractor1, operator2 |

Those users will be working with files in the **dir** directory. This is a long listing of the files in that
directory:

``` bash
[database1@host dir]$ ls -la
total 24
drwxrwxr-x.  2 database1 consultant1   4096 Apr  4 10:23 .
drwxr-xr-x. 10 root      root          4096 Apr  1 17:34 ..
-rw-rw-r--.  1 operator1 operator1     1024 Apr  4 11:02 lfile1
-rw-r--rw-.  1 operator1 consultant1   3144 Apr  4 11:02 lfile2
-rw-rw-r--.  1 database1 consultant1  10234 Apr  4 10:14 rfile1
-rw-r-----.  1 database1 consultant1   2048 Apr  4 10:18 rfile2
```

The **-a** option shows the permissions of hidden files, including the special files used to represent
the directory and its parent. In this example, . reflects the permissions of **dir** itself, and .. the
permissions of its parent directory.

What are the permissions of **rfile1**? The user that owns the file (database1) has read and write
but not execute. The group that owns the file (consultant1) has read and write but not execute.
All other users have read but not write or execute.

The following table explores some of the effects of this set of permissions for these users:

| EFFECT                                                                                                                                | WHY IS THIS TRUE?                                                                                                                                                                                                                |
|:--------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| The user operator1 can change the contents of **rfile1**.                                                                             | User operator1 is a member of the consultant1 group, and that group has both read and write permissions on **rfile1**.                                                                                                           |
| The user database1 can view and modify the contents of **rfile2**.                                                                        | User database1 owns the file and has both read and write access to **rfile2**.                                                                                                                                                       |
| The user operator1 can view but not modify the contents of **rfile2** (without deleting it and recreating it).                            | User operator1 is a member of the consultant1 group, and that group only has read access to **rfile2**.                                                                                                                              |
| The users database2 and contractor1 do not have any access to the contents of **rfile2**.                                                 | **other** permissions apply to users database2 and contractor1, and those permissions do not include read or write permission.                                                                                                       |
| operator1 is the only user who can change the contents of **lfile1** (without deleting it and recreating it).                             | User and group operator1 have write permission on the file, other users do not. But the only member of group operator1 is user operator1.                                                                                        |
| The user database2 can change the contents of **lfile2**.                                                                                 | User database2 is not the user that owns the file and is not in group consultant1, so **other** permissions apply. Those grant write permission.                                                                                     |
| The user database1 can view the contents of **lfile2**, but cannot modify the contents of **lfile2** (without deleting it and recreating it). | User database1 is a member of the group consultant1, and that group only has read permissions on **lfile2**. Even though **other** has write permission, the group permissions take precedence.                                          |
| The user database1 can delete **lfile1** and **lfile2**.                                                                                      | User database1 has write permissions on the directory containing both files (shown by .), and therefore can delete any file in that directory. This is true even if database1 does not have write permission on the file itself. |
|                                                                                                                                      


### MANAGING FILE SYSTEM PERMISSIONS FROM THE COMMAND LINE ###

#### CHANGING FILE AND DIRECTORY PERMISSIONS ####

The command used to change permissions from the command line is **chmod**, which means "change
mode" (permissions are also called the mode of a file). The **chmod** command takes a permission
instruction followed by a list of files or directories to change. The permission instruction can be
issued either symbolically (the symbolic method) or numerically (the numeric method).

**Changing Permissions with the Symbolic Method**

`chmod WhoWhatWhich file|directory`

- Who is u, g, o, a (for user, group, other, all)

- What is +, -, = (for add, remove, set exactly)

- Which is r, w, x (for read, write, execute)

The *symbolic* method of changing file permissions uses letters to represent the different groups of
permissions: u for user, g for group, o for other, and a for all.

With the symbolic method, it is not necessary to set a complete new group of permissions. Instead,
you can change one or more of the existing permissions. Use + or - to add or remove permissions,
respectively, or use = to replace the entire set for a group of permissions.

The permissions themselves are represented by a single letter: r for read, w for write, and x for
execute. When using chmod to change permissions with the symbolic method, using a capital X as
the permission flag will add execute permission only if the file is a directory or already has execute
set for user, group, or other.

!!! info "NOTE"

	The **chmod** command supports the -R option to recursively set permissions on
	the files in an entire directory tree. When using the -R option, it can be useful to
	set permissions symbolically using the X option. This allows the execute (search)
	permission to be set on directories so that their contents can be accessed, without
	changing permissions on most files. Be cautious with the X option, however, because
	if a file has any execute permission set, X will set the specified execute permission on
	that file as well. For example, the following command recursively sets read and write
	access on **demodir** and all its children for their group owner, but only applies group
	execute permissions to directories and files that already have execute set for user,
	group, or other.
	
	``` bash
	[root@host opt]# chmod -R g+rwX demodir
	```

**Examples**

- Remove read and write permission for group and other on file1:

``` bash
[user@host ~]$ chmod go-rw file1
```

- Add execute permission for everyone on file2:

``` bash
[user@host ~]$ chmod a+x file2
```

**Changing Permissions with the Numeric Method**

In the example below the # character represents a digit.

`chmod ### file|directory`

- Each digit represents permissions for an access level: user, group, other.

- The digit is calculated by adding together numbers for each permission you want to add, 4 for
read, 2 for write, and 1 for execute.

Using the numeric method, permissions are represented by a 3-digit (or 4-digit, when setting
advanced permissions) octal number. A single octal digit can represent any single value from 0-7.

In the 3-digit octal (numeric) representation of permissions, each digit stands for one access level,
from left to right: user, group, and other. To determine each digit:

1. Start with 0.

2. If the read permission should be present for this access level, add 4.

3. If the write permission should be present, add 2.

4. If the execute permission should be present, add 1.

Examine the permissions -rwxr-x---. For the user, rwx is calculated as 4+2+1=7. For the group,
r-x is calculated as 4+0+1=5, and for other users, --- is represented with 0. Putting these three
together, the numeric representation of those permissions is 750.

This calculation can also be performed in the opposite direction. Look at the permissions 640. For
the user permissions, 6 represents read (4) and write (2), which displays as rw-. For the group
part, 4 only includes read (4) and displays as r--. The 0 for other provides no permissions (---)
and the final set of symbolic permissions for this file is -rw-r-----.

Experienced administrators often use numeric permissions because they are shorter to type and
pronounce, while still giving full control over all permissions.

**Examples**

- Set read and write permissions for user, read permission for group and other, on samplefile:

``` bash
[user@host ~]$ chmod 644 samplefile
```

- Set read, write, and execute permissions for user, read and execute permissions for group, and no
permission for other on *sampledir*:

``` bash
[user@host ~]$ chmod 750 sampledir
```

#### CHANGING FILE AND DIRECTORY USER OR GROUP OWNERSHIP ####

A newly created file is owned by the user who creates that file. By default, new files have a group
ownership that is the primary group of the user creating the file. In Red Hat Enterprise Linux, a
user's primary group is usually a private group with only that user as a member. To grant access to
a file based on group membership, the group that owns the file may need to be changed.

Only root can change the user that owns a file. Group ownership, however, can be set by root or
by the file's owner. root can grant file ownership to any group, but regular users can make a group
the owner of a file only if they are a member of that group.

File ownership can be changed with the **chown** (change owner) command. For example, to grant
ownership of the test_file file to the student user, use the following command:

``` bash
[root@host ~]# chown student test_file
```

chown can be used with the -R option to recursively change the ownership of an entire directory
tree. The following command grants ownership of test_dir and all files and subdirectories within
it to student:

``` bash
[root@host ~]# chown -R student test_dir
```

The **chown** command can also be used to change group ownership of a file by preceding the
group name with a colon (:). For example, the following command changes the group test_dir to
admins:

``` bash
[root@host ~]# chown :admins test_dir
```

The **chown** command can also be used to change both owner and group at the same time by using
the owner:group syntax. For example, to change the ownership of test_dir to visitor and the
group to guests, use the following command:

``` bash
[root@host ~]# chown visitor:guests test_dir
```

Instead of using **chown**, some users change the group ownership by using the **chgrp** command.
This command works just like **chown**, except that it is only used to change group ownership and the
colon (:) before the group name is not required.

!!! danger "IMPORTANT"

	You may encounter examples of **chown** commands using an alternative syntax that
	separates owner and group with a period instead of a colon:
	
	``` bash
	[root@host ~]# chown owner.group filename
	```

	You should not use this syntax. Always use a colon.
	
	A period is a valid character in a user name, but a colon is not. If the user
	enoch.root, the user enoch, and the group root exist on the system, the result
	of **chown enoch.root filename** will be to have **filename** owned by the user
	enoch.root. You may have been trying to set the file ownership to the user enoch
	and group root. This can be confusing.
	
	If you always use the **chown** colon syntax when setting the user and group at the
	same time, the results are always easy to predict.

### MANAGING DEFAULT PERMISSIONS AND FILE ACCESS ###

#### SPECIAL PERMISSIONS ####

*Special permissions* constitute a fourth permission type in addition to the basic user, group, and
other types. As the name implies, these permissions provide additional access-related features
over and above what the basic permission types allow. This section details the impact of special
permissions, summarized in the table below.

**Effects of Special Permissions on Files and Directories**

| SPECIAL PERMISSION | EFFECT ON FILES                                                               | EFFECT ON DIRECTORIES                                                                                                                          |
|:------------------:|:------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| u+s (suid)         | File executes as the user that owns the file, not the user that ran the file. | No effect.                                                                                                                                     |
| g+s (sgid)         | File executes as the group that owns the file.                                | Files newly created in the directory have their group owner set to match the group owner of the directory.                                     |
| o+t (sticky)       | No effect.                                                                    | Users with write access to the directory can only remove files that they own; they cannot remove or force saves to files owned by other users. |
|                                                                                                              |
	
The setuid permission on an executable file means that commands run as the user owning the file,
not as the user that ran the command. One example is the **passwd** command:

``` bash
[user@host ~]$ ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 35504 Jul 16  2010 /usr/bin/passwd
```

In a long listing, you can identify the setuid permissions by a lowercase s where you would
normally expect the x (owner execute permissions) to be. If the owner does not have execute
permissions, this is replaced by an uppercase S.

The special permission setgid on a directory means that files created in the directory inherit
their group ownership from the directory, rather than inheriting it from the creating user. This is
commonly used on group collaborative directories to automatically change a file from the default
private group to the shared group, or if files in a directory should be always owned by a specific
group. An example of this is the **/run/log/journal** directory:

``` bash
[user@host ~]$ ls -ld /run/log/journal
drwxr-sr-x. 3 root systemd-journal 60 May 18 09:15 /run/log/journal
```

If setgid is set on an executable file, commands run as the group that owns that file, not as the user
that ran the command, in a similar way to setuid works. One example is the locate command:

``` bash
[user@host ~]$ ls -ld /usr/bin/locate
-rwx--s--x. 1 root slocate 47128 Aug 12 17:17 /usr/bin/locate
```

In a long listing, you can identify the setgid permissions by a lowercase s where you would
normally expect the x (group execute permissions) to be. If the group does not have execute
permissions, this is replaced by an uppercase S.

Lastly, the sticky bit for a directory sets a special restriction on deletion of files. Only the owner of
the file (and root) can delete files within the directory. An example is /tmp:

``` bash
[user@host ~]$ ls -ld /tmp
drwxrwxrwt. 39 root root 4096 Feb  8 20:52 /tmp
```

In a long listing, you can identify the sticky permissions by a lowercase t where you would normally
expect the x (other execute permissions) to be. If other does not have execute permissions, this is
replaced by an uppercase T.

**Setting Special Permissions**

- Symbolically: setuid = u+s; setgid = g+s; sticky = o+t

- Numerically (fourth preceding digit): setuid = 4; setgid = 2; sticky = 1

**Examples**

- Add the setgid bit on directory:

``` bash
[user@host ~]# chmod g+s directory
```

- Set the setgid bit and add read/write/execute permissions for user and group, with no access for
others, on directory:

``` bash
[user@host ~]# chmod 2770 directory
```

### DEFAULT FILE PERMISSIONS ###

When you create a new file or directory, it is assigned initial permissions. There are two things that
affect these initial permissions. The first is whether you are creating a regular file or a directory.
The second is the current **umask**.

If you create a new directory, the operating system starts by assigning it octal permissions 0777
(**drwxrwxrwx**). If you create a new regular file, the operating system assignes it octal permissions
0666 (**-rw-rw-rw-**). You always have to explicitly add execute permission to a regular file. This
makes it harder for an attacker to compromise a network service so that it creates a new file and
immediately executes it as a program.

However, the shell session will also set a umask to further restrict the permissions that are initially
set. This is an octal bitmask used to clear the permissions of new files and directories created by
a process. If a bit is set in the umask, then the corresponding permission is cleared on new files.
For example, the umask 0002 clears the write bit for other users. The leading zeros indicate the
special, user, and group permissions are not cleared. A umask of 0077 clears all the group and
other permissions of newly created files.

The **umask** command without arguments will display the current value of the shell's umask:

``` bash
[user@host ~]$ umask
0002
```

Use the **umask** command with a single numeric argument to change the umask of the current shell.
The numeric argument should be an octal value corresponding to the new umask value. You can
omit any leading zeros in the umask.

The system's default umask values for Bash shell users are defined in the **/etc/profile** and **/
etc/bashrc** files. Users can override the system defaults in the **.bash_profile** and **.bashrc**
files in their home directories.

**umask Example**

The following example explains how the umask affects the permissions of files and directories.
Look at the default umask permissions for both files and directories in the current shell. The owner
and group both have read and write permission on files, and other is set to read. The owner and
group both have read, write, and execute permissions on directories. The only permission for other
is read.

``` bash
[user@host ~]$ umask
0002
[user@host ~]$ touch default
[user@host ~]$ ls -l default.txt
-rw-rw-r--. 1 user user 0 May  9 01:54 default.txt
[user@host ~]$ mkdir default
[user@host ~]$ ls -ld default
drwxrwxr-x. 2 user user 0 May  9 01:54 default 
```

By setting the umask value to 0, the file permissions for other change from read to read and write.
The directory permissions for other changes from read and execute to read, write, and execute.

``` bash
[user@host ~]$ umask 0
[user@host ~]$ touch zero.txt
[user@host ~]$ ls -l zero.txt
-rw-rw-rw-. 1 user user 0 May  9 01:54 zero.txt
[user@host ~]$ mkdir zero
[user@host ~]$ ls -ld zero
drwxrwxrwx. 2 user user 0 May  9 01:54 zero 
```

To mask all file and directory permissions for other, set the umask value to 007.

``` bash
[user@host ~]$ umask 007
[user@host ~]$ touch seven.txt
[user@host ~]$ ls -l seven.txt
-rw-rw----. 1 user user 0 May  9 01:55 seven.txt
[user@host ~]$ mkdir seven
[user@host ~]$ ls -ld seven
drwxrwx---. 2 user user 0 May  9 01:54 seven
```

A umask of 027 ensures that new files have read and write permissions for user and read
permission for group. New directories have read and write access for group and no permissions for
other.

``` bash
[user@host ~]$ umask 027
[user@host ~]$ touch two-seven.txt
[user@host ~]$ ls -l two-seven.txt
-rw-r-----. 1 user user 0 May  9 01:55 two-seven.txt
[user@host ~]$ mkdir two-seven
[user@host ~]$ ls -ld two-seven
drwxr-x---. 2 user user 0 May  9 01:54 two-seven 
```

The default umask for users is set by the shell startup scripts. By default, if your account's UID is
200 or more and your username and primary group name are the same, you will be assigned a
umask of 002. Otherwise, your umask will be 022.

As root, you can change this by adding a shell startup script named **/etc/profile.d/local-
umask.sh** that looks something like the output in this example:

``` bash
[root@host ~]# cat /etc/profile.d/local-umask.sh
# Overrides default umask configuration
if [ $UID -gt 199 ] && [ "`id -gn`" = "`id -un`" ]; then
    umask 007
else
    umask 022
fi
```

The preceding example will set the umask to 007 for users with a UID greater than 199 and with a
username and primary group name that match, and to 022 for everyone else. If you just wanted to
set the umask for everyone to 022, you could create that file with just the following content:

`# Overrides default umask configuration`

`umask 022`

To ensure that global umask changes take effect you must log out of the shell and log back in. Until
that time the umask configured in the current shell is still in effect.

### SUMMARY ###

In this chapter, you learned:

- Files have three categories to which permissions apply. A file is owned by a user, a single
group, and other users. The most specific permission applies. User permissions override group
permissions and group permissions override other permissions.

- The **ls** command with the **-l** option expands the file listing to include both the file permissions
and ownership.

- The **chmod** command changes file permissions from the command line. There are two methods
to represent permissions, symbolic (letters) and numeric (digits).

- The **chown** command changes file ownership. The **-R** option recursively changes the ownership
of a directory tree.

- The **umask** command without arguments displays the current umask value of the shell. Every
process on the system has a umask. The default umask values for Bash are defined in the **/etc/
profile** and **/etc/bashrc** files.

### LAB

pdf - 254