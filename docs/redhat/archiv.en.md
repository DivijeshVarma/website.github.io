---
title: "Archiving And Transferring Files"
---

### MANAGING COMPRESSED TAR ARCHIVES ###

#### THE tar COMMAND ####

Archiving and compressing files are useful when creating backups and transferring data across
a network. One of the oldest and most common commands for creating and working with backup
archives is the **tar** command.

With **tar**, users can gather large sets of files into a single file (archive). A **tar** archive is a
structured sequence of file data mixed in with metadata about each file and an index so that
individual files can be extracted. The archive can be compressed using gzip, bzip2, or xz
compression.

The **tar** command can list the contents of archives or extract their files to the current system.

#### SELECTED tar OPTIONS ####

tar command options are divided into operations (the action you want to take): general options
and compression options. The table below shows common options, long version of options, and
their description:

**Overview of tar Operations**

| OPTION          | DESCRIPTION                               |
|:----------------|:------------------------------------------|
| "-c, --create"  | Create a new archive.                     |
| "-x, --extract" | Extract from an existing archive.         |
| "-t, --list"    | List the table of contents of an archive. |

**Selected tar General Options**

| OPTION                       | DESCRIPTION                                                                                                  |
|:-----------------------------|:-------------------------------------------------------------------------------------------------------------|
| "-v, --verbose"              | Verbose. Shows which files get archived or extracted.                                                        |
| "-f, --file="                | File name. This option must be followed by the file name of the archive to use or create.                    |
| "-p, --preserve-permissions" | Preserve the permissions of files and directories when extracting an archive, without subtracting the umask. |
| "-z, --gzip"                 | Use gzip compression (.tar.gz).                                                                              |
| "-j, --bzip2"                | Use bzip2 compression (.tar.bz2). bzip2 typically achieves a better compression ratio than gzip.             |
| "-J, --xz"                   | Use xz compression (.tar.xz). The xz compression typically achieves a better compression ratio than bzip2.   |

#### LISTING OPTIONS OF THE tar COMMAND ####

The **tar** command expects one of the three following options:

- Use the **-c** or **--create** option to create an archive.
- Use the **-t** or **--list** option to list the contents of an archive.
- Use the **-x** or **--extract** option to extract an archive.

Other commonly used options are:

- Use the **-f** or **--file=** option with a file name as an argument of the archive to operate.
- Use the **-v** or **--verbose** option for verbosity; useful to see which files get added to or
extracted from the archive.

!!! info "NOTE"

	The tar command actually supports a third, old option style that uses the standard
	single-letter options with no leading -. It is still commonly encountered, and
	you might run into this syntax when working with other people's instructions or
	commands. The **info tar 'old options'** command discusses how this differs
	from normal short options in some detail.

	You can ignore old options for now and focus on the standard short and long options
	syntax.

### ARCHIVING FILES AND DIRECTORIES ###

The first option to use when creating a new archive is the **c** option, followed by the **f** option, then
a single space, then the file name of the archive to be created, and finally the list of files and
directories that should get added to the archive. The archive is created in the current directory
unless specified otherwise.

!!! danger "WARNING"

	Before creating a tar archive, verify that there is no other archive in the directory
	with the same name as the new archive to be created. The **tar** command overwrites
	an existing archive without warning.

The following command creates an archive named **archive.tar** with the contents of **file1**,
**file2**, and **file3** in the user's home directory.

``` bash
[user@host ~]$ tar -cf archive.tar file1 file2 file3
[user@host ~]$ ls archive.tar
archive.tar
```

The above **tar** command can also be executed using the long version options.

``` bash
[user@host ~]$ tar --file=archive.tar --create file1 file2 file3
```

!!! info "NOTE"

	When archiving files by absolute path names, the leading / of the path is removed
	from the file name by default. Removing the leading / of the path help users to avoid
	overwriting important files when extracting the archive. The **tar** command extracts
	files relative to the current working directory.

For **tar** to be able to archive the selected files, it is mandatory that the user executing the **tar**
command can read the files. For example, creating a new archive of the **/etc** folder and all of
its content requires root privileges, because only the root user is allowed to read all of the
files present in the **/etc** directory. An unprivileged user can create an archive of the **/etc**
directory, but the archive omits files which do not include read permission for the user, and it omits
directories which do not include both read and execute permission for the user.

