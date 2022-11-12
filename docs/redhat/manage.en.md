---
title: "Managing Files From The Command Line"
---

### DESCRIBING LINUX FILE SYSTEM HIERARCHY CONCEPTS ###

#### THE FILE-SYSTEM HIERARCHY ####

All files on a Linux system are stored on file systems, which are organized into a single inverted
tree of directories, known as a file-system hierarchy. This tree is inverted because the root of the
tree is said to be at the top of the hierarchy, and the branches of directories and subdirectories
stretch below the root.

   ![FileSystem](https://1.bp.blogspot.com/-na02ATcbQ-4/XyTlpyPsnOI/AAAAAAAABtc/cO29eDL5uskloIns66P5xUOyamM4oMXvACLcBGAsYHQ/s1600/ezgif-4-1fbe035f5efa.jpg)

The / directory is the root directory at the top of the file-system hierarchy. The / character is also
used as a directory separator in file names. For example, if **etc** is a subdirectory of the / directory,
you could refer to that directory as /etc. Likewise, if the /etc directory contained a file named
issue, you could refer to that file as **/etc/issue**.

Subdirectories of / are used for standardized purposes to organize files by type and purpose. This
makes it easier to find files. For example, in the root directory, the subdirectory **/boot** is used for
storing files needed to boot the system.

!!! info "NOTE"

	The following terms help to describe file-system directory contents:
	
	- static content remains unchanged until explicitly edited or reconfigured.
	
	- dynamic or variable content may be modified or appended by active processes.
	
	- persistent content remains after a reboot, like configuration settings.
	
	- runtime content is process- or system-specific content that is deleted by a reboot.

The following table lists some of the most important directories on the system by name and
purpose.

**Important Red Hat Enterprise Linux Directories**

**/usr** 

Installed software, shared libraries, include files, and read-only program data.
Important subdirectories include:

- */usr/bin*: User commands.

- */usr/sbin*: System administration commands.

- */usr/local*: Locally customized software.


**/etc**

Configuration files specific to this system.

**/var**

Variable data specific to this system that should persist between boots. Files
that dynamically change, such as databases, cache directories, log files,
printer-spooled documents, and website content may be found under */var*.

**/run**

Runtime data for processes started since the last boot. This includes process
ID files and lock files, among other things. The contents of this directory are
recreated on reboot. This directory consolidates */var/run* and */var/lock*
from earlier versions of Red Hat Enterprise Linux.

**/home**

Home directories are where regular users store their personal data and
configuration files.

**/root**

Home directory for the administrative superuser, root.

**/tmp**

A world-writable space for temporary files. Files which have not been
accessed, changed, or modified for 10 days are deleted from this directory
automatically. Another temporary directory exists, */var/tmp*, in which files
that have not been accessed, changed, or modified in more than 30 days are
deleted automatically.

**/boot**

Files needed in order to start the boot process.

**/dev**

Contains special *device files* that are used by the system to access hardware.

!!! danger "IMPORTANT"

	In Red Hat Enterprise Linux 7 and later, four older directories in / have identical
	contents to their counterparts located in **/usr**:
	
	- /bin and /usr/bin
	
	- /sbin and /usr/sbin
	
	- /lib and /usr/lib
	
	- /lib64 and /usr/lib64
	
	In earlier versions of Red Hat Enterprise Linux, these were distinct directories
	containing different sets of files.
	In Red Hat Enterprise Linux 7 and later, the directories in / are symbolic links to the
	matching directories in **/usr**.


### SPECIFYING FILES BY NAME ###

#### ABSOLUTE PATHS AND RELATIVE PATHS ####

The *path* of a file or directory specifies its unique file system location. Following a file path
traverses one or more named subdirectories, delimited by a forward slash (/), until the destination
is reached. Directories, also called folders, contain other files and other subdirectories. They can be
referenced in the same manner as files.


!!! danger "IMPORTANT"

	A space character is acceptable as part of a Linux file name. However, spaces are
	also used by the shell to separate options and arguments on the command line. If
	you enter a command that includes a file that has a space in its name, the shell can
	misinterpret the command and assume that you want to start a new file name or
	other argument at the space.
	
	It is possible to avoid this by putting file names in quotes. However, if you do not
	need to use spaces in file names, it can be simpler to simply avoid using them.

**Absolute Paths**

An absolute path is a fully qualified name, specifying the files exact location in the file system
hierarchy. It begins at the root (/) directory and specifies each subdirectory that must be traversed
to reach the specific file. Every file in a file system has a unique absolute path name, recognized
with a simple rule: A path name with a forward slash (/) as the first character is an absolute
path name. For example, the absolute path name for the system message log file is */var/log/messages*.

Absolute path names can be long to type, so files may also be located relative to the
current working directory for your shell prompt.

**The Current Working Directory and Relative Paths**

When a user logs in and opens a command window, the initial location is normally the user's home
directory. System processes also have an initial directory. Users and processes navigate to other
directories as needed; the terms working directory or current working directory refer to their current
location.

Like an absolute path, a relative path identifies a unique file, specifying only the path necessary to
reach the file from the working directory. Recognizing relative path names follows a simple rule: A
path name with anything other than a forward slash as the first character is a relative path name. A
user in the /var directory could refer to the message log file relatively as *log/messages*.

Linux file systems, including, but not limited to, ext4, XFS, GFS2, and GlusterFS, are case-sensitive.
Creating *FileCase.txt* and *filecase.txt* in the same directory results in two unique files.

Non-Linux file systems might work differently. For example, VFAT, Microsoft's NTFS, and Apple's
HFS+ have case preserving behavior. Although these file systems are not case-sensitive, they do
display file names with the original capitalization used when the file was created. Therefore, if
you tried to make the files in the preceding example on a VFAT file system, both names would be
treated as pointing to the same file instead of two different files.

### NAVIGATING PATHS ###

The **pwd** command displays the full path name of the current working directory for that shell. This
can help you determine the syntax to reach files using relative path names. The ls command lists
directory contents for the specified directory or, if no directory is given, for the current working
directory.


``` bash
[user@host ~]$ pwd
/home/user
[user@host ~]$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
[user@host ~]$
```

Use the **cd** command to change your shell's current working directory. If you do not specify any
arguments to the command, it will change to your home directory.

In the following example, a mixture of absolute and relative paths are used with the cd command to
change the current working directory for the shell.

``` bash
[user@host ~]$ pwd
/home/user
[user@host ~]$ cd Videos
[user@host Videos]$ pwd
/home/user/Videos
[user@host Videos]$ cd /home/user/Documents
[user@host Documents]$ pwd
/home/user/Documents
[user@host Documents]$ cd
[user@host ~]$ pwd
/home/user
```

As you can see in the preceding example, the default shell prompt also displays the last component
of the absolute path to the current working directory. For example, for **/home/user/Videos**, only
**Videos** displays. The prompt displays the tilde character (~) when your current working directory
is your home directory.

The **touch** command normally updates a file's timestamp to the current date and time without
otherwise modifying it. This is useful for creating empty files, which can be used for practice,
because "touching" a file name that does not exist causes the file to be created. In the following
example, the touch command creates practice files in the Documents and Videos subdirectories.

``` bash
[user@host ~]$ touch Videos/blockbuster1.ogg
[user@host ~]$ touch Videos/blockbuster2.ogg
[user@host ~]$ touch Documents/thesis_chapter1.odf
[user@host ~]$ touch Documents/thesis_chapter2.odf
[user@host ~]$
```

The **ls** command has multiple options for displaying attributes on files. The most common and
useful are -l (long listing format), -a (all files, including hidden files), and -R (recursive, to include
the contents of all subdirectories).

``` bash
[user@host ~]$ ls -l
total 15
drwxr-xr-x.  2 user user 4096 Feb  7 14:02 Desktop
drwxr-xr-x.  2 user user 4096 Jan  9 15:00 Documents
drwxr-xr-x.  3 user user 4096 Jan  9 15:00 Downloads
drwxr-xr-x.  2 user user 4096 Jan  9 15:00 Music
drwxr-xr-x.  2 user user 4096 Jan  9 15:00 Pictures
drwxr-xr-x.  2 user user 4096 Jan  9 15:00 Public
drwxr-xr-x.  2 user user 4096 Jan  9 15:00 Templates
drwxr-xr-x.  2 user user 4096 Jan  9 15:00 Videos
[user@host ~]$ ls -la
total 15
drwx------. 16 user user   4096 Feb  8 16:15 .
drwxr-xr-x.  6 root root   4096 Feb  8 16:13 ..
-rw-------.  1 user user  22664 Feb  8 00:37 .bash_history
-rw-r--r--.  1 user user     18 Jul  9  2013 .bash_logout
-rw-r--r--.  1 user user    176 Jul  9  2013 .bash_profile
-rw-r--r--.  1 user user    124 Jul  9  2013 .bashrc
drwxr-xr-x.  4 user user   4096 Jan 20 14:02 .cache
drwxr-xr-x.  8 user user   4096 Feb  5 11:45 .config
drwxr-xr-x.  2 user user   4096 Feb  7 14:02 Desktop
drwxr-xr-x.  2 user user   4096 Jan  9 15:00 Documents
drwxr-xr-x.  3 user user   4096 Jan 25 20:48 Downloads
drwxr-xr-x. 11 user user   4096 Feb  6 13:07 .gnome2
drwx------.  2 user user   4096 Jan 20 14:02 .gnome2_private
-rw-------.  1 user user  15190 Feb  8 09:49 .ICEauthority
drwxr-xr-x.  3 user user   4096 Jan  9 15:00 .local
drwxr-xr-x.  2 user user   4096 Jan  9 15:00 Music
drwxr-xr-x.  2 user user   4096 Jan  9 15:00 Pictures
drwxr-xr-x.  2 user user   4096 Jan  9 15:00 Public
drwxr-xr-x.  2 user user   4096 Jan  9 15:00 Templates
drwxr-xr-x.  2 user user   4096 Jan  9 15:00 Videos
```

The two special directories at the top of the listing refer to the current directory (.) and the parent
directory (..). These special directories exist in every directory on the system. You will discover
their usefulness when you start using file management commands.

!!! danger "IMPORTANT"

	File names beginning with a dot (.) indicate hidden files; you cannot see them in the
	normal view using **ls** and other commands. This is not a security feature. Hidden
	files keep necessary user configuration files from cluttering home directories. Many
	commands process hidden files only with specific command-line options, preventing
	one user's configuration from being accidentally copied to other directories or users.

	To protect file **contents** from improper viewing requires the use of *file permissions*.


``` bash
[user@host ~]$ ls -R
.:
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos

./Desktop:

./Documents:
thesis_chapter1.odf  thesis_chapter2.odf

./Downloads:

./Music:

./Pictures:

./Public:

./Templates:

./Videos:
blockbuster1.ogg  blockbuster2.ogg
[user@host ~]$
```

The **cd** command has many options. A few are so useful as to be worth practicing early and using
often. The command **cd -** changes to the previous directory; where the user was previously to
the current directory. The following example illustrates this behavior, alternating between two
directories, which is useful when processing a series of similar tasks.

``` bash
[user@host ~]$ cd Videos
[user@host Videos]$ pwd
/home/user/Videos
[user@host Videos]$ cd /home/user/Documents
[user@host Documents]$ pwd
/home/user/Documents
[user@host Documents]$ cd -
[user@host Videos]$ pwd
/home/user/Videos
[user@host Videos]$ cd -
[user@host Documents]$ pwd
/home/user/Documents
[user@host Documents]$ cd -
[user@host Videos]$ pwd
/home/user/Videos
[user@host Videos]$ cd
[user@host ~]$
```

The cd .. command uses the .. hidden directory to move up one level to the parent directory,
without needing to know the exact parent name. The other hidden directory (.) specifies the
current directory on commands in which the current location is either the source or destination
argument, avoiding the need to type out the directory's absolute path name.

``` bash
[user@host Videos]$ pwd
/home/user/Videos
[user@host Videos]$ cd .
[user@host Videos]$ pwd
/home/user/Videos
[user@host Videos]$ cd ..
[user@host ~]$ pwd
/home/user
[user@host ~]$ cd ..
[user@host home]$ pwd
/home
[user@host home]$ cd ..
[user@host /]$ pwd
/
[user@host /]$ cd
[user@host ~]$ pwd
/home/user
[user@host ~]$
```

### MANAGING FILES USING COMMAND-LINE TOOLS ###

#### COMMAND-LINE FILE MANAGEMENT ####

To manage files, you need to be able to create, remove, copy, and move them. You also need to
organize them logically into directories, which you also need to be able to create, remove, copy,
and move.

The following table summarizes some of the most common file management commands. The
remainder of this section will discuss ways to use these commands in more detail.

**Common file management commands**

Create a directory

<mark>*mkdir directory*<mark>

Copy a file

<mark>*cp file new-file*<mark>

Copy a directory and its contents

<mark>*cp -r directory new-directory*<mark>

Move or rename a file or directory

<mark>*mv file new-file*<mark>

Remove a file

<mark>*rm file*<mark>

Remove a directory containing files

<mark>*rm -r directory*<mark>

Remove an empty directory

<mark>*rmdir directory*<mark>

#### Creating Directories ####

The **mkdir** command creates one or more directories or subdirectories. It takes as arguments a list
of paths to the directories you want to create.

The **mkdir** command will fail with an error if the directory already exists, or if you are trying to
create a subdirectory in a directory that does not exist. The -p (parent) option creates missing
parent directories for the requested destination. Use the **mkdir -p** command with caution,
because spelling mistakes can create unintended directories without generating error messages.

In the following example, pretend that you are trying to create a directory in the Videos directory

``` bash
[user@host ~]$ mkdir Video/Watched
mkdir: cannot create directory `Video/Watched': No such file or directory
```

The **mkdir** command failed because Videos was misspelled and the directory Video does not
exist. If you had used the mkdir command with the -p option, the directory Video would be
created, which was not what you had intended, and the subdirectory Watched would be created in
that incorrect directory.

After correctly spelling the Videos parent directory, creating the Watched subdirectory will
succeed.

``` bash
[user@host ~]$ mkdir Videos/Watched
[user@host ~]$ ls -R Videos
Videos/:
blockbuster1.ogg  blockbuster2.ogg  Watched

Videos/Watched:
```

In the following example, files and directories are organized beneath the /home/user/
Documents directory. Use the **mkdir** command and a space-delimited list of the directory names
to create multiple directories.

``` bash
[user@host ~]$ cd Documents
[user@host Documents]$ mkdir ProjectX ProjectY
[user@host Documents]$ ls
ProjectX  ProjectY
```

Use the **mkdir -p** command and space-delimited relative paths for each of the subdirectory
names to create multiple parent directories with subdirectories.

``` bash
[user@host Documents]$ mkdir -p Thesis/Chapter1 Thesis/Chapter2 Thesis/Chapter3
[user@host Documents]$ cd
[user@host ~]$ ls -R Videos Documents
Documents:
ProjectX  ProjectY  Thesis
Documents/ProjectX:
Documents/ProjectY:
Documents/Thesis:
Chapter1  Chapter2  Chapter3
Documents/Thesis/Chapter1:
Documents/Thesis/Chapter2:
Documents/Thesis/Chapter3:
Videos:
blockbuster1.ogg  blockbuster2.ogg  Watched
Videos/Watched:
```

The last **mkdir** command created three ChapterN subdirectories with one command. The -p
option created the missing parent directory Thesis.

#### Copying Files ####

The *cp* command copies a file, creating a new file either in the current directory or in a specified
directory. It can also copy multiple files to a directory.

!!! danger "WARNING"

	If the destination file already exists, the cp command overwrites the file.

``` bash
[user@host ~]$ cd Videos
[user@host Videos]$ cp blockbuster1.ogg blockbuster3.ogg
[user@host Videos]$ ls -l
total 0
-rw-rw-r--. 1 user user    0 Feb  8 16:23 blockbuster1.ogg
-rw-rw-r--. 1 user user    0 Feb  8 16:24 blockbuster2.ogg
-rw-rw-r--. 1 user user    0 Feb  8 16:34 blockbuster3.ogg
drwxrwxr-x. 2 user user 4096 Feb  8 16:05 Watched
[user@host Videos]$
```

When copying multiple files with one command, the last argument must be a directory. Copied files
retain their original names in the new directory. If a file with the same name exists in the target
directory, the existing file is overwritten. By default, the *cp* does not copy directories; it ignores
them.

In the following example, two directories are listed, Thesis and ProjectX. Only the last
argument, ProjectX is valid as a destination. The Thesis directory is ignored.

``` bash
[user@host Videos]$ cd ../Documents
[user@host Documents]$ cp thesis_chapter1.odf thesis_chapter2.odf Thesis ProjectX
cp: omitting directory `Thesis'
[user@host Documents]$ ls Thesis ProjectX
ProjectX:
thesis_chapter1.odf  thesis_chapter2.odf
Thesis:
Chapter1  Chapter2  Chapter3
```

In the first *cp* command, the Thesis directory failed to copy, but the *thesis_chapter1.odf*
and *thesis_chapter2.odf* files succeeded.

If you want to copy a file to the current working directory, you can use the special . directory:

``` bash
[user@host ~]$ cp /etc/hostname .
[user@host ~]$ cat hostname
host.example.com
[user@host ~]$
```

Use the copy command with the -r (recursive) option, to copy the Thesis directory and its
contents to the ProjectX directory.

``` bash
[user@host Documents]$ cp -r Thesis ProjectX
[user@host Documents]$ ls -R ProjectX
ProjectX:
Thesis  thesis_chapter1.odf  thesis_chapter2.odf

ProjectX/Thesis:
Chapter1  Chapter2  Chapter3

ProjectX/Thesis/Chapter1:

ProjectX/Thesis/Chapter2:
thesis_chapter2.odf

ProjectX/Thesis/Chapter3:
```

#### Moving Files ####

The *mv* command moves files from one location to another. If you think of the absolute path to a
file as its full name, moving a file is effectively the same as renaming a file. File contents remain
unchanged.

Use the mv command to rename a file.

``` bash
[user@host Videos]$ cd ../Documents
[user@host Documents]$ ls -l thesis*
-rw-rw-r--. 1 user user 0 Feb  6 21:16 thesis_chapter1.odf
-rw-rw-r--. 1 user user 0 Feb  6 21:16 thesis_chapter2.odf
[user@host Documents]$ mv thesis_chapter2.odf thesis_chapter2_reviewed.odf
[user@host Documents]$ ls -l thesis*
-rw-rw-r--. 1 user user 0 Feb  6 21:16 thesis_chapter1.odf
-rw-rw-r--. 1 user user 0 Feb  6 21:16 thesis_chapter2_reviewed.odf
```

Use the **mv** command to move a file to a different directory.

``` bash
[user@host Documents]$ ls Thesis/Chapter1
[user@host Documents]$
[user@host Documents]$ mv thesis_chapter1.odf Thesis/Chapter1
[user@host Documents]$ ls Thesis/Chapter1
thesis_chapter1.odf
[user@host Documents]$ ls -l thesis*
-rw-rw-r--. 1 user user 0 Feb  6 21:16 thesis_chapter2_reviewed.odf
```

#### Removing Files and Directories ####

The *rm* command removes files. By default, rm will not remove directories that contain files, unless
you add the -r or --recursive option.

!!! danger "IMPORTANT"

	There is no command-line undelete feature, nor a "trash bin" from which you can
	restore files staged for deletion.

It is a good idea to verify your current working directory before removing a file or directory.

``` bash
[user@host Documents]$ pwd
/home/student/Documents
```

Use the **rm** command to remove a single file from your working directory.

``` bash
[user@host Documents]$ ls -l thesis*
-rw-rw-r--. 1 user user 0 Feb  6 21:16 thesis_chapter2_reviewed.odf
[user@host Documents]$ rm thesis_chapter2_reviewed.odf
[user@host Documents]$ ls -l thesis*
ls: cannot access 'thesis*': No such file or directory
```

If you attempt to use the **rm** command to remove a directory without using the -r option, the
command will fail.

``` bash
[user@host Documents]$ rm Thesis/Chapter1
rm: cannot remove `Thesis/Chapter1': Is a directory
```

Use the **rm -r** command to remove a subdirectory and its contents.

``` bash
[user@host Documents]$ ls -R Thesis
Thesis/:
Chapter1  Chapter2  Chapter3
Thesis/Chapter1:
thesis_chapter1.odf
Thesis/Chapter2:
thesis_chapter2.odf
Thesis/Chapter3:
[user@host Documents]$ rm -r Thesis/Chapter1
[user@host Documents]$ ls -l Thesis
total 8
drwxrwxr-x. 2 user user 4096 Feb 11 12:47 Chapter2
drwxrwxr-x. 2 user user 4096 Feb 11 12:48 Chapter3
```

The **rm -r** command traverses each subdirectory first, individually removing their files before
removing each directory. You can use the **rm -ri** command to interactively prompt for
confirmation before deleting. This is essentially the opposite of using the -f option, which forces
the removal without prompting the user for confirmation.

``` bash
[user@host Documents]$ rm -ri Thesis
rm: descend into directory `Thesis'? y
rm: descend into directory `Thesis/Chapter2'? y
rm: remove regular empty file `Thesis/Chapter2/thesis_chapter2.odf'? y
rm: remove directory `Thesis/Chapter2'? y
rm: remove directory `Thesis/Chapter3'? y
rm: remove directory `Thesis'? y
```

!!! danger "WARNING"

	If you specify both the -i and -f options, the -f option takes priority and you will
	not be prompted for confirmation before **rm** deletes files.

In the following example, the **rmdir** command only removes the directory that is empty. Just
like the earlier example, you must use the rm -r command to remove a directory that contains
content.

``` bash
[user@host Documents]$ pwd
/home/student/Documents
[user@host Documents]$ rmdir ProjectY
[user@host Documents]$ rmdir ProjectX
rmdir: failed to remove `ProjectX': Directory not empty
[user@host Documents]$ rm -r ProjectX
[user@host Documents]$ ls -lR
.:
total 0
[user@host Documents]$
```


!!! info "NOTE"

	The rm command with no options cannot remove an empty directory. You must use
	the rmdir command, rm -d (which is equivalent to rmdir), or rm -r.

### MAKING LINKS BETWEEN FILES ###

#### MANAGING LINKS BETWEEN FILES ####

**Hard Links and Soft Links**

It is possible to create multiple names that point to the same file. There are two ways to do this: by
creating a hard link to the file, or by creating a *soft link* (sometimes called a symbolic link) to the file.
Each has its advantages and disadvantages.

**Creating Hard Links**

Every file starts with a single hard link, from its initial name to the data on the file system. When
you create a new hard link to a file, you create another name that points to that same data. The
new hard link acts exactly like the original file name. Once created, you cannot tell the difference
between the new hard link and the original name of the file.

You can find out if a file has multiple hard links with the ls -l command. One of the things it
reports is each file's link count, the number of hard links the file has.

``` bash
[user@host ~]$ pwd
/home/user
[user@host ~]$ ls -l newfile.txt
-rw-r--r--. 1 user user 0 Mar 11 19:19 newfile.txt
```

In the preceding example, the link count of newfile.txt is 1. It has exactly one absolute path,
which is /home/user/newfile.txt.

You can use the ln command to create a new hard link (another name) that points to an existing
file. The command needs at least two arguments, a path to the existing file, and the path to the
hard link that you want to create.

The following example creates a hard link named newfile-link2.txt for the existing file
newfile.txt in the */tmp* directory.

``` bash
[user@host ~]$ ln newfile.txt /tmp/newfile-hlink2.txt
[user@host ~]$ ls -l newfile.txt /tmp/newfile-hlink2.txt
-rw-rw-r--. 2 user user 12 Mar 11 19:19 newfile.txt
-rw-rw-r--. 2 user user 12 Mar 11 19:19 /tmp/newfile-hlink2.txt
```

If you want to find out whether two files are hard links of each other, one way is to use the -i
option with the ls command to list the files' inode number. If the files are on the same file system
(discussed in a moment) and their inode numbers are the same, the files are hard links pointing to
the same data.

``` bash
[user@host ~]$ ls -il newfile.txt /tmp/newfile-hlink2.txt
8924107 -rw-rw-r--. 2 user user 12 Mar 11 19:19 newfile.txt
8924107 -rw-rw-r--. 2 user user 12 Mar 11 19:19 /tmp/newfile-hlink2.txt
```

!!! danger "IMPORTANT"

	All hard links that reference the same file will have the same link count, access
	permissions, user and group ownerships, time stamps, and file content. If any of that
	information is changed using one hard link, all other hard links pointing to the same
	file will show the new information as well. This is because each hard link points to the
	same data on the storage device.

Even if the original file gets deleted, the contents of the file are still available as long as at least one
hard link exists. Data is only deleted from storage when the last hard link is deleted.

``` bash
[user@host ~]$ rm -f newfile.txt
[user@host ~]$ ls -l /tmp/newfile-hlink2.txt
-rw-rw-r--. 1 user user 12 Mar 11 19:19 /tmp/newfile-hlink2.txt
[user@host ~]$ cat /tmp/newfile-hlink2.txt
Hello World
```

#### Limitations of Hard Links ####

Hard links have some limitations. Firstly, hard links can only be used with regular files. You cannot
use **ln** to create a hard link to a directory or special file.

Secondly, hard links can only be used if both files are on the same file system. The file-system
hierarchy can be made up of multiple storage devices. Depending on the configuration of your
system, when you change into a new directory, that directory and its contents may be stored on a
different file system.

You can use the **df** command to list the directories that are on different file systems. For example,
you might see output like the following:

``` bash
[user@host ~]$ df
Filesystem                   1K-blocks    Used Available Use% Mounted on
devtmpfs                        886788       0    886788   0% /dev
tmpfs                           902108       0    902108   0% /dev/shm
tmpfs                           902108    8696    893412   1% /run
tmpfs                           902108       0    902108   0% /sys/fs/cgroup
/dev/mapper/rhel_rhel8--root  10258432 1630460   8627972  16% /
/dev/sda1                      1038336  167128    871208  17% /boot
tmpfs                           180420       0    180420   0% /run/user/1000
[user@host ~]$ 
```

Files in two different "Mounted on" directories and their subdirectories are on different file
systems. (The most specific match wins.) So, the system in this example, you can create a hard
link between /var/tmp/link1 and /home/user/file because they are both subdirectories
of / but not any other directory on the list. But you cannot create a hard link between /boot/
test/badlink and /home/user/file because the first file is in a subdirectory of /boot (on
the "Mounted on" list) and the second file is not.

#### Creating Soft Links ####

The **ln -s** command creates a soft link, which is also called a "symbolic link." A soft link is not a
regular file, but a special type of file that points to an existing file or directory.

Soft links have some advantages over hard links:
- They can link two files on different file systems.
- They can point to a directory or special file, not just a regular file.

In the following example, the ln -s command is used to create a new soft link for the existing file
/home/user/newfile-link2.txt that will be named /tmp/newfile-symlink.txt.

``` bash
[user@host ~]$ ln -s /home/user/newfile-link2.txt /tmp/newfile-symlink.txt
[user@host ~]$ ls -l newfile-link2.txt /tmp/newfile-symlink.txt
-rw-rw-r--. 1 user user 12 Mar 11 19:19 newfile-link2.txt
lrwxrwxrwx. 1 user user 11 Mar 11 20:59 /tmp/newfile-symlink.txt -> /home/user/
newfile-link2.txt
[user@host ~]$ cat /tmp/newfile-symlink.txt
Soft Hello World
```

In the preceding example, the first character of the long listing for /tmp/newfile-symlink.txt
is l instead of -. This indicates that the file is a soft link and not a regular file. (A d would indicate
that the file is a directory.)

When the original regular file gets deleted, the soft link will still point to the file but the target is
gone. A soft link pointing to a missing file is called a "dangling soft link."

``` bash
[user@host ~]$ rm -f newfile-link2.txt
[user@host ~]$ ls -l /tmp/newfile-symlink.txt
lrwxrwxrwx. 1 user user 11 Mar 11 20:59 /tmp/newfile-symlink.txt -> /home/user/
newfile-link2.txt
[user@host ~]$ cat /tmp/newfile-symlink.txt
cat: /tmp/newfile-symlink.txt: No such file or directory
```

!!! danger "IMPORTANT"

	One side-effect of the dangling soft link in the preceding example is that if you later
	create a new file with the same name as the deleted file (/home/user/newfile-
	link2.txt), the soft link will no longer be "dangling" and will point to the new file.
	Hard links do not work like this. If you delete a hard link and then use normal tools
	(rather than ln) to create a new file with the same name, the new file will not be
	linked to the old file.
	
	One way to compare hard links and soft links that might help you understand how
	they work:
	
	- A hard link points a name to data on a storage device
	- A soft link points a name to another name, that points to data on a storage device

A soft link can point to a directory. The soft link then acts like a directory. Changing to the soft
link with **cd** will make the current working directory the linked directory. Some tools may keep
track of the fact that you followed a soft link to get there. For example, by default **cd** will update
your current working directory using the name of the soft link rather than the name of the actual
directory. (There is an option, -P, that will update it with the name of the actual directory instead.)

In the following example, a soft link named /home/user/configfiles is created that points to
the /etc directory.

``` bash
[user@host ~]$ ln -s /etc /home/user/configfiles
[user@host ~]$ cd /home/user/configfiles
[user@host configfiles]$ pwd
/home/user/configfiles
```

### MATCHING FILE NAMES WITH SHELL EXPANSIONS ###

#### COMMAND-LINE EXPANSIONS ####

The Bash shell has multiple ways of expanding a command line including *pattern matching*, home
directory expansion, string expansion, and variable substitution. Perhaps the most powerful of
these is the path name-matching capability, historically called *globbing*. The Bash globbing feature,
sometimes called “wildcards”, makes managing large numbers of files easier. Using *metacharacters*
that “expand” to match file and path names being sought, commands perform on a focused set of
files at once.

**Pattern Matching**

Globbing is a shell command-parsing operation that expands a wildcard pattern into a list of
matching path names. Command-line metacharacters are replaced by the match list prior to
command execution. Patterns that do not return matches display the original pattern request as
literal text. The following are common metacharacters and pattern classes.

Table of Metacharacters and Matches

| PATTERN     | MATCHES                                                                                                     |
|:-----------:|:-----------------------------------------------------------------------------------------------------------:|
| *           | Any string of zero or more characters.                                                                      |
| ?           | Any single character.                                                                                       |
| [abc...]    | Any one character in the enclosed class (between the square brackets).                                      |
| [!abc...]   | Any one character not in the enclosed class.                                                                |
| [^abc...]   | Any one character not in the enclosed class.                                                                |
| [[:alpha:]] | Any alphabetic character.                                                                                   |
| [[:lower:]] | Any lowercase character.                                                                                    |
| [[:upper:]] | Any uppercase character.                                                                                    |
| [[:alnum:]] | Any alphabetic character or digit.                                                                          |
| [[:punct:]] | Any printable character not a space or alphanumeric.                                                        |
| [[:digit:]] | Any single digit from 0 to 9.                                                                               |
| [[:space:]] | Any single white space character. This may include tabs, newlines, carriage returns, form feeds, or spaces. |


For the next few examples, pretend that you have run the following commands to create some
sample files.

``` bash
[user@host ~]$ mkdir glob; cd glob
[user@host glob]$ touch alfa bravo charlie delta echo able baker cast dog easy
[user@host glob]$ ls
able  alfa  baker  bravo  cast  charlie  delta  dog  easy  echo
[user@host glob]$ 
```

The first example will use simple pattern matches with the asterisk (*) and question mark (?)
characters, and a class of characters, to match some of those file names.

``` bash
[user@host glob]$ ls a*
able  alfa
[user@host glob]$ ls *a*
able  alfa  baker  bravo  cast  charlie  delta  easy
[user@host glob]$ ls [ac]*
able  alfa  cast  charlie
[user@host glob]$ ls ????
able  alfa  cast  easy  echo
[user@host glob]$ ls ?????
baker  bravo  delta
[user@host glob]$ 
```

**Tilde Expansion**

The tilde character (~), matches the current user's home directory. If it starts a string of characters
other than a slash (/), the shell will interpret the string up to that slash as a user name, if one
matches, and replace the string with the absolute path to that user's home directory. If no user
name matches, then an actual tilde followed by the string of characters will be used instead.

In the following example the echo command is used to display the value of the tilde character. The
echo command can also be used to display the values of brace and variable expansion characters,
and others.

``` bash
[user@host glob]$ ls ~root
/root
[user@host glob]$ ls ~user
/home/user
[user@host glob]$ ls ~/glob
able  alfa  baker  bravo  cast  charlie  delta  dog  easy  echo
[user@host glob]$ echo ~/glob
/home/user/glob
[user@host glob]$ 
```

**Brace Expansion**

Brace expansion is used to generate discretionary strings of characters. Braces contain a comma-
separated list of strings, or a sequence expression. The result includes the text preceding or
following the brace definition. Brace expansions may be nested, one inside another. Also double-
dot syntax (..) expands to a sequence such that {m..p} will expand to m n o p.

``` bash
[user@host glob]$ echo {Sunday,Monday,Tuesday,Wednesday}.log
Sunday.log Monday.log Tuesday.log Wednesday.log
[user@host glob]$ echo file{1..3}.txt
file1.txt file2.txt file3.txt
[user@host glob]$ echo file{a..c}.txt
filea.txt fileb.txt filec.txt
[user@host glob]$ echo file{a,b}{1,2}.txt
filea1.txt filea2.txt fileb1.txt fileb2.txt
[user@host glob]$ echo file{a{1,2},b,c}.txt
filea1.txt filea2.txt fileb.txt filec.txt
[user@host glob]$ 
```

A practical use of brace expansion is to quickly create a number of files or directories.

``` bash
[user@host glob]$ mkdir ../RHEL{6,7,8}
[user@host glob]$ ls ../RHEL*
RHEL6 RHEL7 RHEL8
[user@host glob]$ 
```

**Variable Expansion**

A variable acts like a named container that can store a value in memory. Variables make it easy to
access and modify the stored data either from the command line or within a shell script.

You can assign data as a value to a variable using the following syntax:

``` bash
[user@host ~]$ VARIABLENAME=value
```

You can use variable expansion to convert the variable name to its value on the command line. If a
string starts with a dollar sign ($), then the shell will try to use the rest of that string as a variable
name and replace it with whatever value the variable has.

``` bash
[user@host ~]$ USERNAME=operator
[user@host ~]$ echo $USERNAME
operator
```

To help avoid mistakes due to other shell expansions, you can put the name of the variable in curly
braces, for example ${VARIABLENAME}.

``` bash
[user@host ~]$ USERNAME=operator
[user@host ~]$ echo ${USERNAME}
operator
```

Shell variables and ways to use them will be covered in more depth later in this course.

**Command Substitution**

Command substitution allows the output of a command to replace the command itself on the
command line. Command substitution occurs when a command is enclosed in parentheses, and
preceded by a dollar sign ($). The $(command) form can nest multiple command expansions
inside each other.

``` bash
[user@host glob]$ echo Today is $(date +%A).
Today is Wednesday.
[user@host glob]$ echo The time is $(date +%M) minutes past $(date +%l%p).
The time is 26 minutes past 11AM.
[user@host glob]$ 
```

!!! info "NOTE"

	An older form of command substitution uses backticks. Disadvantages
	to the backticks form include: 1) it can be easy to visually confuse backticks with
	single quote marks, and 2) backticks cannot be nested.

