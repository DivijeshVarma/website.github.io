---
title: "Getting Help In Red Hat Enterprise Linux"
---

### READING MANUAL PAGES ###

#### INTRODUCING THE MAN COMMAND ####

One source of documentation that is generally available on the local system are system manual
pages or man pages These pages are shipped as part of the software packages for which they
provide documentation, and can be accessed from the command line by using the **man** command.

The historical Linux Programmer's Manual, from which man pages originate, was large enough to
be multiple printed sections. Each section contains information about a particular topic.

**Common Sections of the Linux Manual**

| SECTION | CONTENT TYPE                                                        |
|:-------:|:--------------------------------------------------------------------|
| 1       | User commands (both executable and shell programs)                  |
| 2       | System calls (kernel routines invoked from user space)              |
| 3       | Library functions (provided by program libraries)                   |
| 4       | Special files (such as device files)                                |
| 5       | File formats (for many configuration files and structures)          |
| 6       | Games (historical section for amusing programs)                     |
| 7       | Conventions, standards, and miscellaneous (protocols, file systems) |
| 8       | System administration and privileged commands (maintenance tasks)   |
| 9       | Linux kernel API (internal kernel calls)                            |

To distinguish identical topic names in different sections, man page references include the section
number in parentheses after the topic. For example, **passwd**(1) describes the command to change
passwords, while **passwd**(5) explains the **/etc/passwd** file format for storing local user accounts.

To read specific man pages, use **man topic**. Contents are displayed one screen at a time. The man
command searches manual sections in alphanumeric order. For example, **man passwd** displays
passwd(1) by default. To display the man page topic from a specific section, include the section
number argument: **man 5 passwd** displays passwd(5).

### NAVIGATE AND SEARCH MAN PAGES ###

The ability to efficiently search for topics and navigate man pages is a critical administration
skill. GUI tools make it easy to configure common system resources, but using the command-line
interface is still more efficient. To effectively navigate the command line, you must be able to find
the information you need in man pages.

The following table lists basic navigation commands when viewing man pages:

**Navigating Man Pages**

| COMMAND   | RESULT                                                |
|:---------:|:------------------------------------------------------|
| Spacebar  | Scroll forward (down) one screen                      |
| PageDown  | Scroll forward (down) one screen                      |
| PageUp    | Scroll backward (up) one screen                       |
| DownArrow | Scroll forward (down) one line                        |
| UpArrow   | Scroll backward (up) one line                         |
| D         | Scroll forward (down) one half-screen                 |
| U         | Scroll backward (up) one half-screen                  |
| /string   | Search forward (down) for string in the man page      |
| N         | Repeat previous search forward (down) in the man page |
| Shift+N   | Repeat previous search backward (up) in the man page  |
| G         | Go to start of the man page.                          |
| Shift+G   | Go to end of the man page.                            |
| Q         | Exit **man** and return to the command shell prompt   |

!!! danger "IMPORTANT"

	When performing searches, string allows regular expression syntax. While simple text
	(such as passwd) works as expected, regular expressions use meta-characters (such
	as $, *, ., and ^) for more sophisticated pattern matching. Therefore, searching with
	strings that include program expression meta-characters, such as make $$$, might
	yield unexpected results.
	
	Regular expressions and syntax are discussed in Red Hat System Administration II,
	and in the regex(7) man topic.

**Reading Man Pages**

Each topic is separated into several parts. Most topics share the same headings and are presented
in the same order. Typically a topic does not feature all headings, because not all headings apply
for all topics.

Common headings are:

| HEADING     | DESCRIPTION                                                            |
|:-----------:|:-----------------------------------------------------------------------|
| NAME        | Subject name. Usually a command or file name. Very brief description.  |
| SYNOPSIS    | Summary of the command syntax.                                         |
| DESCRIPTION | In-depth description to provide a basic understanding of the topic.    |
| OPTIONS     | Explanation of the command execution options.                          |
| EXAMPLES    | Examples of how to use the command, function, or file.                 |
| FILES       | A list of files and directories related to the man page.               |
| SEE ALSO    | Related information, normally other man page topics.                   |
| BUGS        | Known bugs in the software.                                            |
| AUTHOR      | Information about who has contributed to the development of the topic. |

### SEARCHING FOR MAN PAGES BY KEYWORD ###

A keyword search of man pages is performed with **man -k keyword**, which displays a list of
keyword-matching man page topics with section numbers.

``` bash
[student@desktopX ~]$ man -k passwd
checkPasswdAccess (3) - query the SELinux policy database in the kernel.
chpasswd (8)          - update passwords in batch mode
ckpasswd (8)          - nnrpd password authenticator
fgetpwent_r (3)       - get passwd file entry reentrantly
getpwent_r (3)        - get passwd file entry reentrantly
...
passwd (1)            - update user's authentication tokens
sslpasswd (1ssl)      - compute password hashes
passwd (5)            - password file
passwd.nntp (5)       - Passwords for connecting to remote NNTP servers
passwd2des (3)        - RFS password encryption
...
```