To create the tar archive named, **/root/etc.tar**, with the **/etc** directory as content as user
root:

``` bash
[root@host ~]# tar -cf /root/etc.tar /etc
tar: Removing leading `/' from member names
[root@host ~]# 
```

!!! danger "IMPORTANT"

	Some advanced permissions that we have not covered in this course, such as ACLs
	and SELinux contexts, are not automatically stored in a **tar** archive. Use the **--
	xattrs** option when creating an archive to store those extended attributes in the
	tar archive.

#### LISTING CONTENTS OF AN ARCHIVE ####

The **t** option directs **tar** to list the contents (table of contents, hence t) of the archive. Use the **f**
option with the name of the archive to be queried. For example:

``` bash
[root@host ~]# tar -tf /root/etc.tar
etc/
etc/fstab
etc/crypttab
etc/mtab
...output omitted...
```

#### EXTRACTING FILES FROM AN ARCHIVE ####

A tar archive should usually be extracted in an empty directory to ensure it does not overwrite any
existing files. When root extracts an archive, the **tar** command preserves the original user and
group ownership of the files. If a regular user extracts files using tar, the file ownership belongs to
the user extracting the files from the archive.

To restore files from the **/root/etc.tar** archive to the **/root/etcbackup** directory, run:

``` bash
[root@host ~]# mkdir /root/etcbackup
[root@host ~]# cd /root/etcbackup
[root@host etcbackup]# tar -tf /root/etc.tar
etc/
etc/fstab
etc/crypttab
etc/mtab
...output omitted...
[root@host etcbackup]# tar -xf /root/etc.tar
```

By default, when files get extracted from an archive, the umask is subtracted from the permissions
of archive content. To preserve the permissions of an archived file, the **p** option when extracting an
archive.

In this example, an archive named, **/root/myscripts.tar**, is extracted in the **/root/scripts**
directory while preserving the permissions of the extracted files:

``` bash
[root@host ~]# mkdir /root/scripts
[root@host ~]# cd /root/scripts
[root@host scripts]# tar -xpf /root/myscripts.tar
```

#### CREATING A COMPRESSED ARCHIVE ####

The **tar** command supports three compression methods. There are three different compression
methods supported by the **tar** command. The gzip compression is the fastest and oldest one
and is most widely available across distributions and even across platforms. bzip2 compression
creates smaller archive files compared to gzip but is less widely available than gzip, while the xz
compression method is relatively new, but usually offers the best compression ratio of the methods
available.
 
!!! info "NOTE"

	The effectiveness of any compression algorithm depends on the type of data that
	is compressed. Data files that are already compressed, such as compressed picture
	formats or RPM files, usually lead to a low compression ratio.

It is good practice to use a single top-level directory, which can contain other directories and files,
to simplify the extraction of the files in an organized way.

Use one of the following options to create a compressed tar archive:

- **-z** or **--gzip** for gzip compression (**filename.tar.gz** or **filename.tgz**)
- **-j** or **--bzip2** for bzip2 compression (**filename.tar.bz2**)
- **-J** or **-xz** for xz compression (**filename.tar.xz**)

To create a gzip compressed archive named **/root/etcbackup.tar.gz**, with the contents
from the **/etc** directory on host:

``` bash
[root@host ~]# tar -czf /root/etcbackup.tar.gz /etc
tar: Removing leading `/' from member names
```

To create a bzip2 compressed archive named **/root/logbackup.tar.bz2**, with the contents
from the **/var/log** directory on host:

``` bash
[root@host ~]$ tar -cjf /root/logbackup.tar.bz2 /var/log
tar: Removing leading `/' from member names
```

To create a xz compressed archive named, **/root/sshconfig.tar.xz**, with the contents from
the **/etc/ssh** directory on host:

``` bash
[root@host ~]$ tar -cJf /root/sshconfig.tar.xz /etc/ssh
tar: Removing leading `/' from member names
```

After creating an archive, verify the content of an archive using the tf options. It is not mandatory
to use the option for compression agent when listing the content of a compressed archive file. For
example, to list the content archived in the **/root/etcbackup.tar.gz** file, which uses the gzip
compression, use the following command:

