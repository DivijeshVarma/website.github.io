---
title: "Accessing The Command Line"
---

### INTRODUCTION TO THE BASH SHELL 

A command line is a text-based interface which can be used to input instructions to a computer
system. The Linux command line is provided by a program called the shell. Various options for the
shell program have been developed over the years, and different users can be configured to use
different shells. Most users, however, stick with the current default.

The default shell for users in Red Hat Enterprise Linux is the ***GNU Bourne-Again Shell (bash)***. Bash
is an improved version of one of the most successful shells used on UNIX-like systems, the ***Bourne
Shell (sh)***.

When a shell is used interactively, it displays a string when it is waiting for a command from the
user. This is called the shell prompt. When a regular user starts a shell, the default prompt ends
with a **$** character, as shown below.

``` bash
[user@host ~]$ 
```

The **$** character is replaced by a ***#*** character if the shell is running as the superuser, root. This
makes it more obvious that it is a superuser shell, which helps to avoid accidents and mistakes
which can affect the whole system. The superuser shell prompt is shown below.

``` bash
[root@host ~]# 
```

Using **bash** to execute commands can be powerful. The bash shell provides a scripting language
that can support automation of tasks. The shell has additional capabilities that can simplify or
make possible operations that are hard to accomplish efficiently with graphical tools.

!!! info "NOTE"

	The **bash** shell is similar in concept to the command-line interpreter found in recent
	versions of Microsoft Windows, cmd.exe, although bash has a more sophisticated
	scripting language. It is also similar to Windows PowerShell in Windows 7 and
	Windows Server 2008 R2 and later. Administrators using the Apple Mac who use the
	Terminal utility may be pleased to note that bash is the default shell in macOS.

### SHELL BASICS ###

Commands entered at the shell prompt have three basic parts:

- Command to run

- Options to adjust the behavior of the command

- Arguments, which are typically targets of the command

The command is the name of the program to run. It may be followed by one or more options, which
adjust the behavior of the command or what it will do. Options normally start with one or two
dashes (-a or --all, for example) to distinguish them from arguments. Commands may also
be followed by one or more arguments, which often indicate a target that the command should
operate upon.

For example, the command **usermod -L user01** has a command (usermod), an option (-L), and
an argument (user01). The effect of this command is to lock the password of the user01 user
account.

### LOGGING IN TO A LOCAL COMPUTER ###

To run the shell, you need to log in to the computer on a terminal. A terminal is a text-based
interface used to enter commands into and print output from a computer system. There are several
ways to do this.

The computer might have a hardware keyboard and display for input and output directly connected
to it. This is the Linux machine's physical console. The physical console supports multiple virtual
consoles, which can run separate terminals. Each virtual console supports an independent login
session. You can switch between them by pressing **Ctrl+Alt** and a function key (F1 through F6)
at the same time. Most of these virtual consoles run a terminal providing a text login prompt, and if
you enter your username and password correctly, you will log in and get a shell prompt.

The computer might provide a graphical login prompt on one of the virtual consoles. You can use
this to log in to a graphical environment. The graphical environment also runs on a virtual console.
To get a shell prompt you must start a terminal program in the graphical environment. The shell
prompt is provided in an application window of your graphical terminal program.

!!! info "NOTE"
    
	Many system administrators choose not to run a graphical environment on their
	servers. This allows resources which would be used by the graphical environment to
	be used by the server's services instead.

In Red Hat Enterprise Linux 8, if the graphical environment is available, the login screen will run
on the first virtual console, called *tty1*. Five additional text login prompts are available on virtual
consoles two through six.

If you log in using the graphical login screen, your graphical environment will start on the first
virtual console that is not currently being used by a login session. Normally, your graphical session
will replace the login prompt on the second virtual console (*tty2*). However, if that console is in
use by an active text login session (not just a login prompt), the next free virtual console is used
instead.

The graphical login screen continues to run on the first virtual console (*tty1*). If you are already
logged in to a graphical session, and log in as another user on the graphical login screen or use the
Switch User menu item to switch users in the graphical environment without logging out, another
graphical environment will be started for that user on the next free virtual console.

When you log out of a graphical environment, it will exit and the physical console will automatically
switch back to the graphical login screen on the first virtual console.

