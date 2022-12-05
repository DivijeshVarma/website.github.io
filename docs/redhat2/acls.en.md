---
title: "Controlling Access To Files With Acls"
---

### INTERPRETING FILE ACLS

#### ACCESS CONTROL LIST CONCEPTS

Standard Linux file permissions are satisfactory when files are used by only a single owner, and a
single designated group of people. However, some use cases require that files are accessed with
different file permission sets by multiple named users and groups. Access Control Lists (ACLs)
provide this function.

With ACLs, you can grant permissions to multiple users and groups, identified by user name,
group name, UID, or GID, using the same permission flags used with regular file permissions: read,
write, and execute. These additional users and groups, beyond the file owner and the file's group
affiliation, are called named users and named groups respectively, because they are named not in a
long listing, but rather within an ACL.

Users can set ACLs on files and directories that they own. Privileged users, assigned the
**CAP_FOWNER** Linux capability, can set ACLs on any file or directory. New files and subdirectories
automatically inherit ACL settings from the parent directory's default ACL, if they are set. Similar
to normal file access rules, the parent directory hierarchy needs at least the other search (execute)
permission set to enable named users and named groups to have access.

**File-system ACL Support**

File systems need to be mounted with ACL support enabled. XFS file systems have built-in ACL
support. Other file systems, such as ext3 or ext4 created on Red Hat Enterprise Linux 8, have the
**acl** option enabled by default, although on earlier versions you should confirm that ACL support
is enabled. To enable file-system ACL support, use the ACL option with the **mount** command or in
the file system's entry in **/etc/fstab** configuration file.

#### VIEWING AND INTERPRETING ACL PERMISSIONS

The **ls -l** command only outputs minimal ACL setting details:

```bash
[user@host content]$ ls -l reports.txt
-rwxrw----+ 1 user operators 130 Mar 19 23:56 reports.txt

```

The plus sign (+) at the end of the 10-character permission string indicates that an extended ACL
structure with entries exists on this file.

user:

Shows the user ACL settings, which are the same as the standard user file settings; rwx.

group:

Shows the current ACL mask settings, not the group owner settings; rw.

other:

Shows the other ACL settings, which are the same as the standard other file settings; no
access.

!!! danger "IMPORTANT"

    Changing group permissions on a file with an ACL by using chmod does not change
    the group owner permissions, but does change the ACL mask. Use setfacl -m
    g::perms file if the intent is to update the file's group owner permissions.

**View File ACLs**

To display ACL settings on a file, use getfacl file:

```bash
[user@host content]$ getfacl reports.txt
# file: reports.txt
# owner: user
# group: operators
user::rwx
user:consultant3:---
user:1005:rwx
 #effective:rw-
group::rwx
 #effective:rw-
group:consultant1:r--
group:2210:rwx
 #effective:rw-
mask::rw-
other::---

```
Review each section of the previous example:

Commented entries:

```
# file: reports.txt
# owner: user
# group: operators

```

The first three lines are comments that identify the file name, owner (**user**), and group owner
(**operators**). If there are any additional file flags, such as **setuid** or **setgid**, then a fourth
comment line will appear showing which flags are set.

User entries:

```
user::rwx

user:consultant3:---

user:1005:rwx  #effective:rw-

``` 
`user::rwx`  File owner permissions. **user** has **rwx**.

`user:consultant3:---`   Named user permissions. One entry for each named user associated with this file.
consultant3 has no permissions.

`user:1005:rwx #effective:rw-`   Named user permissions. UID 1005 has rwx, but the mask limits the effective permissions to **rw** only.

Group entries:

```
group::rwx   #effective:rw-

group:consultant1:r--

group:2210:rwx   #effective:rw-
```

`group::rwx   #effective:rw-`    Group owner permissions. operators has rwx, but the mask limits the effective permissions
to rw only.

`group:consultant1:r--`    Named group permissions. One entry for each named group associated with this file.
consultant1 has r only.

`group:2210:rwx   #effective:rw-`    Named group permissions. GID 2210 has rwx, but the mask limits the effective permissions to rw only.

Mask entry:

`mask::rw-`

Mask settings show the maximum permissions possible for all named users, the group owner, and
named groups. UID 1005, operators, and GID 2210 cannot execute this file, even though each
entry has the execute permission set.

Other entry:

`other::---`

Other or "world" permissions. All other UIDs and GIDs have NO permissions.

**Viewing Directory ACLs**

To display ACL settings on a directory, use the **getfacl directory** command:

