---
title: "Creating, Viewing, And Editing Text Files"
---

### REDIRECTING OUTPUT TO A FILE OR PROGRAM ###

#### STANDARD INPUT, STANDARD OUTPUT, AND STANDARD ERROR ####

A running program, or *process*, needs to read input from somewhere and write output to
somewhere. A command run from the shell prompt normally reads its input from the keyboard and
sends its output to its terminal window.

A process uses numbered channels called *file descriptors* to get input and send output. All
processes start with at least three file descriptors. *Standard input* (channel 0) reads input from the
keyboard. *Standard output* (channel 1) sends normal output to the terminal. *Standard error* (channel
2) sends error messages to the terminal. If a program opens separate connections to other files, it
may use higher-numbered file descriptors.

![Process I/O channels (file descriptors)](https://www.thegeeksearch.com/images/post/wp-content-uploads-2020-10-linux-file-redirection_hub4aac9916d24ff7778403105fc9a6da8_20977_750x0_resize_q100_h2_box.webp "Process I/O channels (file descriptors)")

**Channels (File Descriptors)**

| NUMBER | CHANNEL NAME | DESCRIPTION     | DEFAULT CONNECTION | USAGE             |
|:------:|:-------------|-----------------|--------------------|-------------------|
| 0      | stdin        | Standard input  | Keyboard           | read only         |
| 1      | stdout       | Standard output | Terminal           | write only        |
| 2      | stderr       | Standard error  | Terminal           | write only        |
| 3+     | filename     | Other files     | none               | read and/or write |

### REDIRECTING OUTPUT TO A FILE ###

*I/O* redirection changes how the process gets its input or output. Instead of getting input from
the keyboard, or sending output and errors to the terminal, the process reads from or writes to
files. Redirection lets you save messages to a file that are normally sent to the terminal window.
Alternatively, you can use redirection to discard output or errors, so they are not displayed on the
terminal or saved.

Redirecting **stdout** suppresses process output from appearing on the terminal. As seen in
the following table, redirecting only **stdout** does not suppress **stderr** error messages from
displaying on the terminal. If the file does not exist, it will be created. If the file does exist and the
redirection is not one that appends to the file, the file's contents will be overwritten.
If you want to discard messages, the special file **/dev/null** quietly discards channel output
redirected to it and is always an empty file.

| USAGE         | EXPLANATION                                           |
|:-------------:|:------------------------------------------------------|
| > file        | redirect stdout to overwrite a file                   |
| >> file       | redirect stdout to append to a file                   |
| 2> file       | redirect stderr to overwrite a file                   |
| 2> /dev/null  | stderr error messages by redirecting to /dev/null     |
| > file  2>&1  | redirect stdout and stderr to overwrite the same file |
| &> file       | redirect stdout and stderr to overwrite the same file |
| >> file  2>&1 | redirect stdout and stderr to append to the same file |
| &>> file      | redirect stdout and stderr to append to the same file |

!!! danger "IMPORTANT"

	The order of redirection operations is important. The following sequence redirects
	standard output to file and then redirects standard error to the same place as
	standard output (file).
	
	`> file 2>&1`
	
	However, the next sequence does redirection in the opposite order. This redirects
	standard error to the default place for standard output (the terminal window, so no
	change) and then redirects only standard output to file.
	
	`2>&1 > file`
	
	Because of this, some people prefer to use the merging redirection operators:
	
	`&>file instead of >file 2>&1`
	
	`&>>file instead of >>file 2>&1(in Bash 4 / RHEL 6 and later)`
	
	However, other system administrators and programmers who also use other shells
	related to **bash** (known as Bourne-compatible shells) for scripting commands think
	that the newer merging redirection operators should be avoided, because they are
	not standardized or implemented in all of those shells and have other limitations.
	The authors of this course take a neutral stance on this topic, and both syntaxes are
	likely to be encountered in the field.

**Examples for Output Redirection**

Many routine administration tasks are simplified by using redirection. Use the previous table to
assist while considering the following examples:

- Save a time stamp for later reference.

``` bash
[user@host ~]$ date > /tmp/saved-timestamp
```

- Copy the last 100 lines from a log file to another file.

``` bash
[user@host ~]$ tail -n 100 /var/log/dmesg > /tmp/last-100-boot-messages
```

- Concatenate four files into one.

``` bash
[user@host ~]$ cat file1 file2 file3 file4 > /tmp/all-four-in-one
```

- List the home directory's hidden and regular file names into a file.

``` bash
[user@host ~]$ ls -a > /tmp/my-file-names
```

- Append output to an existing file.

``` bash
[user@host ~]$ echo "new line of information" >> /tmp/many-lines-of-information
[user@host ~]$ diff previous-file current-file >> /tmp/tracking-changes-made
```

- The next few commands generate error messages because some system directories are
inaccessible to normal users. Observe as the error messages are redirected. Redirect errors to a
file while viewing normal command output on the terminal.

``` bas
[user@host ~]$ find /etc -name passwd 2> /tmp/errors
```

- Save process output and error messages to separate files.

``` bash
[user@host ~]$ find /etc -name passwd > /tmp/output 2> /tmp/errors
```

- Ignore and discard error messages.

``` bash
[user@host ~]$ find /etc -name passwd > /tmp/output 2> /dev/null
```

- Store output and generated errors together.

``` bash
[user@host ~]$ find /etc -name passwd &> /tmp/save-both
```

- Append output and generated errors to an existing file.

``` bash
[user@host ~]$ find /etc -name passwd >> /tmp/save-both 2>&1
```

### CONSTRUCTING PIPELINES ###

A **pipeline** is a sequence of one or more commands separated by the pipe character (|). A pipe
connects the standard output of the first command to the standard input of the next command.

![Process I/O piping](https://www.thegeeksearch.com/images/post/wp-content-uploads-2020-10-linux-pipelines-basics_hu35331ae3cd8d08fad9a0ab1501e7bdf9_7345_623x0_resize_q100_h2_box.webp "Process I/O piping")

Pipelines allow the output of a process to be manipulated and formatted by other processes before
it is output to the terminal. One useful mental image is to imagine that data is "flowing" through
the pipeline from one process to another, being altered slightly by each command in the pipeline
through which it flows.

!!! info "NOTE"

	Pipelines and I/O redirection both manipulate standard output and standard input.
	Redirection sends standard output to files or gets standard input from files. Pipes
	send the standard output from one process to the standard input of another process.

**Pipeline Examples**

This example takes the output of the *ls* command and uses less to display it on the terminal one
screen at a time.

``` bash
[user@host ~]$ ls -l /usr/bin | less
```

The output of the ls command is piped to *wc -l*, which counts the number of lines received from
ls and prints that to the terminal.

``` bash
[user@host ~]$ ls | wc -l
```

In this pipeline, *head* will output the first 10 lines of output from *ls -t*, with the final result
redirected to a file.

``` bash
[user@host ~]$ ls -t | head -n 10 > /tmp/ten-last-changed-files
```

**Pipelines, Redirection, and the tee Command**

When redirection is combined with a pipeline, the shell sets up the entire pipeline first, then it
redirects input/output. If output redirection is used in the middle of a pipeline, the output will go to
the file and not to the next command in the pipeline.

In this example, the output of the *ls* command goes to the file, and *less* displays nothing on the
terminal.

``` bash
[user@host ~]$ ls > /tmp/saved-output | less
```

The *tee* command overcomes this limitation. In a pipeline, *tee* copies its standard input to its
standard output and also redirects its standard output to the files named as arguments to the
command. If you imagine data as water flowing through a pipeline, *tee* can be visualized as a "T"
joint in the pipe which directs output in two directions.

![Process I/O piping with tee](https://www.thegeeksearch.com/images/post/_huf90ae59f9e7f9c3ca94da3a0ea3a6dd0_9839_a2160b7f0a2b09d22a52058172bcbb6d.webp "Process I/O piping with tee")

**Pipeline Examples Using the tee Command**

This example redirects the output of the ls command to the file and passes it to less to be
displayed on the terminal one screen at a time.

``` bash
[user@host ~]$ ls -l | tee /tmp/saved-output | less
```

If *tee* is used at the end of a pipeline, then the final output of a command can be saved and output
to the terminal at the same time.

``` bash
[user@host ~]$ ls -t | head -n 10 | tee /tmp/ten-last-changed-files
```

!!! danger "IMPORTANT"

	Standard error can be redirected through a pipe, but the merging redirection
	operators (**&>** and **&>>**) cannot be used to do this.
	
	The following is the correct way to redirect both standard output and standard error
	through a pipe:
	
	``` bash
	[user@host ~]$ find -name / passwd 2>&1 | less
	```

### EDITING TEXT FILES FROM THE SHELL PROMPT ###

#### EDITING FILES WITH VIM ####

A key design principle of Linux is that information and configuration settings are commonly stored
in text-based files. These files can be structured in various ways, as lists of settings, in INI-like
formats, as structured XML or YAML, and so on. However, the advantage of text files is that they
can be viewed and edited using any simple text editor.

Vim is an improved version of the **vi** editor distributed with Linux and UNIX systems. Vim is highly
configurable and efficient for practiced users, including such features as split screen editing, color
formatting, and highlighting for editing text.

**Why Learn Vim?**

You should know how to use at least one text editor that can be used from a text-only shell prompt.
If you do, you can edit text-based configuration files from a terminal window, or from remote logins
through *ssh* or the Web Console. Then you do not need access to a graphical desktop in order to
edit files on a server, and in fact that server might not need to run a graphical desktop environment
at all.

But then, why learn Vim instead of other possible options? The key reason is that Vim is almost
always installed on a server, if any text editor is present. This is because *vi* was specified by the
POSIX standard that Linux and many other UNIX-like operating systems comply with in large part.

In addition, Vim is often used as the *vi* implementation on other common operating systems or
distributions. For example, macOS currently includes a lightweight installation of Vim by default.
So Vim skills learned for Linux might also help you get things done elsewhere.

**Starting Vim**

Vim may be installed in Red Hat Enterprise Linux in two different ways. This can affect the features
and Vim commands available to you.

Your server might only have the vim-minimal package installed. This is a very lightweight
installation that includes only the core feature set and the basic *vi* command. In this case, you can
open a file for editing with *vi filename*, and all the core features discussed in this section will be
available to you.

Alternatively, your server might have the vim-enhanced package installed. This provides a much
more comprehensive set of features, an on-line help system, and a tutorial program. In order to
start Vim in this enhanced mode, you use the ***vim*** command.

``` bash
[user@host ~]$ vim filename
```

Either way, the core features that we will discuss in this section will work with both commands.

!!! info "NOTE"

	If vim-enhanced is installed, regular users will have a shell alias set so that if they run
	the **vi** command, they will automatically get the **vim** command instead. This does
	not apply to root and other users with UIDs below 200 (which are used by system
	services).

	If you are editing files as the root user and you expect vi to run in enhanced mode,
	this can be a surprise. Likewise, if vim-enhanced is installed and a regular user wants
	the simple vi for some reason, they might need to use **\vi** to override the alias
	temporarily.
	
	Advanced users can use **\vi --version** and **vim --version** to compare the
	feature sets of the two commands.

**Vim Operating Modes**

An unusual characteristic of Vim is that it has several modes of operation, including *command
mode, extended command mode, edit mode*, and *visual mode*. Depending on the mode, you may
be issuing commands, editing text, or working with blocks of text. As a new Vim user, you should
always be aware of your current mode as keystrokes have different effects in different modes.

![Moving between Vim modes](https://www.linuxprobe.com/links/RH124/images/edit/vim_modes_essential.png "Moving between Vim modes")

When you first open Vim, it starts in command mode, which is used for navigation, cut and paste,
and other text manipulation. Enter each of the other modes with single character keystrokes to
access specific editing functionality:

- An *i* keystroke enters insert mode, where all text typed becomes file content. Pressing *Esc*
returns to command mode.

- A *v* keystroke enters visual mode, where multiple characters may be selected for text
manipulation. Use *Shift+V* for multiline and *Ctrl+V* for block selection. The same keystroke
used to enter visual mode (*v, Shift+V or Ctrl+V*) is used to exit.

- The : keystroke begins extended command mode for tasks such as writing the file (to save it),
and quitting the Vim editor.

!!! info "NOTE"

	If you are not sure what mode Vim is in, you can try pressing **Esc** a few times to get
	back into command mode. Pressing **Esc** in command mode is harmless, so a few
	extra key presses are okay.

**The Minimum, Basic Vim Workflow**

Vim has efficient, coordinated keystrokes for advanced editing tasks. Although considered useful
with practice, Vim's capabilities can overwhelm new users.

The *i* key puts Vim into insert mode. All text entered after this is treated as file contents until you
exit insert mode. The *Esc* key exits insert mode and returns Vim to command mode. The u key will
undo the most recent edit. Press the *x* key to delete a single character. The *:w* command writes
(saves) the file and remains in command mode for more editing. The *:wq* command writes (saves)
the file and quits Vim. The *:q!* command quits Vim, discarding all file changes since the last write.
The Vim user must learn these commands to accomplish any editing task.

**Rearranging Existing Text**

In Vim, copy and paste is known as yank and put, using command characters *Y* and *P*. Begin by
positioning the cursor on the first character to be selected, and then enter visual mode. Use the
arrow keys to expand the visual selection. When ready, press *y* to yank the selection into memory.
Position the cursor at the new location, and then press *p* to put the selection at the cursor.

**Visual Mode in Vim**

Visual mode is a great way to highlight and manipulate text. There are three keystrokes:

- Character mode: v

- Line mode: Shift+V

- Block mode: Ctrl+V

Character mode highlights sentences in a block of text. The word *VISUAL* will appear at the bottom
of the screen. Press *v* to enter visual character mode. *Shift+V* enters line mode. *VISUAL LINE*
will appear at the bottom of the screen.

Visual block mode is perfect for manipulating data files. From the cursor, press the *Ctrl+V* to
enter visual block. *VISUAL BLOCK* will appear at the bottom of the screen. Use the arrow keys to
highlight the section to change.

!!! info "NOTE"

	Vim has a lot of capabilities, but you should master the basic workflow first. You do
	not need to quickly understand the entire editor and its capabilities. Get comfortable
	with those basics through practice and then you can expand your Vim vocabulary by
	learning additional Vim commands (keystrokes).
	
	The exercise for this section will introduce you to the **vimtutor** command. This
	tutorial, which ships with vim-enhanced, is an excellent way to learn the core
	functionality of Vim.

### CHANGING THE SHELL ENVIRONMENT ###

#### USING SHELL VARIABLES ####

The Bash shell allows you to set *shell variables* that you can use to help run commands or to modify
the behavior of the shell. You can also export shell variables as environment variables, which are
automatically copied to programs run from that shell when they start. You can use variables to help
make it easier to run a command with a long argument, or to apply a common setting to commands
run from that shell.

Shell variables are unique to a particular shell session. If you have two terminal windows open, or
two independent login sessions to the same remote server, you are running two shells. Each shell
has its own set of values for its shell variables.

**Assigning Values to Variables**

Assign a value to a shell variable using the following syntax:

`VARIABLENAME=value`

Variable names can contain uppercase or lowercase letters, digits, and the underscore character
(_). For example, the following commands set shell variables:

``` bash
[user@host ~]$ COUNT=40
[user@host ~]$ first_name=John
[user@host ~]$ file1=/tmp/abc
[user@host ~]$ _ID=RH123
```

Remember, this change only affects the shell in which you run the command, not any other shells
you may be running on that server.

You can use the *set* command to list all shell variables that are currently set. (It also lists all shell
functions, which you can ignore.) This list is long enough that you may want to pipe the output into
the *less* command so that you can view it one page at a time.

``` bash
[user@host ~]$ set | less
BASH=/usr/bin/bash
BASHOPTS=checkwinsize:cmdhist:complete_fullquote:expand_aliases:extglob:extquote:
force_fignore:histappend:interactive_comments:progcomp:promptvars:sourcepath
BASHRCSOURCED=Y
...output omitted...
```

**Retrieving Values with Variable Expansion**

You can use variable expansion to refer to the value of a variable that you have set. To do this,
precede the name of the variable with a dollar sign ($). In the following example, the *echo*
command prints out the rest of the command line entered, but after variable expansion is
performed.

For example, the following command sets the variable COUNT to 40.

``` bash
[user@host ~]$ COUNT=40
```

If you enter the command echo COUNT, it will print out the string COUNT.

``` bash
[user@host ~]$ echo COUNT
COUNT
```

But if you enter the command *echo $COUNT*, it will print out the value of the variable COUNT.

``` bash
[user@host ~]$ echo $COUNT
40
```

A more practical example might be to use a variable to refer to a long file name for multiple
commands.

``` bash
[user@host ~]$ file1=/tmp/tmp.z9pXW0HqcC
[user@host ~]$ ls -l $file1
-rw-------. 1 student student 1452 Jan 22 14:39 /tmp/tmp.z9pXW0HqcC
[user@host ~]$ rm $file1
[user@host ~]$ ls -l $file1
total 0
```

!!! danger "IMPORTANT"

	If there are any trailing characters adjacent to the variable name, you might need
	to protect the variable name with curly braces. You can always use curly braces
	in variable expansion, but you will also see many examples in which they are not
	needed and are omitted.
	
	In the following example, the first **echo** command tries to expand the nonexistent
	variable COUNTx, which does not cause an error but instead returns nothing.
	
	``` bash
	[user@host ~]$ echo Repeat $COUNTx
	Repeat
	[user@host ~]$ echo Repeat ${COUNT}x
	Repeat 40x
	```

**Configuring Bash with Shell Variables**

Some shell variables are set when Bash starts but can be modified to adjust the shell's behavior.
For example, two shell variables that affect the shell history and the *history* command are
HISTFILE and HISTFILESIZE. If HISTFILE is set, it specifies the location of a file to save
the shell history in when it exits. By default this is the user's *~/.bash_history* file. The
HISTFILESIZE variable specifies how many commands should be saved in that file from the
history.

Another example is PS1, which is a shell variable that controls the appearance of the shell prompt.
If you change this value, it will change the appearance of your shell prompt. A number of special
character expansions supported by the prompt are listed in the "PROMPTING" section of the
bash(1) man page.

``` bash
[user@host ~]$ PS1="bash\$ "
bash$ PS1="[\u@\h \W]\$ "
[user@host ~]$ 
```

Two items to note about the above example: first, because the value set by PS1 is a prompt, it is
virtually always desirable to end the prompt with a trailing space. Second, whenever the value of
a variable contains some form of space, including a space, a tab, or a return, the value must be
surrounded by quotes, either single or double; this is not optional. Unexpected results will occur
if the quotes are omitted. Examine the PS1 example above and note that it conforms to both the
recommendation (trailing space) and the rule (quotes).

### CONFIGURING PROGRAMS WITH ENVIRONMENT VARIABLES ###

The shell provides an *environment* to the programs you run from that shell. Among other things,
this environment includes information on the current working directory on the file system, the
command-line options passed to the program, and the values of *environment variables*. The
programs may use these environment variables to change their behavior or their default settings.

Shell variables that are not environment variables can only be used by the shell. Environment
variables can be used by the shell and by programs run from that shell.

!!! info "NOTE"

	HISTFILE, HISTFILESIZE, and PS1, learned in the previous section, do not need
	to be exported as environment variables because they are only used by the shell
	itself, not by the programs that you run from the shell.

You can make any variable defined in the shell into an environment variable by marking it for
export with the **export** command.

``` bash
[user@host ~]$ EDITOR=vim
[user@host ~]$ export EDITOR
```

You can set and export a variable in one step:

``` bash
[user@host ~]$ export EDITOR=vim
```

Applications and sessions use these variables to determine their behavior. For example, the shell
automatically sets the HOME variable to the file name of the user's home directory when it starts.
This can be used to help programs determine where to save files.

Another example is LANG, which sets the locale. This adjusts the preferred language for program
output; the character set; the formatting of dates, numbers, and currency; and the sort order
for programs. If it is set to **en_US.UTF-8**, the locale will use US English with UTF-8 Unicode
character encoding. If it is set to something else, for example **fr_FR.UTF-8**, it will use French
UTF-8 Unicode encoding.

``` bash
[user@host ~]$ date
Tue Jan 22 16:37:45 CST 2019
[user@host ~]$ export LANG=fr_FR.UTF-8
[user@host ~]$ date
mar. janv. 22 16:38:14 CST 2019
```

Another important environment variable is PATH. The PATH variable contains a list of colon-
separated directories that contain programs:

``` bash
[user@host ~]$ echo $PATH
/home/user/.local/bin:/home/user/bin:/usr/share/Modules/bin:/usr/local/bin:/usr/
bin:/usr/local/sbin:/usr/sbin
```

When you run a command such as **ls**, the shell looks for the executable file ls in each of those
directories in order, and runs the first matching file it finds. (On a typical system, this is /usr/
bin/ls.)

You can easily add additional directories to the end of your PATH. For example, perhaps you have
executable programs or scripts that you want to run like regular commands in **/home/user/sbin**.
You can add **/home/user/sbin** to the end of your PATH for the current session like this:

``` bash
[user@host ~]$ export PATH=${PATH}:/home/user/sbin
```

To list all the environment variables for a particular shell, run the **env** command:

``` bash
[user@host ~]$ env
...output omitted...
LANG=en_US.UTF-8
HISTCONTROL=ignoredups
HOSTNAME=host.example.com
XDG_SESSION_ID=4
...output omitted...
```

**Setting the Default Text Editor**

The EDITOR environment variable specifies the program you want to use as your default text
editor for command-line programs. Many programs use **vi** or **vim** if it is not specified, but you can
override this preference if required:

``` bash
[user@host ~]$ export EDITOR=nano
```

!!! danger "IMPORTANT"

	By convention, environment variables and shell variables that are automatically set
	by the shell have names that use all uppercase characters. If you are setting your
	own variables, you may want to use names made up of lowercase characters to help
	avoid naming collisions.

### SETTING VARIABLES AUTOMATICALLY ###

If you want to set shell or environment variables automatically when your shell starts, you can edit
the Bash startup scripts. When Bash starts, several text files containing shell commands are run
which initialize the shell environment.

The exact scripts that run depend on how the shell was started, whether it is an interactive login
shell, an interactive non-login shell, or a shell script.

Assuming the default **/etc/profile**, **/etc/bashrc**, and **~/.bash_profile** files, if you want
to make a change to your user account that affects all your interactive shell prompts at startup,
edit your **~/.bashrc** file. For example, you could set that account's default editor to **nano** by
editing the file to read:

```
# .bashrc
# Source global definitions
if [ -f /etc/bashrc ]; then
 . /etc/bashrc
fi
# User specific environment
PATH="$HOME/.local/bin:$HOME/bin:$PATH"
export PATH
# User specific aliases and functions
export EDITOR=nano
```

!!! info "NOTE"

	The best way to adjust settings that affect all user accounts is by adding a file with a
	name ending in **.sh** containing the changes to the **/etc/profile.d** directory. To
	do this, you need to be logged in as the root user.

#### UNSETTING AND UNEXPORTING VARIABLES ####

To unset and unexport a variable entirely, use the **unset** command:

``` bash
[user@host ~]$ echo $file1
/tmp/tmp.z9pXW0HqcC
[user@host ~]$ unset file1
[user@host ~]$ echo $file1
[user@host ~]$ 
```

To unexport a variable without unsetting it, use the export -n command:

``` bash
[user@host ~]$ export -n PS1
```

### SUMMARY ###

In this chapter, you learned:

- Running programs, or processes, have three standard communication channels, standard input,
standard output, and standard error.

- You can use I/O redirection to read standard input from a file or write the output or errors from a
process to a file.

- Pipelines can be used to connect standard output from one process to standard input of another
process, and can be used to format output or build complex commands.

- You should know how to use at least one command-line text editor, and Vim is generally
installed.

- Shell variables can help you run commands and are unique to a particular shell session.

- Environment variables can help you configure the behavior of the shell or the processes it starts.