!!! info "NOTE"

	In Red Hat Enterprise Linux 6 and 7, the graphical login screen runs on the first
	virtual console, but when you log in your initial graphical environment replaces the
	login screen on the first virtual console instead of starting on a new virtual console.
	In Red Hat Enterprise Linux 5 and earlier, the first six virtual consoles always
	provided text login prompts. If the graphical environment is running, it is on virtual
	console seven (accessed through **Ctrl+Alt+F7**).

A *headless server* does not have a keyboard and display permanently connected to it. A data center
may be filled with many racks of headless servers, and not providing each with a keyboard and
display saves space and expense. To allow administrators to log in, a headless server might have
a login prompt provided by its serial console, running on a serial port which is connected to a
networked console server for remote access to the serial console.

The *serial console* would normally be used to fix the server if its own network card became
misconfigured and logging in over its own network connection became impossible. Most of the
time, however, headless servers are accessed by other means over the network.

### LOGGING IN OVER THE NETWORK ###

Linux users and administrators often need to get shell access to a remote system by connecting
to it over the network. In a modern computing environment, many headless servers are actually
virtual machines or are running as public or private cloud instances. These systems are not
physical and do not have real hardware consoles. They might not even provide access to their
(simulated) physical console or serial console.

In Linux, the most common way to get a shell prompt on a remote system is to use Secure Shell
(SSH). Most Linux systems (including Red Hat Enterprise Linux) and macOS provide the *OpenSSH*
command-line program **ssh** for this purpose.

In this example, a user with a shell prompt on the machine host uses **ssh** to log in to the remote
Linux system remotehost as the user remoteuser:

``` bash
[user@host ~]$ ssh remoteuser@remotehost
remoteuser@remotehost's password: password
[remoteuser@remotehost ~]$
```

The **ssh** command encrypts the connection to secure the communication against eavesdropping
or hijacking of the passwords and content.

Some systems (such as new cloud instances) do not allow users to use a password to log in with
**ssh** for tighter security. An alternative way to authenticate to a remote machine without entering a
password is through *public key authentication*.

With this authentication method, users have a special identity file containing a private key, which
is equivalent to a password, and which they keep secret. Their account on the server is configured
with a matching public key, which does not have to be secret. When logging in, users can configure
**ssh** to provide the private key and if their matching public key is installed in that account on that
remote server, it will log them in without asking for a password.

In the next example, a user with a shell prompt on the machine host logs in to remotehost
as remoteuser using **ssh**, using public key authentication. The **-i** option is used to specify
the user's private key file, which is mylab.pem. The matching public key is already set up as an
authorized key in the remoteuser account.

``` bash
[user@host ~]$ ssh -i mylab.pem remoteuser@remotehost
[remoteuser@remotehost ~]$
```

For this to work, the private key file must be readable only by the user that owns the file. In the
preceding example, where the private key is in the mylab.pem file, the command **chmod 600
mylab.pem** could be used to ensure this. How to set file permissions is discussed in more detail in
a later chapter.

Users might also have private keys configured that are tried automatically, but that discussion is
beyond the scope of this section. The References at the end of this section contain links to more
information on this topic.

!!! info "NOTE"

	The first time you log in to a new machine, you will be prompted with a warning from
	**ssh** that it cannot establish the authenticity of the host:
	
	``` bash
	[user@host ~]$ ssh -i mylab.pem remoteuser@remotehost
	The authenticity of host 'remotehost (192.0.2.42)' can't be established.
	ECDSA key fingerprint is 47:bf:82:cd:fa:68:06:ee:d8:83:03:1a:bb:29:14:a3.
	Are you sure you want to continue connecting (yes/no)? yes
	[remoteuser@remotehost ~]$
	```
	
	Each time you connect to a remote host with ssh, the remote host sends ssh its
	host key to authenticate itself and to help set up encrypted communication. The ssh
	command compares that against a list of saved host keys to make sure it has not
	changed. If the host key has changed, this might indicate that someone is trying to
	pretend to be that host to hijack the connection which is also known as man-in-the-
	middle attack. In SSH, host keys protect against man-in-the-middle attacks, these
	host keys are unique for each server, and they need to be changed periodically and
	whenever a compromise is suspected.

	You will get this warning if your local machine does not have a host key saved for
	the remote host. If you enter yes, the host key that the remote host sent will be
	accepted and saved for future reference. Login will continue, and you should not see
	this message again when connecting to this host. If you enter no, the host key will be
	rejected and the connection closed.
	
	If the local machine does have a host key saved and it does not match the one
	actually sent by the remote host, the connection will automatically be closed with a
	warning.