```bash
[user@host content]$ getfacl .
# file: .
# owner: user
# group: operators
# flags: -s-
user::rwx
user:consultant3:---
user:1005:rwx
group::rwx
group:consultant1:r-x
group:2210:rwx
mask::rwx
other::---
default:user::rwx
default:user:consultant3:---
default:group::rwx
default:group:consultant1:r-x
default:mask::rwx
default:other::---
```
Review each section of the previous example:

Opening comment entries:

```
# file: .
# owner: user
# group: operators
# flags: -s-

```
The first three lines are comments that identify the directory name, owner (user), and group
owner (operators). If there are any additional directory flags (**setuid, setgid, sticky**), then a
fourth comment line shows which flags are set; in this case, **setgid**.

Standard ACL entries:

```
user::rwx
user:consultant3:---
user:1005:rwx
group::rwx
group:consultant1:r-x
group:2210:rwx
mask::rwx
other::---

```
The ACL permissions on this directory are the same as the file example shown earlier, but apply to the directory. The key difference is the inclusion of the execute permission on these entries (when
appropriate) to allow directory search permission.

Default user entries:

```
default:user::rwx
default:user:consultant3:---

```

`default:user::rwx` Default file owner ACL permissions. The file owner will get rwx, read/write on new files and execute on new subdirectories.

`default:user:consultant3:---` Default named user ACL permissions. One entry for each named user who will automatically get the default ACL applied to new files or subdirectories. consultant3 always defaults to no permissions.

Default group entries:

```
default:group::rwx
default:group:consultant1:r-x

```

`default:group::rwx` Default group owner ACL permissions. The file group owner will get **rwx**, read/write on new files and execute on new subdirectories.

`default:group:consultant1:r-x` Default named group ACL permissions. One entry for each named group which will
automatically get the default ACL. consultant1 will get rx, read-only on new files, and execute on new subdirectories.

Default ACL mask entry:

`default:mask::rwx`

Default mask settings show the initial maximum permissions possible for all new files or directories
created that have named user ACLs, the group owner ACL, or named group ACLs: read and write for new files and execute permission on new subdirectories. New files never get execute permission.

Default other entry:

`default:other::---`

Default other or "world" permissions. All other UIDs and GIDs have no permissions to new files or
new subdirectories.

The **default** entries in the previous example do not include the named user (UID **1005**) and
named group (GID **2210**); consequently, they will not automatically get initial ACL entries
added for them to any new files or new subdirectories. This effectively limits them to files and
subdirectories that they already have ACLs on, or if the relevant file owner adds the ACL later
using **setfacl**. They can still create their own files and subdirectories.

!!! info "NOTE"

    The output from **getfacl** command can be used as input to **setfacl** for restoring
    ACLs, or for copying ACLs from a source file or directory and save them into a
    new file. For example, to restore ACLs from a backup, use **getfacl -R /dir1**
    **> file1** to generate a recursive ACL output dump file for the directory and its
    contents. The output can then be used for recovery of the original ACLs, passing
    the saved output as input to the **setfacl** command. For example, to perform a
    bulk update of the same directory in the current path use the following command:
    **setfacl --set-file=file1**

**The ACL Mask**

The ACL mask defines the maximum permissions that you can grant to named users, the group
owner, and named groups. It does not restrict the permissions of the file owner or other users. All
files and directories that implement ACLs will have an ACL mask.

The mask can be viewed with **getfacl** and explicitly set with **setfacl**. It will be calculated and
added automatically if it is not explicitly set, but it could also be inherited from a parent directory
default mask setting. By default, the mask is recalculated whenever any of the affected ACLs are
added, modified, or deleted.

**ACL Permission Precedence**
When determining whether a process (a running program) can access a file, file permissions and
ACLs are applied as follows:

- If the process is running as the user that owns the file, then the file's user ACL permissions
apply.

- If the process is running as a user that is listed in a named user ACL entry, then the named user
ACL permissions apply (as long as it is permitted by the mask).

- If the process is running as a group that matches the group owner of the file, or as a group with
an explicitly named group ACL entry, then the matching ACL permissions apply (as long as it is
permitted by the mask).

- Otherwise, the file's other ACL permissions apply.

### EXAMPLES OF ACL USE BY THE OPERATING SYSTEM

Red Hat Enterprise Linux has examples that demonstrate typical ACL use for extended permission
requirements.

**ACLs on Systemd Journal Files**

