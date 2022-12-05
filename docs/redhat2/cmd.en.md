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
### USING EXIT CODES WITHIN A SCRIPT

After a script has processed all of its contents, it exits to the process that called it. However, there
may be times when it is desirable to exit a script before it finishes, such as when an error condition
is encountered. This can be accomplished with the use of the **exit** command within a script. When
a script encounters the **exit** command, it exits immediately and does not process the remainder
of the script.

The **exit** command can be executed with an optional integer argument between **0** and **255**, which
represents an exit code. An exit code is a code that is returned after a process has completed. An
exit code value of **0** represents no error. All other nonzero values indicate an error exit code. You
can use different nonzero values to differentiate between different types of errors encountered.
This exit code is passed back to the parent process, which stores it in the ? variable and can be
accessed with **$?** as demonstrated in the following examples.

```bash
[user@host bin]$ cat hello
#!/bin/bash
echo "Hello, world"
exit 0
[user@host bin]$ ./hello
Hello, world
[user@host bin]$ echo $?
0

```
If the **exit** command is called without an argument, then the script exits and passes the exit
status of the last command executed to the parent process.

#### TESTING SCRIPT INPUTS

To ensure that scripts are not easily disrupted by unexpected conditions, it is good practice to
not make assumptions regarding input, such as command-line arguments, user input, command
substitutions, variable expansions, and file-name expansions. Integrity checking can be performed
by using Bash's test command.

Like all commands, the test command produces an exit code upon completion, which is stored
as the value $?. To see the conclusion of a test, display the value of $? immediately following the
execution of the test command. Again, an exit status value of 0 indicates the test succeeded, and
nonzero values indicate the test failed.

Tests are performed using a variety of operators. Operators can be used to determine if a number
is greater than, greater than or equal to, less than, less than or equal to, or equal to another
number. They can be used to test if a string of text is the same or not the same as another string of
text. Operators can also be used to evaluate if a variable has a value or not.

!!! info "NOTE"

    Many types of operators are used in shell scripting, in addition to the comparison
    operators taught here. The man page for **test**(1) lists the important conditional
    expression operators with descriptions. The **bash**(1) man page also explains
    operator use and evaluation, but is a very difficult read for beginners. It is
    recommended that students further their advanced shell scripting needs through
    books and courses dedicated to shell programming.


The following examples demonstrate the use of the **test** command using Bash's numeric comparison operators.

```bash
[user@host ~]$ test 1 -gt 0 ; echo $?
0
[user@host ~]$ test 0 -gt 1 ; echo $?
1

```
Tests can be performed using the Bash test command syntax, [ **<TESTEXPRESSION>** ].
They can also be performed using Bash's newer extended test command syntax,
[[ **<TESTEXPRESSION>** ]], which has been available since Bash version 2.02 and provides
features such as glob pattern matching and regex pattern matching.

The following examples demonstrate the use of Bash's **test** command syntax and Bash's numeric
comparison operators.

```bash
[user@host ~]$ [ 1 -eq 1 ]; echo $?
0
[user@host ~]$ [ 1 -ne 1 ]; echo $?
1
[user@host ~]$ [ 8 -gt 2 ]; echo $?
0
[user@host ~]$ [ 2 -ge 2 ]; echo $?
0
[user@host ~]$ [ 2 -lt 2 ]; echo $?
1
[user@host ~]$ [ 1 -lt 2 ]; echo $?
0
```
The following examples demonstrate the use of Bash's string comparison operators.

```bash
[user@host ~]$ [ abc = abc ]; echo $?
0
[user@host ~]$ [ abc == def ]; echo $?
1
[user@host ~]$ [ abc != def ]; echo $?
0

```
The following examples demonstrate the use of Bash's string unary operators.

