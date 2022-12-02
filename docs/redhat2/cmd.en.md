---
title: "Improving Command-Line Productivity"
---

### WRITING SIMPLE BASH SCRIPTS

#### CREATING AND EXECUTING BASH SHELL SCRIPTS

Many simple, common system administration tasks are accomplished using command-line tools.
Tasks with greater complexity often require chaining together multiple commands that pass
results between them. Using the Bash shell environment and scripting features, Linux commands
are combined into shell scripts to easily solve repetitive and difficult real-world problems.

In its simplest form, a Bash shell script is an executable file that contains a list of commands, and
possibly with programming logic to control decision-making in the overall task. When well-written,
a shell script is a powerful command-line tool on its own, and can be leveraged by other scripts.

Shell scripting proficiency is essential to successful system administration in any operational
environment. Working knowledge of shell scripting is crucial in enterprise environments, where
script use can improve the efficiency and accuracy of routine task completion.

You can create a Bash shell script by opening a new empty file in a text editor. While you can
use any text editor, advanced editors, such as **vim or emacs**, understand Bash shell syntax and
can provide **color-coded** highlighting. This highlighting helps identify common errors such as
improper syntax, unpaired quotes, unclosed parenthesis, braces, and brackets, and much more.

**Specifying the Command Interpreter**

The first line of a script begins with the notation **'#!'**, commonly referred to as **sh-bang** or **she-
bang**, from the names of those two characters, **sharp** and **bang**. This specific two-byte **magic
number** notation indicates an interpretive script; syntax that follows the notation is the fully-
qualified filename for the correct **command interpreter** needed to process this script's lines.
To understand how **magic numbers** indicate file types in Linux, see the **file(1)** and **magic(5)**
man pages. For script files using Bash scripting syntax, the first line of a shell script begins as
follows:

```bash
#!/bin/bash
```
**Executing a Bash Shell Script**

A completed shell script must be executable to run as an ordinary command. Use the **chmod**
command to add execute permission, possibly in conjunction with the **chown** command to change
the file ownership of the script. Grant execute permission only for intended users of the script.

If you place the script in one of the directories listed in the shell's PATH environmental variable,
then you can invoke the shell script using the file name alone as with any other command. The shell
uses the first command it finds with that file name; avoid using existing command names for your
shell script file name. Alternatively, you can invoke a shell script by entering a path name to the
script on the command line. The **which** command, followed by the file name of the executable
script, displays the path name to the command that will be executed.

```bash
[user@host ~]$ which hello
~/bin/hello
[user@host ~]$ echo $PATH
/home/user/.local/bin:/home/user/bin:/usr/share/Modules/bin:/usr/local/bin:/usr/
bin:/usr/local/sbin:/usr/sbin
```

**Quoting Special Characters**

A number of characters and words have special meaning to the Bash shell. However, occasionally
you will want to use these characters for their literal values, rather than for their special meanings.
To do this, use one of three tools to remove (or escape) the special meaning: the backslash (\),
single quotes (''), or double quotes ("").

The backslash escape character removes the special meaning of the single character immediately
following it. For example, to display the literal string # **not a comment** with the **echo** command,
the # sign must not be interpreted by Bash as having special meaning. Place the backslash
character in front of the # sign.

```bash
[user@host ~]$ echo # not a comment
[user@host ~]$ echo \# not a comment
# not a comment

```

When you need to escape more than one character in a text string, either use the escape
character multiple times or employ single quotes (''). Single quotes preserve the literal meaning of
all characters they enclose. Observe the escape character and single quotes in action:

```bash
[user@host ~]$ echo # not a comment #
[user@host ~]$ echo \# not a comment #
# not a comment
[user@host ~]$ echo \# not a comment \#
# not a comment #
[user@host ~]$ echo '# not a comment #'
# not a comment #

```

Use double quotation marks to suppress globbing and shell expansion, but still allow command
and variable substitution. Variable substitution is conceptually identical to command substitution,
but may use optional brace syntax. Observe the examples of various forms of quotation mark use
below.

Use single quotation marks to interpret all text literally. Besides suppressing globbing and shell
expansion, quotations direct the shell to additionally suppress command and variable substitution.
The question mark (?) is a **meta-character** that also needs protection from expansion.