### LOGGING OUT ###

When you are finished using the shell and want to quit, you can choose one of several ways to end
the session. You can enter the **exit** command to terminate the current shell session. Alternatively,
finish a session by pressing **Ctrl+D**.

The following is an example of a user logging out of an SSH session:

``` bash
[remoteuser@remotehost ~]$ exit
logout
Connection to remotehost closed.
[user@host ~]$
```

### EXECUTING COMMANDS USING THE BASH SHELL ###

#### BASIC COMMAND SYNTAX ####

The GNU Bourne-Again Shell (**bash**) is a program that interprets commands typed in by the
user. Each string typed into the shell can have up to three parts: the command, options (which
usually begin with a - or --), and arguments. Each word typed into the shell is separated from each
other with spaces. Commands are the names of programs that are installed on the system. Each
command has its own options and arguments.

When you are ready to execute a command, press the **Enter** key. Type each command on a
separate line. The command output is displayed before the next shell prompt appears.

``` bash
[user@host]$ whoami
user
[user@host]$ 
```

If you want to type more than one command on a single line, use the semicolon (;) as a command
separator. A semicolon is a member of a class of characters called metacharacters that have special
meanings for **bash**. In this case the output of both commands will be displayed before the next
shell prompt appears.

The following example shows how to combine two commands (command1 and command2) on the
command line.

``` bash
[user@host]$ command1;command2
```

#### EXAMPLES OF SIMPLE COMMANDS ####

The **date** command displays the current date and time. It can also be used by the superuser to
set the system clock. An argument that begins with a plus sign (+) specifies a format string for the
date command.

``` bash
[user@host ~]$ date
Sat Jan  26 08:13:50 IST 2019
[user@host ~]$ date +%R
08:13
[user@host ~]$ date +%x
01/26/2019
```

The **passwd** command changes a user's own password. The original password for the account
must be specified before a change is allowed. By default, **passwd** is configured to require a strong
password, consisting of lowercase letters, uppercase letters, numbers, and symbols, and is not
based on a dictionary word. The superuser can use the **passwd** command to change other users'
passwords.

``` bash
[user@host ~]$ passwd
Changing password for user user.
Current password: old_password
New password: new_password
Retype new password: new_password
passwd: all authentication tokens updated successfully.
```

Linux does not require file name extensions to classify files by type. The **file** command scans the
beginning of a file's contents and displays what type it is. The files to be classified are passed as
arguments to the command.

``` bash
[user@host ~]$ file /etc/passwd
/etc/passwd: ASCII text
[user@host ~]$ file /bin/passwd
/bin/passwd: setuid ELF 64-bit LSB shared object, x86-64, version 1 
(SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, 
for GNU/Linux 3.2.0, BuildID[sha1]=a3637110e27e9a48dced9f38b4ae43388d32d0e4,
 stripped
[user@host ~]$ file /home
/home: directory
```

### VIEWING THE CONTENTS OF FILES ###

One of the most simple and frequently used commands in Linux is **cat**. The **cat** command allows
you to create single or multiple files, view the contents of files, concatenate the contents from
multiple files, and redirect contents of the file to a terminal or files.

The example shows how to view the contents of the **/etc/passwd** file.

``` bash
[user@host ~]$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
...output omitted...
```

Use the following command to display the contents of multiple files.

``` bash
[user@host ~]$ cat file1 file2
Hello World!!
Introduction to Linux commands.
```

Some files are very long and can take up more room to display than that provided by the terminal.
The **cat** command does not display the contents of a file as pages. The **less** command displays
one page of a file at a time and lets you scroll at your leisure.

The **less** command allows you to page forward and backward through files that are longer than
can fit on one terminal window. Use the UpArrow key and the DownArrow key to scroll up and
down. Press Q to exit the command.

The **head** and **tail** commands display the beginning and end of a file, respectively. By default
these commands display 10 lines of the file, but they both have a **-n** option that allows a different
number of lines to be specified. The file to display is passed as an argument to these commands.

``` bash
[user@host ~]$ head /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
[user@host ~]$ tail -n 3 /etc/passwd
gdm:x:42:42::/var/lib/gdm:/sbin/nologin
gnome-initial-setup:x:977:977::/run/gnome-initial-setup/:/sbin/nologin
avahi:x:70:70:Avahi mDNS/DNS-SD Stack:/var/run/avahi-daemon:/sbin/nologin
```