```bash
[user@host ~]$ STRING=''; [ -z "$STRING" ]; echo $?
0
[user@host ~]$ STRING='abc'; [ -n "$STRING" ]; echo $?
0

```
!!! info "NOTE"

    The space characters inside the test brackets are mandatory, because they
    separate the words and elements within the test expression. The shell's command
    parsing routine divides all command lines into words and operators by recognizing
    spaces and other metacharacters, using built-in parsing rules. For full treatment
    of this advanced concept, see the man page getopt(3). The left square bracket
    character ([) is itself a built-in alias for the test command. Shell words, whether
    they are commands, subcommands, options, arguments or other token elements,
    are always delimited by spaces.


### CONDITIONAL STRUCTURES

Simple shell scripts represent a collection of commands that are executed from beginning to end.
Conditional structures allow users to incorporate decision making into shell scripts, so that certain
portions of the script are executed only when certain conditions are met.

**Using the if/then Construct**

The simplest of the conditional structures in Bash is the if/then construct, which has the following
syntax.

```
if <CONDITION>; then
<STATEMENT>
...
<STATEMENT>
fi

```
With this construct, if a given condition is met, one or more actions are taken. If the given
condition is not met, then no action is taken. The numeric, string, and file tests previously
demonstrated are frequently utilized for testing the conditions in **if/then** statements. The **fi**
statement at the end closes the **if/then** construct. The following code section demonstrates the
use of an **if/then** construct to start the psacct service if it is not active.

```bash
[user@host ~]$ systemctl is-active psacct > /dev/null 2>&1
[user@host ~]$ if [ $? -ne 0 ]; then
> sudo systemctl start psacct
> fi

```
**Using the if/then/else Construct**

The **if/then** construct can be further expanded so that different sets of actions can be taken
depending on whether a condition is met. This is accomplished with the **if/then/else** construct.

```
if <CONDITION>; then
<STATEMENT>
...
<STATEMENT>
else
<STATEMENT>
...
<STATEMENT>
fi

```
The following code section demonstrates the use of an **if/then/else** statement to start the
psacct service if it is not active and to stop it if it is active.

```
[user@host ~]$ systemctl is-active psacct > /dev/null 2>&1
[user@host ~]$ if [ $? -ne 0 ]; then
> sudo systemctl start psacct
> else
> sudo systemctl stop psacct
> fi

```
**Using the if/then/elif/then/else Construct**

Lastly, the **if/then/else** construct can be further expanded to test more than one condition,
executing a different set of actions when a condition is met. The construct for this is shown in the
following example:

```
if <CONDITION>; then
<STATEMENT>
...
<STATEMENT>
elif <CONDITION>; then
<STATEMENT>
...
<STATEMENT>
else
<STATEMENT>
...
<STATEMENT>
fi
```
In this conditional structure, Bash tests the conditions in the order presented. When it finds a
condition that is true, Bash executes the actions associated with the condition and then skips
the remainder of the conditional structure. If none of the conditions are true, Bash executes the
actions enumerated in the **else** clause.

The following code section demonstrates the use of an **if/then/elif/then/else** statement
to run the **mysql** client if the mariadb service is active, run the **psql** client if the postgresql
service is active, or run the **sqlite3** client if both the mariadb and postgresql services are not
active.

```bash
[user@host ~]$ systemctl is-active mariadb > /dev/null 2>&1
MARIADB_ACTIVE=$?
[user@host ~]$ sudo systemctl is-active postgresql > /dev/null 2>&1
POSTGRESQL_ACTIVE=$?
[user@host ~]$ if [ "$MARIADB_ACTIVE" -eq 0 ]; then
> mysql
> elif [ "$POSTGRESQL_ACTIVE" -eq 0 ]; then
> psql
> else
> sqlite3
> fi
```
### MATCHING TEXT IN COMMAND OUTPUT WITH REGULAR EXPRESSIONS

#### WRITING REGULAR EXPRESSIONS

Regular expressions provide a pattern matching mechanism that facilitates finding specific
content. The **vim , grep, and less** commands can all use regular expressions. Programming
languages such as Perl, Python, and C can all use regular expressions when using pattern matching
criteria.

Regular expressions are a language of their own, which means they have their own syntax and
rules. This section looks at the syntax used when creating regular expressions, as well as showing
some regular expression examples.

**Describing a Simple Regular Expression**

The simplest regular expression is an exact match. An exact match is when the characters in the
regular expression match the type and order in the data that is being searched.

Suppose a user is looking through the following file looking for all occurrences of the pattern **cat**:

```
cat
dog
concatenate
dogma
category
educated
boondoggle
vindication
chilidog

```
**cat** is an exact match of a **c**, followed by an **a**, followed by a **t** with no other characters in between.
Using **cat** as the regular expression to search the previous file returns the following matches:

```
cat
concatenate
category
educated
vindication
```
**Matching the Start and End of a Line**

The previous section used an exact match regular expression on a file. Note that the regular
expression would match the search string no matter where on the line it occurred: the beginning,
end, or middle of the word or line. Use a line anchor to control the location of where the regular
expression looks for a match.

To search at the beginning of a line, use the caret character (^). To search at the end of a line, use
the dollar sign ($).

Using the same file as above, the **^cat** regular expression would match two words. The **$cat**
regular expression would not find any matching words.

```
cat
dog
concatenate
dogma
category
educated
boondoggle
vindication
chilidog

```
To locate lines in the file ending with **dog**, use that exact expression and an end of line anchor to
create the regular expression dog$. Applying dog$ to the file would find two matches:

```
dog
chilidog

```
To locate the only word on a line, use both the beginning and end-of-line anchors. For example, to
locate the word **cat** when it is the only word on a line, use **^cat$**.

```
cat dog rabbit
cat
horse cat cow
cat pig

```
**Adding Wildcards and Multipliers to Regular Expressions**

Regular expressions use a period or dot (.) to match any single character with the exception
of the newline character. A regular expression of c.t searches for a string that contains a c
followed by any single character followed by a t. Example matches include **cat, concatenate,
vindication, c5t, and c$t**.

Using an unrestricted wildcard you cannot predict the character that would match the wildcard. To
match specific characters, replace the unrestricted wildcard with acceptable characters. Changing
the regular expression to **c[aou]t** matches patterns that start with a c, followed by either an a, o,
or u, followed by a t.

Multipliers are a mechanism used often with wildcards. Multipliers apply to the previous character
in the regular expression. One of the more common multipliers used is the asterisk, or star
character (*). When used in a regular expression, this multiplier means match zero or more of the
previous expression. You can use * with expressions, not just characters. For example, c[aou]*t. A
regular expression of c.*t matches **cat, coat, culvert**, and even ct (zero characters between
the c and the t). Any data starting with a c, then zero or more characters, ending with a t.

Another type of multiplier would indicate the number of previous characters desired in the pattern.
An example of using an explicit multiplier would be **'c.\{2\}t'**. This regular expression will
match any word beginning with a c, followed by exactly any two characters, and ending with a t.
**'c.\{2\}t'** would match two words in the example below:

```
cat
coat convert
cart covert
cypher

```
!!! info "NOTE"
   
    It is recommend practice to use single quotes to encapsulate the regular expression
    because they often contain shell metacharacters (such as **$, ***, and **{}**). This
    ensures that the characters are interpreted by the command and not by the shell.

!!! info "NOTE"

    This course has introduced two distinct metacharacter text parsing systems: shell
    pattern matching (also known as file globbing or file-name expansion), and regular
    expressions. Because both systems use similar metacharacters, such as the asterisk
    (*), but have differences in metacharacter interpretation and rules, the two systems
    can be confusing until each is sufficiently mastered.

    Pattern matching is a command-line parsing technique designed for specifying
    many file names easily, and is primarily supported only for representing file-name
    patterns on the command line. Regular expressions are designed to represent
    any form or pattern in text strings, no matter how complex. Regular expression
    are internally supported by numerous text processing commands, such as **grep,
    sed, awk, python, perl**, and many applications, with some minimal command-
    dependent variations in interpretation rules.

**Regular Expressions**


OPTION | DESCRIPTION |
---------|----------|
 . | The period (.) matches any single character. |
 ? | The preceding item is optional and will be matched at most once. |
 * | The preceding item will be matched zero or more times. |
 + | The preceding item will be matched one or more times. |
{n}   | The preceding item is matched exactly n times. |
{n,}   | The preceding item is matched n or more times. |
{,m}   | The preceding item is matched at most m times. |
 {n,m}  | The preceding item is matched at least n times, but not more than m times. |
 [:alnum:]  | Alphanumeric characters: '[:alpha:]' and '[:digit:]'; in the 'C' locale and ASCII character encoding this is the same as '[0-9A-Za-z]'. |
 [:alpha:]  | Alphabetic characters: '[:lower:]' and '[:upper:]'; in the 'C' locale and ASCII character encoding, this is the same as '[A-Za-z]'. |
  [:blank:] | Blank characters: space and tab. |
 [:cntrl:]  | Control characters. In ASCII, these characters have octal codes 000 through 037, and 177 (DEL). In other character sets, these are the equivalent characters, if any. |
 [:digit:]  | Digits: 0 1 2 3 4 5 6 7 8 9. |
 [:graph:]  | Graphical characters: '[:alnum:]' and '[:punct:]'. |
 [:lower:] | Lower-case letters; in the 'C' locale and ASCII character encoding, this is a b c d e f g h i j k l m n o p q r s t u v w x y z. |
 [:print:] | Printable characters: '[:alnum:]', '[:punct:]', and space. |
 [:punct:] | Punctuation characters; in the 'C' locale and ASCII character encoding, this is! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ' { | } ~. In other character sets, these are the equivalent characters, if any. |
 [:space:] | Space characters: in the 'C' locale, this is tab, newline, vertical tab, form feed,carriage return, and space. |
 [:upper:] | Upper-case letters: in the 'C' locale and ASCII character encoding, this is A B C D E F G H I J K L M N O P Q R S T U V W X Y Z. |
 [:xdigit:] | Hexadecimal digits: 0 1 2 3 4 5 6 7 8 9 A B C D E F a b c d e f. |
 \b | Match the empty string at the edge of a word. |
 \B | Match the empty string provided it is not at the edge of a word. |
 \< | Match the empty string at the beginning of word. |
 "\>" | Match the empty string at the end of word. |
 \w | Match word constituent. Synonym for '[_[:alnum:]]'. |
 \W | Match non-word constituent. Synonym for '[^_[:alnum:]]'. |
 \s | Match white space. Synonym for '[[:space:]]'. |
 \S | Match non-whitespace. Synonym for '[^[:space:]]'. |

### MATCHING REGULAR EXPRESSIONS WITH GREP

The **grep** command, provided as part of the distribution, uses regular expressions to isolate
matching data.

**Isolating data using the grep command**

The grep command provides a regular expression and a file on which the regular expression
should be matched.

```bash
[user@host ~]$ grep '^computer' /usr/share/dict/words
computer
computerese
computerise
computerite
computerizable
computerization
computerize
computerized
computerizes
computerizing
computerlike
computernik
computers

```
!!! info "NOTE"

        It is recommend practice to use single quotes to encapsulate the regular expression
        because they often contain shell metacharacters (such as $, *, and {}). This ensures
        that the characters are interpreted by _grep_ and not by the shell.

The **grep** command can be used in conjunction with other commands using a pipe operator (|).
For example:

```bash
[root@host ~]# ps aux | grep chrony
chrony  662     0.0     0.1     29440   2468 ?    S     10:56   0:00 /usr/sbin/chronyd

```
**grep Options**

The **grep** command has many useful options for adjusting how it uses the provided regular
expression with data.

**Table of Common grep Options**

OPTION | FUNCTION |
---------|----------|
 -i | Use the regular expression provided but do not enforce case sensitivity (run case-insensitive). |
 -v | Only display lines that do not contain matches to the regular expression. |
 -r  | Apply the search for data matching the regular expression recursively to a group of files or directories. |
  -A NUMBER  | Display NUMBER of lines after the regular expression match.  |
  -B NUMBER  |  Display NUMBER of lines before the regular expression match. |
  -e  | With multiple -e options used, multiple regular expressions can be supplied and will be used with a logical OR.  |

There are many other options to **grep**. Use the **man** page to research them.

**grep Examples**

The next examples use varied configuration files and log files.

Regular expressions are case-sensitive by default. Use the **-i** option with **grep** to run a case-
insensitive search. The following example searches for the pattern **serverroot**.

```bash
[user@host ~]$ cat /etc/httpd/conf/httpd.conf
...output omitted...
ServerRoot "/etc/httpd"

```
```
#
# Listen: Allows you to bind Apache to specific IP addresses and/or
# ports, instead of the default. See also the <VirtualHost>
# directive.
#
# Change this to Listen on specific IP addresses as shown below to
# prevent Apache from glomming onto all bound IP addresses.
#
#Listen 12.34.56.78:80
Listen 80

```

```bash
[user@host ~]$ grep -i serverroot /etc/httpd/conf/httpd.conf
# with "/", the value of ServerRoot is prepended -- so 'log/access_log'
# with ServerRoot set to '/www' will be interpreted by the
# ServerRoot: The top of the directory tree under which the server's
# ServerRoot at a non-local disk, be sure to specify a local disk on the
# same ServerRoot for multiple httpd daemons, you will need to change at
ServerRoot "/etc/httpd"
```
In cases where you know what you are not looking for, the **-v** option is very useful. The **-v** option
only displays lines that do not match the regular expression. In the following example, all lines,
regardless of case, that do not contain the regular expression **server** are returned.

```bash
[user@host ~]$ cat /etc/hosts
127.0.0.1
 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1
 localhost localhost.localdomain localhost6 localhost6.localdomain6
172.25.254.254 classroom.example.com classroom
172.25.254.254 content.example.com content
172.25.254.254 materials.example.com materials
172.25.250.254 workstation.lab.example.com workstation
### rht-vm-hosts file listing the entries to be appended to /etc/hosts
172.25.250.10
172.25.250.11
172.25.250.254
servera.lab.example.com servera
serverb.lab.example.com serverb
workstation.lab.example.com workstation
[user@host ~]$ grep -v -i server /etc/hosts
127.0.0.1
 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1
 localhost localhost.localdomain localhost6 localhost6.localdomain6
172.25.254.254 classroom.example.com classroom
172.25.254.254 content.example.com content
172.25.254.254 materials.example.com materials
172.25.250.254 workstation.lab.example.com workstation
### rht-vm-hosts file listing the entries to be appended to /etc/hosts
172.25.250.254
 workstation.lab.example.com workstation

```
To look at a file without being distracted by comment lines use the **-v** option. In the following
example, the regular expression matches all lines that begin with a # or ; (typical characters that
indicate the line will be interpreted as a comment). Those lines are then omitted from the output.