systemd-journald uses ACL entries to allow read access to the **/run/log/journal/cb44...8ae2/system.journal** file to the adm and wheel groups. This ACL allows the members of the adm and wheel groups to have read access to the logs managed by **journalctl** without having the need to give special permissions to the privileged content inside **/var/log/**, like **messages, secure** or **audit**.

Due to the systemd-journald configuration, the parent folder of the **system.journal** file
can change, but the systemd-journald applies ACLs to the new folder and file automatically.

!!! info "NOTE"

    System administrators should set an ACL on the **/var/log/journal/** folder when
    systemd-journald is configured to use persistent storage.

```bash
[user@host ]$ getfacl /run/log/journal/cb44...8ae2/system.journal
getfacl: Removing leading '/' from absolute path names
# file: run/log/journal/cb44...8ae2/system.journal
# owner: root
# group: systemd-journal
user::rw-
group::r--
group:adm:r--
group:wheel:r--
mask::r--
other::---

```

**ACL on Systemd Managed Devices**

systemd-udev uses a set of udev rules that enable the uaccess tag to some devices, such as
CD/DVD players or writers, USB storage devices, sound cards, and many others. The previous
mentioned udev rules sets ACLs on those devices to allow users logged in to a graphical user
interface (for example gdm) to have full control of those devices.

The ACLs will remain active until the user logs out of the GUI. The next user who logs in to the GUI
will have a new ACL applied for the required devices.

In the following example, you can see the user has an ACL entry with **rw** permissions applied to
the **/dev/sr0** device that is a CD/DVD drive.

```bash
[user@host ]$ getfacl /dev/sr0
getfacl: Removing leading '/' from absolute path names
# file: dev/sr0
# owner: root
# group: cdrom
user::rw-
user:group:rw-
group::rw-
mask::rw-
other::---
```
### SECURING FILES WITH ACLS

#### CHANGING ACL FILE PERMISSIONS

Use setfacl to add, modify, or remove standard ACLs on files and directories.

ACLs use the normal file system representation of permissions, "**r**" for read permission, "**w**"
for write permission, and "**x**" for execute permission. A "-" (dash) indicates that the relevant
permission is absent. When (recursively) setting ACLs, an uppercase "**X**" can be used to indicate
that execute permission should only be set on directories and not regular files, unless the file
already has the relevant execute permission. This is the same behavior as **chmod**.

**Adding or Modifying ACLs**

ACLs can be set via the command-line using the **-m** option, or passed in via a file using the -
**M** option (use "-" (dash) instead of a file name for **stdin**). These two options are the "modify"
options; they add new ACL entries or replace specific existing ACL entries on a file or directory.
Any other existing ACL entries on the file or directory remain untouched.

!!! info "NOTE"

    Use the **--set** or **--set-file** options to completely replace the ACL settings on a file.

When first defining an ACL on a file, if the add operation does not include settings for the file
owner, group owner, or other permissions, then these will be set based on the current standard
file permissions (these are also known as the base ACL entries and cannot be deleted), and a new
mask value will be calculated and added as well.

To add or modify a user or named user ACL:

```bash
[user@host ~]$ setfacl -m u:name:rX file

```
If name is left blank, then it applies to the file owner, otherwise name can be a username or UID
value. In this example, the permissions granted would be read-only, and if already set, execute
(unless file was a directory, in which case the directory would get the execute permission set to
allow directory search).

ACL file owner and standard file owner permissions are equivalent; consequently, using **chmod** on
the file owner permissions is equivalent to using **setfacl** on the file owner permissions. **chmod**
has no effect on named users.

To add or modify a group or named group ACL:

```bash
[user@host ~]$ setfacl -m g:name:rw file

```

This follows the same pattern for adding or modifying a user ACL entry. If name is left blank, then
it applies to the group owner. Otherwise, specify a group name or GID value for a named group.
The permissions would be read and write in this example.

**chmod** has no effect on any group permissions for files with ACL settings, but it updates the ACL
mask.

To add or modify the other ACL:

```bash
[user@host ~]$ setfacl -m o::- file

```

other only accepts permission settings. Typical permission settings for others are: no permissions
at all, set with a dash (-); and read-only permissions set as usual with **r**. Of course, you can set any
of the standard permissions.

ACL other and standard other permissions are equivalent, so using **chmod** on the other
permissions is equivalent to using **setfacl** on the other permissions.

You can add multiple entries with the same command; use a comma-separated list of entries:

```bash
[user@host ~]$ setfacl -m u::rwx,g:consultants:rX,o::- file

```
This sets the file owner to read, write, and execute, sets the named group **consultants** to read-
only and conditional execute, and restricts all other users to no permissions. The group owner
maintains existing file or ACL permissions and other "named" entries remain unchanged.

**Using getfacl as Input**

You can use the output from **getfacl** as input to **setfacl**:

```bash
[user@host ~]$ getfacl file-A | setfacl --set-file=- file-B

```
The **--set-file** option accepts input from a file or from stdin. The dash character (-) specifies
the use of stdin. In this case, file-B will have the same ACL settings as file-A.

**Setting an Explicit ACL Mask**

You can set an ACL mask explicitly on a file or directory to limit the maximum effective
permissions for named users, the group owner, and named groups. This restricts any existing
permissions that exceed the mask, but does not affect permissions that are less permissive than
the mask.

```bash
[user@host ~]$ setfacl -m m::r file

```

This adds a mask value that restricts any named users, the group owner, and any named groups to
read-only permission, regardless of their existing settings. The file owner and other users are not
impacted by the mask setting.

**getfacl** shows an **effective** comment beside entries that are restricted by a mask setting.

!!! danger "IMPORTANT"

    By default, each time one of the impacted ACL settings (named users, group owner,
    or named groups) is modified or deleted, the ACL mask is recalculated, potentially
    resetting a previous explicit mask setting.

    To avoid the mask recalculation, use the **-n** option or include a mask setting (**-m
    m::perms**) with any **setfacl** operation that modifies mask-affected ACL settings.

**Recursive ACL Modifications**

When setting an ACL on a directory, use the **-R** option to apply the ACL recursively. Remember
that you are likely to want to use the "**X**" (capital X) permission with recursion so that files with the
execute permission set retain the setting and directories get the execute permission set to allow
directory search. It is considered good practice to also use the uppercase "**X**" when non-recursively
setting ACLs because it prevents administrators from accidentally adding execute permissions to
a regular file.

```bash
[user@host ~]$ setfacl -R -m u:name:rX directory

```
This adds the user name to the directory directory and all existing files and subdirectories, setting
read-only and conditional execute permissions.

**Deleting ACLs**

Deleting specific ACL entries follows the same basic format as the modify operation, except that
":perms" is not specified.

```bash
[user@host ~]$ setfacl -x u:name,g:name file

```

This removes only the named user and the named group from the file or directory ACL. Any other
existing ACL entries remain active.

You can include both the delete (**-x**) and modify (**-m**) operations in the same **setfacl** operation.

The mask can only be deleted if there are no other ACLs set (excluding the base ACL which
cannot be deleted), so it must be deleted last. The file will no longer have any ACLs and **ls -l** will
not show the plus sign (+) next to the permissions string. Alternatively, to delete all ACL entries on
a file or directory (including default ACL on directories), use the following command:

```bash
[user@host ~]$ setfacl -b file

```
#### CONTROLLING DEFAULT ACL FILE PERMISSIONS

To ensure that files and directories created within a directory inherit certain ACLs, use the default
ACL on a directory. You can set a default ACL and any of the standard ACL settings, including a
default mask.

The directory itself still requires standard ACLs for access control because the default ACLs do
not implement access control for the directory; they only provide ACL permission inheritance
support. For example:

```bash
[user@host ~]$ setfacl -m d:u:name:rx directory

```

This adds a default named user (**d:u:name**) with read-only permission and execute permission on
subdirectories.

The **setfacl** command for adding a default ACL for each of the ACL types is exactly the same as
for standard ACLs, but prefaced with **d:**. Alternatively, use the **-d** option on the command line.

!!! danger "IMPORTANT"

    When setting default ACLs on a directory, ensure that users will be able to access
    the contents of new subdirectories created in it by including the execute permission
    on the default ACL.

    Users will not automatically get the execute permission set on newly created regular
    files because unlike new directories, the ACL mask of a new regular file is **rw-**.

!!! info "NOTE"

    New files and new subdirectories continue to get their owner UID and primary group
    GID values set from the creating user, except when the parent directory **setgid**
    flag is enabled, in which case the primary group GID is the same as the parent
    directory GID.

**Deleting Default ACL Entries**

Delete a default ACL the same way that you delete a standard ACL, prefacing with **d:**, or use the **-
d** option.

```bash
[user@host ~]$ setfacl -x d:u:name directory

```
This removes the default ACL entry that was added in the previous example.

To delete all default ACL entries on a directory, use **setfacl -k directory**.

### SUMMARY

In this chapter, you learned:

- ACLs provide fine-grained access control to files and directories.

- The **getfacl** command displays the ACLs on a file or directory.

- The **setfacl** command sets, modifies, and removes default and standard ACLs on files and
directories.

- Use default ACLs for controlling new files and directories permissions.

- Red Hat Enterprise Linux uses systemd and udev to apply predefined ACLs on devices, folders,
and files.

### LAB

pdf - 114