The **wc** command counts lines, words, and characters in a file. It takes a -l, -w, or -c option to

``` bash
[user@host ~]$ wc /etc/passwd
  45  102 2480 /etc/passwd
[user@host ~]$ wc -l /etc/passwd ; wc -l /etc/group
45 /etc/passwd
70 /etc/group
[user@host ~]$ wc -c /etc/group /etc/hosts
 966 /etc/group
 516 /etc/hosts
1482 total
```

### CONTINUING A LONG COMMAND ON ANOTHER LINE ###

Commands that support many options and arguments can quickly grow quite long and are
automatically scrolled by the Bash shell. As soon as the cursor reaches the right margin of the
window, the command continues on the next line. To make the readability of the command easier,
you can break it up so that if fits on more than one line.

To do this, add a backslash character (\) as the last character on the line. This tells the shell to
ignore the newline character and treat the next line as if it were part of the current line. The Bash
shell will start the next line with the continuation prompt, usually a greater-than character (>),
which indicates that the line is a continuation of the previous line. You can do this more than once.

``` bash
[user@host]$ head -n 3 \
> /usr/share/dict/words \
> /usr/share/dict/linux.words
==> /usr/share/dict/words <==
1080
10-point
10th
==> /usr/share/dict/linux.words <==
1080
10-point
10th
[user@host ~]$ 
```

### COMMAND HISTORY ###

The **history** command displays a list of previously executed commands prefixed with a command
number.

The exclamation point character (!) is a metacharacter that is used to expand previous commands
without having to retype them. The **!number** command expands to the command matching the
number specified. The **!string** command expands to the most recent command that begins with
the string specified.

``` bash
[user@host ~]$ history
   ...output omitted...
   23  clear
   24  who
   25  pwd
   26  ls /etc
   27  uptime
   28  ls -l
   29  date
   30  history
[user@host ~]$ !ls
ls -l
total 0
drwxr-xr-x. 2 user user 6 Mar 29 21:16 Desktop
...output omitted...
[user@host ~]$ !26
ls /etc
abrt                     hosts                     pulse
adjtime                  hosts.allow               purple
aliases                  hosts.deny                qemu-ga
...output omitted...
```

The arrow keys can be used to navigate through previous commands in the shell's history.
**UpArrow** edits the previous command in the history list. **DownArrow** edits the next command
in the history list. **LeftArrow** and **RightArrow** move the cursor left and right in the current
command from the history list, so that you can edit it before running it.

You can use either the **Esc+**. or **Alt+**. key combination to insert the last word of the previous
command at the cursor's current location. Repeated use of the key combination will replace that
text with the last word of even earlier commands in the history. The Alt+. key combination
is particularly convenient because you can hold down Alt and press . repeatedly to easily go
through earlier and earlier commands.

### EDITING THE COMMAND LINE ###

When used interactively, **bash** has a command-line editing feature. This allows the user to use text
editor commands to move around within and modify the current command being typed. Using the
arrow keys to move within the current command and to step through the command history was
introduced earlier in this session. More powerful editing commands are introduced in the following
table.

**Useful Command-line Editing Shortcuts**

|SHORTCUT | DESCRIPTION |
|-------:|:-----------|
|Ctrl+A   | Jump to the beginning of the command line. |
|Ctrl+E   | Jump to the end of the command line. |
|Ctrl+U   |  Clear from the cursor to the beginning of the command line.
|Ctrl+K   |  Clear from the cursor to the end of the command line.
|Ctrl+Left | Arrow Jump to the beginning of the previous word on the command line.
|Ctrl+Right | Arrow Jump to the end of the next word on the command line.
|Ctrl+R     | Search the history list of commands for a pattern.

There are several other command-line editing commands available, but these are the most useful
commands for new users. The other commands can be found in the **bash(1)** man page.

### ACCESSING THE COMMAND LINE ###

- Use the date command to display the current time and date.

``` bash
[student@workstation ~]$ date
Thu Jan 22 10:13:04 PDT 2019
```

- Display the current time in 12-hour clock time (for example, 11:42:11 AM). Hint: The format
string that displays that output is %r.

   Use the +%r argument with the date command to display the current time in 12-hour clock
   time.

``` bash
[student@workstation ~]$ date +%r
10:14:07 AM
```