```bash
[user@host ~]$ cat /etc/ethertypes
#
# Ethernet frame types
#
 This file describes some of the various Ethernet
#
 protocol types that are used on Ethernet networks.
#
# This list could be found on:
#
 http://www.iana.org/assignments/ethernet-numbers
#
 http://www.iana.org/assignments/ieee-802-numbers
#
# <name>
 <hexnumber> <alias1>...<alias35> #Comment
#
IPv4     0800    ip ip4      # Internet IP (IPv4)
X25      0805
ARP      0806    ether-arp  #
FR_ARP      0808            # Frame Relay ARP    [RFC1701]

[user@host ~]$ grep -v '^[#;]' /etc/ethertypes
IPv4     0800    ip ip4      # Internet IP (IPv4)
X25      0805
ARP      0806    ether-arp   #
FR_ARP   0808    # Frame Relay ARP   [RFC1701]
```

The **grep** command with the **-e** option allows you to search for more than one regular expression
at a time. The following example, using a combination of **less** and **grep**, locates all occurrences of
**pam_unix, user root** and **Accepted publickey** in the **/var/log/secure** log file.

```bash
[root@host ~]# cat /var/log/secure | grep -e 'pam_unix' \
-e 'user root' -e 'Accepted publickey' | less
Mar 19 08:04:46 jegui sshd[6141]: pam_unix(sshd:session): session opened for user
root by (uid=0)
Mar 19 08:04:50 jegui sshd[6144]: Disconnected from user root 172.25.250.254 port
41170
Mar 19 08:04:50 jegui sshd[6141]: pam_unix(sshd:session): session closed for user
root
Mar 19 08:04:53 jegui sshd[6168]: Accepted publickey for student from
172.25.250.254 port 41172 ssh2: RSA SHA256:M8ikhcEDm2tQ95Z0o7ZvufqEixCFCt
+wowZLNzNlBT0

```
To search for text in a file opened using **vim** or **less**, use the slash character (/) and type the
pattern to find. Press **Enter** to start the search. Press **N** to find the next match.