``` bash
[root@host ~]# tar -tf /root/etcbackup.tar.gz /etc
etc/
etc/fstab
etc/crypttab
etc/mtab
...output omitted...
```

#### EXTRACTING A COMPRESSED ARCHIVE ####

The first step when extracting a compressed **tar** archive is to determine where the archived
files should be extracted to, then create and change to the target directory. The **tar** command
determines which compression was used and it is usually not necessary to use the same
compression option used when creating the archive. It is valid to add the decompression method to
the **tar** command. If one chooses to do so, the correct decompression type option must be used;
otherwise tar yields an error about the decompression type specified in the options not matching
the file's decompression type.

To extract the contents of a gzip compressed archive named **/root/etcbackup.tar.gz** in the
**/tmp/etcbackup** directory:

``` bash
[root@host ~]# mkdir /tmp/etcbackup
[root@host ~]# cd /tmp/etcbackup
[root@host etcbackup]# tar -tf /root/etcbackup.tar.gz
etc/
etc/fstab
etc/crypttab
etc/mtab
...output omitted...
[root@host etcbackup]# tar -xzf /root/etcbackup.tar.gz
```

To extract the contents of a bzip2 compressed archive named **/root/logbackup.tar.bz2** in
the **/tmp/logbackup** directory:

``` bash
[root@host ~]# mkdir /tmp/logbackup
[root@host ~]# cd /tmp/logbackup
[root@host logbackup]# tar -tf /root/logbackup.tar.bz2
var/log/
var/log/lastlog
var/log/README
var/log/private/
var/log/wtmp
var/log/btmp
...output omitted...
[root@host logbackup]# tar -xjf /root/logbackup.tar.bz2
```

To extract the contents of a xz compressed archive named **/root/sshbackup.tar.xz** in the **/
tmp/sshbackup** directory:

``` bash
[root@host ~]$ mkdir /tmp/sshbackup
[root@host ~]# cd /tmp/sshbackup
[root@host logbackup]# tar -tf /root/sshbackup.tar.xz
etc/ssh/
etc/ssh/moduli
etc/ssh/ssh_config
etc/ssh/ssh_config.d/
etc/ssh/ssh_config.d/05-redhat.conf
etc/ssh/sshd_config
...output omitted...
[root@host sshbackup]# tar -xJf /root/sshbackup.tar.xz
```

Listing a compressed tar archive works in the same way as listing an uncompressed **tar** archive.

!!! info "NOTE"

	Additionally, gzip, bzip2, and xz can be used independently to compress single
	files. For example, the gzip etc.tar command results in the etc.tar.gz
	compressed file, while the bzip2 abc.tar command results in the abc.tar.bz2
	compressed file, and the xz myarchive.tar command results in the
	myarchive.tar.xz compressed file.
	
	The corresponding commands to decompress are gunzip, bunzip2, and unxz.
	For example, the gunzip **/tmp/etc.tar.gz** command results in the etc.tar
	uncompressed tar file, while the bunzip2 abc.tar.bz2 command results in the
	abc.tar uncompressed tar file, and the unxz myarchive.tar.xz command
	results in the myarchive.tar uncompressed tar file.

### TRANSFERRING FILES BETWEEN SYSTEMS SECURELY ###

#### TRANSFERRING FILES USING SECURE COPY ####

OpenSSH is useful for securely running shell commands on remote systems. The Secure Copy
command, **scp**, which is part of the OpenSSH suite, copies files from a remote system to the
local system or from the local system to a remote system. The command uses the SSH server for
authentication and encrypts data when it is being transferred.

You can specify a remote location for the source or destination of the files you are copying. The
format of the remote location should be in the form [**user@]host:/path**. The **user@** portion of
the argument is optional. If it is missing, your current local username will be used. When you run
the command, your scp client will authenticate to the remote SSH server just like ssh, using key-
based authentication or prompting you for your password.

The following example demonstrates how to copy the local **/etc/yum.conf** and **/etc/hosts**
files on host, to the remoteuser's home directory on the remotehost remote system:

``` bash
[user@host ~]$ scp /etc/yum.conf /etc/hosts remoteuser@remotehost:/home/remoteuser
remoteuser@remotehost's password: password
yum.conf                                   100%  813     0.8KB/s   00:00
hosts                                      100%  227     0.2KB/s   00:00
```