Popular system administration topics are in sections 1 (user commands), 5 (file formats), and 8
(administrative commands). Administrators using certain troubleshooting tools also use section
2 (system calls). The remaining sections are generally for programmer reference or advanced
administration.

!!! info "Note"

	Keyword searches rely on an index generated by the mandb(8) command, which
	must be run as root. The command runs daily through **cron.daily**, or by
	**anacrontab** within an hour of boot, if out of date.


!!! danger "IMPORTANT"

	The man command -K (uppercase) option performs a full-text page search, not just
	titles and descriptions like the -k option. A full-text search uses greater system
	resources and take more time.

### READING INFO DOCUMENTATION ###

#### INTRODUCING GNU INFO ####

Man pages have a format useful as a command reference, but less useful as general
documentation. For such documents, the GNU Project developed a different online documentation
system, known as *GNU Info*. Info documents are an important resource on a Red Hat Enterprise
Linux system because many fundamental components and utilities, such as the *coreutils* package
and *glibc* standard libraries, are either developed by the GNU Project or utilize the Info document
system.

!!! danger "IMPORTANT"

	You might wonder why there are two local documentation systems, man pages and
	Info documents. Some of the reasons for this are practical in nature, and some have
	to do with the way Linux and its applications have been developed by various open
	source communities over the years.
	
	Man pages have a much more formal format, and typically document a specific
	command or function from a software package, and are structured as individual text
	files. Info documents typically cover particular software packages as a whole, tend
	to have more practical examples of how to use the software, and are structured as
	hypertext documents.
	
	You should be familiar with both systems in order to take maximum advantage of the
	information available to you from the system.

**Reading Info Documentation**

To launch the Info document viewer, use the *pinfo* command. *pinfo* opens in the top directory.

![pinfo](https://www.linuxprobe.com/links/RH124/images/help/pinfo.png "pinfo info document viewer, top directory")

Info documentation is comprehensive and hyperlinked. It is possible to output info pages to
multiple formats. By contrast, man pages are optimized for printed output. The Info format is more
flexible than man pages, allowing thorough discussion of complex commands and concepts. Like
man pages, Info nodes are read from the command line, using the *pinfo* command.

A typical man page has a small amount of content focusing on one particular topic, command,
tool, or file. The Info documentation is a comprehensive document. Info provides the following
improvements:

- One single document for a large system containing all the necessary information for that system

- Hyperlinks

- A complete browsable document index

- A full text search of the entire document

Some commands and utilities have both *man* pages and info documentation; usually, the Info
documentation is more in depth. Compare the differences in *tar* documentation using *man* and
*pinfo*:

``` bash
[user@host ~]$ man tar
[user@host ~]$ pinfo tar
```

The *pinfo* reader is more advanced than the original info command. To browse a specific topic,
use the pinfo topic command. The pinfo command without an argument opens the top
directory. New documentation becomes available in pinfo when their software packages are
installed.

!!! info "NOTE"

	If no Info topic exists in the system for a particular entry that you requested, Info will
	look for a matching man page and display that instead.

### COMPARING GNU INFO AND MAN PAGE NAVIGATION ###

The **pinfo** command and the **man** command use slightly different navigational keystrokes. The
following table compares the navigational keystrokes for both commands:

**pinfo and man, key binding comparison**

| NAVIGATION                                 | PINFO             | MAN                |
|:------------------------------------------:|:------------------|--------------------|
| Scroll forward (down) one screen           | PageDown or Space | PageDown or Space  |
| Scroll backward (up) one screen            | PageUp or b       | PageUp or b        |
| Display the directory of topics            | D                 | -                  |
| Scroll forward (down) one half-screen      | -                 | D                  |
| Display the parent node of a topic         | U                 | -                  |
| Display the top (up) of a topic            | HOME              | G                  |
| Scroll backward (up) one half-screen       | -                 | U                  |
| Scroll forward (down) to next hyperlink    | DownArrow         | -                  |
| Open topic at cursor location              | ENTER             | -                  |
| Scroll forward (down) one line             | -                 | DownArrow or Enter |
| Scroll backward (up) to previous hyperlink | UpArrow           | -                  |
| Scroll backward (up) one line              | -                 | UpArrow            |
| Search for a pattern                       | /string           | /string            |
| Display next node (chapter) in topic       | N                 | -                  |
| Repeat previous search forward (down)      | / then Enter      | n                  |
| Display previous node (chapter) in topic   | P                 | -                  |
| Repeat previous search backward (up)       | -                 | ShiftN             |
| Quit the program                           | Q                 | Q                  |


### SUMMARY ###

In this chapter, you learned:

- Man pages are viewed with the **man** command and provide information on components of a Linux
system, such as files, commands, and functions.

- By convention, when referring to a man page the name of a page is followed by its section
number in parentheses.

- Info documents are viewed with the **pinfo** command and are made up of a collection of
hypertext nodes, providing information about software packages as a whole.

- The navigational keystrokes used by **man** and **pinfo** are slightly different.

### LAB

pdf - 143