- What kind of file is /home/student/zcat? Is it readable by humans?
Use the file command to determine its file type.

``` bash
[student@workstation ~]$ file zcat
zcat: POSIX shell script, ASCII text executable
```

- Use the wc command and Bash shortcuts to display the size of zcat.

The wc command can be used to display the number of lines, words, and bytes in the zcat
script. Instead of retyping the file name, use the Bash history shortcut Esc+. (the keys Esc
and . pressed at the same time) to reuse the argument from the previous command.

``` bash
[student@workstation ~]$ wc Esc+.
[student@workstation ~]$ wc zcat
```

- Display the first 10 lines of zcat.
The head command displays the beginning of the file. Try using the Esc+. shortcut again.

``` bash
[student@workstation ~]$ head Esc+.
[student@workstation ~]$ head zcat
#!/bin/sh
# Uncompress files to standard output.
# Copyright (C) 2007, 2010-2018 Free Software Foundation, Inc.
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3 of the License, or
# (at your option) any later version.
```

- Display the last 10 lines of the zcat file.
Use the tail command to display the last 10 lines of the zcat file.

``` bash
[student@workstation ~]$ tail Esc+.
[student@workstation ~]$ tail zcat
With no FILE, or when FILE is -, read standard input.
Report bugs to <bug-gzip@gnu.org>."
case $1 in
--help)    printf '%s\n' "$usage"   || exit 1;;
--version) printf '%s\n' "$version" || exit 1;;
esac
exec gzip -cd "$@"
```

- Repeat the previous command exactly with three or fewer keystrokes.

Repeat the previous command exactly. Either press the UpArrow key once to scroll back
through the command history one command and then press Enter (uses two keystrokes),
or enter the shortcut command !! and then press Enter (uses three keystrokes) to run the
most recent command in the command history . (Try both.)

``` bash
[student@workstation]$ !!
tail zcat
With no FILE, or when FILE is -, read standard input.
Report bugs to <bug-gzip@gnu.org>."
case $1 in
--help)    printf '%s\n' "$usage"   || exit 1;;
--version) printf '%s\n' "$version" || exit 1;;
esac
exec gzip -cd "$@"
```

- Repeat the previous command, but use the -n 20 option to display the last 20 lines in the
file. Use command-line editing to accomplish this with a minimal number of keystrokes.
UpArrow displays the previous command. Ctrl+A makes the cursor jump to the beginning
of the line. Ctrl+RightArrow jumps to the next word, then add the -n 20 option and hit
Enter to execute the command.

``` bash
[student@workstation ~]$ tail -n 20 zcat
  -l, --list        list compressed file contents
  -q, --quiet       suppress all warnings
  -r, --recursive   operate recursively on directories
  -S, --suffix=SUF  use suffix SUF on compressed files
      --synchronous synchronous output (safer if system crashes, but slower)
  -t, --test        test compressed file integrity
  -v, --verbose     verbose mode
      --help        display this help and exit
      --version     display version information and exit
With no FILE, or when FILE is -, read standard input.
Report bugs to <bug-gzip@gnu.org>."
case $1 in
--help)    printf '%s\n' "$usage"   || exit 1; exit;;
--version) printf '%s\n' "$version" || exit 1; exit;;
esac
exec gzip -cd "$@"
```

- Use the shell history to run the date +%r command again.
Use the history command to display the list of previous commands and to identify the
specific date command to be executed. Use !number to run the command, where number is
the command number to use from the output of the history command.
Note that your shell history may be different from the following example. Determine the
command number to use based on the output of your own history command.

``` bash
[student@workstation ~]$ history
1   date
2   date +%r
3   file zcat
4   wc zcat
5   head zcat
6   tail zcat
7   tail -n 20 zcat
8   history
[student@workstation ~]$ !2
date +%r
10:49:56 AM
```

### SUMMARY ###

In this chapter, you learned:

- The Bash shell is a command interpreter that prompts interactive users to specify Linux
commands.

- Many commands have a **--help** option that displays a usage message or screen.

- Using workspaces makes it easier to organize multiple application windows.

- The Activities button located at the upper-left corner of the top bar provides an overview mode
that helps a user organize windows and start applications.

- The **file** command scans the beginning of a file's contents and displays what type it is.

- The **head** and **tail** commands display the beginning and end of a file, respectively.

- You can use **Tab** completion to complete file names when typing them as arguments to
commands.