```bash
[user@host ~]$ var=$(hostname -s); echo $var
host
[user@host ~]$ echo "***** hostname is ${var} *****"
***** hostname is host *****
[user@host ~]$ echo Your username variable is \$USER.
Your username variable is $USER.
[user@host ~]$ echo "Will variable $var evaluate to $(hostname -s)?"
Will variable host evaluate to host?
[user@host ~]$ echo 'Will variable $var evaluate to $(hostname -s)?'
Will variable $var evaluate to $(hostname -s)?
[user@host ~]$ echo "\"Hello, world\""
"Hello, world"
[user@host ~]$ echo '"Hello, world"'
"Hello, world"
```
**Providing Output From a Shell Script**

The echo command displays arbitrary text by passing the text as an argument to the command.
By default, the text displays on standard output (STDOUT), but it can also be directed to standard
error (STDERR) using output redirection. In the following simple Bash script, the **echo** command
displays the message "**Hello, world**" to STDOUT.

```bash
[user@host ~]$ cat ~/bin/hello
#!/bin/bash
echo "Hello, world"
[user@host ~]$ hello
Hello, world

```
!!! info "NOTE"

	This user can just run **hello** at the prompt because the **~/bin** (**/home/user/bin**)
	directory is in the user's PATH variable and the **hello** script in it is executable. The
	shell automatically finds the script there, as long as there is no other executable file
	called **hello** in any of the directories listed prior to **/home/user/bin** in the PATH.


The **echo** command is widely used in shell scripts to display informational or error messages.
These messages can be a helpful indicator of the progress of a script and can be directed either to
standard output, standard error, or be redirected to a log file for archiving. When displaying error
messages, it is good practice to direct them to STDERR to make it easier to differentiate error
messages from normal status messages.

```bash
[user@host ~]$ cat ~/bin/hello
#!/bin/bash
echo "Hello, world"
echo "ERROR: Houston, we have a problem." >&2
[user@host ~]$ hello 2> hello.log
Hello, world
[user@host ~]$ cat hello.log
ERROR: Houston, we have a problem.

```
The **echo** command can also be very helpful when trying to debug a problematic shell script. The
addition of **echo** statements to the portion of the script that is not behaving as expected can help
clarify the commands being executed, as well as the values of variables being invoked.

### RUNNING COMMANDS MORE EFFICIENTLY USING LOOPS

#### USING LOOPS TO ITERATE COMMANDS

System administrators often encounter repetitive tasks in their day-to-day activities. Repetitive
tasks can take the form of executing an action multiple times on a target, such as checking a
process every minute for 10 minutes to see if it has completed. Task repetition can also take the
form of executing an action once across multiple targets, such as backing up each database on a
system. The for loop is one of the multiple shell looping constructs offered by Bash, and can be
used for task iterations.

**Processing Items from the Command Line**

Bash's for loop construct uses the following syntax.

```
for VARIABLE in LIST; do
COMMAND VARIABLE
done

```
The loop processes the strings provided in LIST in order one by one and exits after processing the
last string in the list. Each string in the list is temporarily stored as the value of VARIABLE, while the
**for** loop executes the block of commands contained in its construct. The naming of the variable is
arbitrary. Typically, the variable value is referenced by commands in the command block.

The list of strings provided to a **for** loop can be supplied in several ways. It can be a list of
strings entered directly by the user, or be generated from different types of shell expansion,
such as variable, brace, or file-name expansion, or command substitution. Some examples that
demonstrate the different ways strings can be provided to **for** loops follow.

```bash
[user@host ~]$ for HOST in host1 host2 host3; do echo $HOST; done
host1
host2
host3
[user@host ~]$ for HOST in host{1,2,3}; do echo $HOST; done
host1
host2
host3
[user@host ~]$ for HOST in host{1..3}; do echo $HOST; done
host1
host2
host3
[user@host ~]$ for FILE in file*; do ls $FILE; done
filea
fileb
filec
[user@host ~]$ for FILE in file{a..c}; do ls $FILE; done
filea
fileb
filec
[user@host ~]$ for PACKAGE in $(rpm -qa | grep kernel); \
do echo "$PACKAGE was installed on \
$(date -d @$(rpm -q --qf "%{INSTALLTIME}\n" $PACKAGE))"; done
abrt-addon-kerneloops-2.1.11-12.el7.x86_64 was installed on Tue Apr 22 00:09:07
EDT 2014
kernel-3.10.0-121.el7.x86_64 was installed on Thu Apr 10 15:27:52 EDT 2014
kernel-tools-3.10.0-121.el7.x86_64 was installed on Thu Apr 10 15:28:01 EDT 2014
kernel-tools-libs-3.10.0-121.el7.x86_64 was installed on Thu Apr 10 15:26:22 EDT
2014
[user@host ~]$ for EVEN in $(seq 2 2 10); do echo "$EVEN"; done
2
4
6
8
10
```