```bash
[root@host ~]# vim /var/log/boot.log
...output omitted...
[^[[0;32m OK ^[[0m] Reached target Initrd Default Target.^M
Starting dracut pre-pivot and cleanup hook...^M
[^[[0;32m OK ^[[0m] Started dracut pre-pivot and cleanup hook.^M
Starting Cleaning Up and Shutting Down Daemons...^M
Starting Plymouth switch root service...^M
Starting Setup Virtual Console...^M
[^[[0;32m OK ^[[0m] Stopped target Timers.^M
[^[[0;32m OK ^[[0m] Stopped dracut pre-pivot and cleanup hook.^M
[^[[0;32m OK ^[[0m] Stopped target Initrd Default Target.^M
[root@host ~]# less /var/log/messages
...output omitted...
Feb 26 15:51:07 jegui NetworkManager[689]: <info>
 [1551214267.8584] Loaded device
plugin: NMTeamFactory (/usr/lib64/NetworkManager/1.14.0-14.el8/libnm-device-
plugin-team.so)
Feb 26 15:51:07 jegui NetworkManager[689]: <info>
 [1551214267.8599] device (lo):
carrier: link connected
Feb 26 15:51:07 jegui NetworkManager[689]: <info>
 [1551214267.8600] manager:
(lo): new Generic device (/org/freedesktop/NetworkManager/Devices/1)
Feb 26 15:51:07 jegui NetworkManager[689]: <info>
 [1551214267.8623] manager:
(ens3): new Ethernet device (/org/freedesktop/NetworkManager/Devices/2)
Feb 26 15:51:07 jegui NetworkManager[689]: <info>
 [1551214267.8653] device
(ens3): state change: unmanaged -> unavailable (reason 'managed', sys-iface-
state: 'external')
/device
```

### SUMMARY

In this chapter, you learned:

- How to create and execute simple Bash scripts.

- How to use loops to iterate through a list of items from the command-line and in a shell script.

- How to search for text in log files and configuration files using regular expressions and grep.

### LAB

pdf - 31