**Protecting Arguments from Expansion**

Many characters have special meaning in the Bash shell. To keep the shell from performing shell
expansions on parts of your command line, you can quote and escape characters and strings.

The backslash (\) is an escape character in the Bash shell. It will protect the character immediately
following it from expansion.

``` bash
[user@host glob]$ echo The value of $HOME is your home directory.
The value of /home/user is your home directory.
[user@host glob]$ echo The value of \$HOME is your home directory.
The value of $HOME is your home directory.
[user@host glob]$
```

In the preceding example, protecting the dollar sign from expansion caused Bash to treat it as a
regular character and it did not perform variable expansion on *$HOME*.

To protect longer character strings, single quotes (') or double quotes (") are used to enclose
strings. They have slightly different effects. Single quotes stop all shell expansion. Double quotes
stop most shell expansion.

Use double quotation marks to suppress globbing and shell expansion, but still allow command and
variable substitution.

``` bash
[user@host glob]$ myhost=$(hostname -s); echo $myhost
host
[user@host glob]$ echo "***** hostname is ${myhost} *****"
***** hostname is host *****
[user@host glob]$ 
```

Use single quotation marks to interpret all text literally.

``` bash
[user@host glob]$ echo "Will variable $myhost evaluate to $(hostname -s)?"
Will variable myhost evaluate to host?
[user@host glob]$ echo 'Will variable $myhost evaluate to $(hostname -s)?'
Will variable $myhost evaluate to $(hostname -s)?
[user@host glob]$ 
```

!!! danger "IMPORTANT" 

	The single quote (') and the command substitution backtick (`) can be easy to
	confuse, both on the screen and on the keyboard. Using one when you mean to use

### SUMMARY ###

In this chapter, you learned:

- Files on a Linux system are organized into a single inverted tree of directories, known as a file-
system hierarchy.

- Absolute paths start with a / and specify the location of a file in the file-system hierarchy.

- Relative paths do not start with a / and specify the location of a file relative to the current
working directory.

- Five key commands are used to manage files: mkdir, rmdir, cp, mv, and rm.

- Hard links and soft links are different ways to have multiple file names point to the same data.

- The Bash shell provides pattern matching, expansion, and substitution features to help you
efficiently run commands.