You can also copy a file in the other direction, from a remote system to the local file system. In this
example, the file **/etc/hostname** on remotehost is copyed to the local directory **/home/user**.
The **scp** command authenticates to remotehost as the user remoteuser.

``` bash
[user@host ~]$ scp remoteuser@remotehost:/etc/hostname /home/user
remoteuser@remotehost's password: password
hostname                                   100%   22     0.0KB/s   00:00
```

To copy a whole directory tree recursively, use the **-r** option. In the following example, the remote
directory **/var/log** on remotehost is copied recursively to the local directory **/tmp/** on host.
You must connect to the remote system as root to make sure you can read all files in the remote **/
var/log** directory.

``` bash
[user@host ~]$ scp -r root@remoteuser:/var/log /tmp
root@remotehost's password: password
...output omitted...
```

#### TRANSFERRING FILES USING THE SECURE FILE TRANSFER PROGRAM ####

To interactively upload or download files from a SSH server, use the Secure File Transfer Program,
**sftp**. A session with the **sftp** command uses the secure authentication mechanism and
encrypted data transfer to and from the SSH server.

Just like the **scp** command, the **sftp** command uses [**user@]host** to identify the target system
and user name. If you do not specify a user, the command will attempt to log in using your local
user name as the remote user name. You will then be presented with an sftp> prompt.

``` bash
[user@host ~]$ sftp remoteuser@remotehost
remoteuser@remotehost's password: password
Connected to remotehost.
sftp> 
```

The interactive **sftp** session accepts various commands that work the same way on the remote
file system as they do in the local file system, such as **ls, cd, mkdir, rmdir, and pwd**. The **put**
command uploads a file to the remote system. The **get** command downloads a file from the remote
system. The **exit** command exits the sftp session.

To upload the **/etc/hosts** file on the local system to the newly created directory **/home/
remoteuser/hostbackup** on remotehost. The sftp session always assumes that the **put**
command is followed by a file on the local file system and starts in the connecting user's home
directory; in this case, **/home/remoteuser**:

``` bash
sftp> mkdir hostbackup
sftp> cd hostbackup
sftp> put /etc/hosts
Uploading /etc/hosts to /home/remoteuser/hostbackup/hosts
/etc/hosts                                 100%  227     0.2KB/s   00:00
sftp> 
```

To download **/etc/yum.conf** from the remote host to the current directory on the local system,
execute the command **get /etc/yum.conf** and exit the **sftp** session with the **exit** command.

``` bash
sftp> get /etc/yum.conf
Fetching /etc/yum.conf to yum.conf
/etc/yum.conf                              100%  813     0.8KB/s   00:00
sftp> exit
[user@host ~]$ 
```

### SYNCHRONIZING FILES BETWEEN SYSTEMS SECURELY ###

#### SYNCHRONIZE FILES AND DIRECTORIES WITH rsync ####

The **rsync** command is another way to securely copy files from one system to another. The tool
uses an algorithm that minimizes the amount of data copied by synchronizing only the portions
of files that have changed. It differs from **scp** in that if two files or directories are similar between
two servers, **rsync** copies the differences between the file systems on the two servers, while **scp**
would need to copy everything.

One of the advantages of **rsync** is that it can copy files between a local system and a remote
system securely and efficiently. While the initial synchronization of a directory takes about the
same time as copying it, any subsequent synchronization only requires the differences to be
copied over the network, speeding updates, possibly substantially.

One of the most important options of **rsync** is the **-n** option to perform a dry run. A dry run is a
simulation of what happens when the command gets executed. The dry run shows the changes
**rsync** would perform when the command is run without the dry run option. You should perform
a dry run before performing an **rsync** operation to ensure no important files get overwritten or
deleted.

The two most common options when synchronizing files and directories with **rsync** are the **-v** and
**-a** options.

The **-v** or **--verbose** option provides more detailed output as the synchronization runs. This is
useful for troubleshooting and to help see progress.

The **-a** or **--archive** option enables "archive mode". This is a quick way to enable recursive
copying and turn on a large number of useful options to preserve most characteristics of the files.
Archive mode is the same as specifying the following options:

**Options Enabled with rsync -a (Archive Mode)**

| OPTION            | DESCRIPTION                                      |
|:------------------|:-------------------------------------------------|
| "-r, --recursive" | synchronize recursively the whole directory tree |
| "-l, --links"     | synchronize symbolic links                       |
| "-p, --perms"     | preserve permissions                             |
| "-t, --times"     | preserve time stamps                             |
| "-g, --group"     | preserve group ownership                         |
| "-o, --owner"     | preserve the owner of the files                  |
| "-D, --devices"   | synchronize device file                          |

Archive mode does not preserve hard links, because this can add significant time to the
synchronization. If you want to preserve hard links too, add the **-H** option.

!!! info "NOTE"

	If you are using advanced permissions, you might need to add two additional
	options:
	
	- -A to preserve ACLs
	- -X to preserve SELinux contexts

You can use **rsync** to synchronize the contents of a local file or directory with a file or directory on
a remote machine, using either machine as the source. You can also synchronize the contents of
two local files or directories.

For example, to synchronize contents of the **/var/log** directory to the **/tmp** directory:

``` bash
[user@host ~]$ su -
Password: password
[root@host ~]# rsync -av /var/log /tmp
receiving incremental file list
log/
log/README
log/boot.log
...output omitted...
log/tuned/tuned.log

sent 11,592,423 bytes  received 779 bytes  23,186,404.00 bytes/sec
total size is 11,586,755  speedup is 1.00
[user@host ~]$ ls /tmp
log  ssh-RLjDdarkKiW1
[user@host ~]$ 
```

A trailing slash at the end of the source directory synchronizes the content of a directory without
newly creating the subdirectory in the target directory. In this example, the **log** directory is not
created in the **/tmp** directory. Only the content of the **/var/log/** directory is synchronized into
the **/tmp** directory.

``` bash
[root@host ~]# rsync -av /var/log/ /tmp
sending incremental file list
./
README
boot.log
...output omitted...
tuned/tuned.log
sent 11,592,389 bytes  received 778 bytes  23,186,334.00 bytes/sec
total size is 11,586,755  speedup is 1.00
[root@host ~]# ls /tmp
anaconda                  dnf.rpm.log-20190318  private
audit                     dnf.rpm.log-20190324  qemu-ga
boot.log                  dnf.rpm.log-20190331  README
...output omitted...
```

!!! danger "IMPORTANT"

	When entering the source directory for the **rsync** command, it is essential
	to remember that whether a trailing slash is present on the directory name is
	significant. It determines whether the directory or just the contents of the directory
	are synchronized to the target.
	
	Bash **Tab**-completion automatically adds a trailing slash to the end of directory
	names.

Just like the **scp** and **sftp** commands, for rsync you specify remote locations using the
[**user@]host:/path** format. The remote location can be either the source system or destination
system, but one of the two machines has to be local.

In order to preserve file ownership, you need to be root on the destination system. If the
destination is remote, authenticate as root. If the destination is local, you must run **rsync** as
root.

In this example, synchronize the local **/var/log** directory to the **/tmp** directory on the
remotehost system:

``` bash
[root@host ~]# rsync -av /var/log remotehost:/tmp
root@remotehost's password: password
receiving incremental file list
log/
log/README
log/boot.log
...output omitted...
sent 9,783 bytes  received 290,576 bytes  85,816.86 bytes/sec
total size is 11,585,690  speedup is 38.57
```

In the same way, the **/var/log** remote directory on remotehost can be synchronized to the **/
tmp** local directory on host:

``` bash
[root@host ~]# rsync -av remotehost:/var/log /tmp
root@remotehost's password: password
receiving incremental file list
log/boot.log
log/dnf.librepo.log
log/dnf.log
...output omitted...
sent 9,783 bytes  received 290,576 bytes  85,816.86 bytes/sec
total size is 11,585,690  speedup is 38.57
```

### SUMMARY ###

In this chapter, you learned:

- The **tar** command creates an archive file from a set of files and directories, extracts files from
the archive, and lists the contents of an archive.

- The **tar** command provides a set of different compression methods reduce archive size.

- Besides providing a secure remote shell, the SSH service also provides the **scp** and **sftp**
commands as secure ways to transfer files from and to a remote system running the SSH server.

- The **rsync** command securely and efficiently synchronizes files between two directories, either
one of which can be on a remote system.
