---
title: "Tritree"
---

### RHCSA Notes

### **Which**

which command in Linux is a command which is used to locate the executable file associated with the given command by searching it in the path environment variable.
It has 3 return status as follows:

* 0 : If all specified commands are found and executable.
* 1 : If one or more specified commands is nonexistent or not executable.
* 2 : If an invalid option is specified.

<mark>echo $?</mark> exit code to show errors for last command
if error output 1
if no error output 0


### **Locate**

The locate command finds files in Linux using the file name. locate is used for obtaining instantaneous results, and it is an essential utility when speed is a priority.

install mlocate package.

`locate mysql | less`

Print the number of matched files

`locate -c mysql`

Limit the number of search results

`locate  git -n 10`

To ignore case sensitivity using the -i option

`locate -i git`

Search for a File with an Exact Name

`locate -r mysql$`

---

### **Tee**

It is like T junction or pipe

tee is used to save to a file compulsory

Save output to out.txt and send to input of wc simultaneously

ls -l | tee out.txt | wc

To display output to terminal 

ls -l | tee out.txt

Pass output to wc command

ls -l | tee out.txt | wc > wc.txt

ls -l | tee out.txt | wc | tee wc.txt

---

### **Xargs**

To Convert output stream into command line arguments

Read filenames from test.txt and remove every file

cat test.txt | xargs rm

Read filenames from test.txt and display file contents to terminal

cat test.txt | xargs cat

---

### **Find**

The find command is used to search and locate the list of files and directories based on conditions you specify for files that match the arguments.

[Link](https://www.tecmint.com/35-practical-examples-of-linux-find-command/)

---

### **Tar**

The Linux “tar” stands for tape archive, which is used by a large number of Linux/Unix system administrators to deal with tape drives backup.

[Link](https://www.tecmint.com/18-tar-command-examples-in-linux/)

---

### **xz**

xz is a new general-purpose, command line data compression utility, similar to gzip and bzip2. It can be used to compress or decompress a file according to the selected operation mode.
It supports various formats to compress or decompress files.

[link](https://www.tecmint.com/xz-command-examples-in-linux/)

---

### **Links**

Links are a very handy way to create a shortcut to an original directory.
In Linux there are two different types of links:

1. Hard links

2. Symbolic links

[link](https://www.geeksforgeeks.org/soft-hard-links-unixlinux/)

---

### **Inode**

Linux must allocate an index node (inode) for every file and directory in the filesystem. Inodes do not store actual data. Instead, they store the metadata where you can find the storage blocks of each file’s data.

[link](https://docs.rackspace.com/support/how-to/what-are-inodes-in-linux/)

---

### **Cut**

The cut command in UNIX is a command for cutting out the sections from each line of files and writing the result to standard output. It can be used to cut parts of a line by byte position, character and field.

Display specific character: 

`cut -c 8 test.txt`

Display range of characters:

`cut -c 4-8 test.txt`

Display 5 character to last character:

`cut -c 5- test.txt`

Display 1 to specified character:

`cut -c -3 test.txt`

Display 3 to 5 and 7 to 10 characters:

`cut -c 3-5,7-10 test.txt`

Display specific column data:

 -d -- delimiter ignore pipe symbol, by default TAB.
 
 -f -- field
 
we get 3 column data.

`cut -d "|" -f 3 test.txt`

Display range of columns like 2 to 4:

`cut -d "|" -f 2-4 test.txt`

Display 2 to last column:

`cut -d "|" -f 2- text.txt`

Display 1 to specified column:

`cut -d "|" -f -3 text.txt`

Display 1,3,5 columns:

`cut -d "|" -f 1,3,5 text.txt`

Display all columns except 3 and 5:

 use --complement option

`cut -d "|" --complement -f 3,5 text.txt`

 3 to 5 columns:

`cut -d "|" --complement -f 3-5 text.txt`

---

### **Tr**

 Stands for translate or delete.
 
Search and replace a to A character:

`echo "this is a line" | tr 'a' 'A'`

`echo "this is a line" | tr 'aeiou' 'AEIOU'`

Delete specified characters:

`echo "this is a line" | tr -d 'aeiou'`

Remove repeated characters in word:

`echo "thiiis iis aaa linee" | tr -s 'aeiou'`


---

### **Regular Expressions**

By using wild card characters, we can build regular expressions.

 "*" --> represents zero or more characters, i.e any number of characters.
 
 "?" --> represents only one character
 
 "[]" --> range of characters
 
 "[abc]" --> a or b or c (we have to take only one character a or b or c)
 
 "[!abc]" --> except a,b,c. any character
 
 "[a-z]" --> any lower case alphabet symbol
 
 "[A-Z]" --> any upper case alphabet symbol
 
 "[a-zA-z]" --> any alphabet symbol either lower or upper
 
 "[0-9]" --> any digit 0 to 9
 
 "[a-zA-z0-9]" --> any alphanumeric character
 
 "[!a-zA-z0-9]" --> Except any alphanumeric character (special symbol)
 
 "[[:lower:]]" --> Any lower case alphabet symbol
 
 "[[:upper:]]" --> Any upper case alphabet symbol
 
 "[[:alpha:]]" --> Any alphabet symbol
 
 "[[:digit:]]" --> Any digit from 0 to 9
 
 "[[:alnum:]]" --> This is the same as ‘[0-9A-Za-z]’
 
 "[![:digit:]]" --> Any character except digit
 
 "{ }" --> List of files with comma seperator.
 
 "{a..z}" --> a to z files
 
 `touch {a..d}.java` --> create a,b,c,d files
 
* To list out all files where file name starts with a or b or c

`ls [a-c]*` --> a.txt, b.txt, c.java

* List first one uppercase alpha, second digit, third lowercase alpha

`ls [[:upper:]][[:digit:]][[:lower:]]` --> A7b W4u Z6k

* List all files starts with special symbol

`ls [![:alnum:]]*` --> %abc.txt

* List all files .java or .py extension

`ls {*.java,*.py}` --> b.java, c.java, j.py, k.py

* Remove all files start with a or b or c and ends with e or t

`rm [abc]*[et]` --> a.txt and b.txt will be removed

* List files begin with upper and has letter d in 3rd char, ends with lowercase

`[[:upper:]]?d*[[:lower]]`



### **Password authentication**

Password authentication for aws instance not keybased
vi /etc/ssh/sshd_config

PasswordAuthentication yes

```mysql
systemctl restart sshd
```

visudo for sudo access in /etc/sudoers file without password

username All=(ALL) NOPASSWD: ALL

---

To check memory:
```mysql
free -h
```
---

### **/etc/hosts**

All operating systems with network support have a hosts file to translate hostnames to IP addresses.
Whenever you open a website by typing its hostname, your system will read through the hosts file to check for the corresponding IP and then open it.
The hosts file is a simple text file located in the etc folder on Linux and Mac OS (/etc/hosts).

Whenever you type an address, your system will check the hosts file for its presence;
if it is present there, you will be directed to the corresponding IP. If the hostname is not defined in the hosts file,
your system will check the DNS server of your internet to look up for the corresponding IP and redirect you accordingly.

Why Edit /etc/hosts file?

By editing the hosts files, you can achieve the following things:

* Block a website:

You can block a website by redirecting it to the IP of your localhost or the default route.

For example, if we want to block google.com, we can add the following text to our file:

`127.0.0.1 www.google.com`

or

`0.0.0.0 www.google.com`

Now when you try to open www.google.com from your browser, you will see an error.

* Access Remote Computer Through an Alias:

Suppose we have a server located at a local network that we want to access. We usually have to type the server’s IP to access it unless it has been defined on our local DNS.
One way to avoid typing the IP, again and again, is to assign an alias to the server in the hosts file as follows:

`192.168.1.10 myserver`

The IP corresponds to the location of the server we want to access and myserver is the new alias we want to use.

Sometimes you may need to restart the web browser to apply changes.

---

### **Findmnt**

The more common command to check mounted file systems on linux is the mount command which is used to not only list mounted devices, but also mount and unmount them as and when needed.

The findmnt command shows details like the device name, mount directory, mount options and the file system type.

* List the file systems:

```mysql
findmnt
```

* Output in list format:

```mysql
findmnt -l
```

* df style output:

```mysql
findmnt -D
```

* Read file systems from fstab:

```mysql
findmnt -s
```

* Filter filesystems by type:

```mysql
findmnt -t ext4
```

* Search by source device:

```bash
findmnt -S /dev/sda1
```

* Search by mount point:

```bash
findmnt -T /
```

---

To run previous command:

```bash
!f
```

---

To list environment variables:

```bash
env
```

---

### **Grep**

The grep filter searches a file for a particular pattern of characters, and displays all lines that contain that pattern.
The pattern that is searched in the file is referred to as the regular expression (grep stands for global search for regular expression and print out).

* Search any line that contains the word in filename on Linux:

`grep 'word' filename`

* Perform a case-insensitive search for the word ‘bar’ in Linux and Unix:

`grep -i 'bar' file1`

* Look for all files in the current directory and in all of its subdirectories in Linux for the word ‘httpd’:

`grep -R 'httpd' .`

* Search and display the total number of times that the string ‘nixcraft’ appears in a file named frontpage.md:

`grep -c 'nixcraft' frontpage.md`

* Use grep to search words only:

You can force the grep command to select only those lines containing matches that form whole words

`grep -w "boo" file`

* Output with the number of the line in the text file:

`grep -n 'root' /etc/passwd`

* Force grep invert match:

For example print all line that do not contain the word start root

`grep -v '^root' /etc/passwd`

* Display lines before and after the match:

Want to see the lines before your matches

`grep -B 3 'root' /etc/passwd`

display the lines after your matches

`grep -A 3 'root' /etc/passwd`

prints the searched line and 3 lines after and before the result

`grep -C 3 'root' /etc/passwd`

* Display the file names that matches the pattern:

`grep -l "unix" *`

* Displaying only the matched pattern:

By default, grep displays the entire line which has the matched string. We can make the grep to display only the matched string by using the -o option.

`grep -o "unix" geekfile.txt`

* Matching the lines that start with a string:

The ^ regular expression pattern specifies the start of a line.

`grep "^unix" geekfile.txt`

* Matching the lines that end with a string:

The $ regular expression pattern specifies the end of a line.

`grep "os$" geekfile.txt`

* Specifies multiple contents with -e option:

`grep –e "Agarwal" –e "divi" –e "jesh" geekfile.txt`

* Takes patterns from file, one per line using option -f:

`grep –f pattern.txt  geekfile.txt`

* To search for string in directory recursive ignore errors:

```mysql
sudo grep -R anon /etc/ 2> /dev/null
```

* To find files:

```mysql
sudo grep -Rl anon /etc/ 2> /dev/null
```

#### Egrep

The egrep command searches for a text pattern, using extended regular expressions to perform the match.
Running egrep is equivalent to running grep with the -E option.

Grep can understand some patterns but not all patterns.

* Display all lines start with 'D'

`grep '^D' text`

* Display all lines end with 'D'

`grep 'D$' text`

* Search for vowels in file

`grep '[aeiou]' text`

* Search for other than vowels in file

`grep '[^aeiou]' text`

* Search for multiple patterns

`grep -e 'hyd' -e 'ban' text`

or

`egrep '(hyd|ban)' text`

#### Fgrep

Fixed strings grep cannot understand patterns, it can understand only strings.

To search fast we use grep and performance wise.

`grep -F`

Some words are there in file.txt search those in demo.txt.

`fgrep -f file.txt demo.txt`

---

### **Shuffling**

shuf -i 1-100 -n 1
8

`touch pot/dir$(shuf -i 1-100 -n 1)/divi.txt`

`find pot -type f -name 'divi.txt'`

move find result to desktop

`find pot -type f -name 'divi.txt' -exec mv {} ~/Desktop \;`

Remove all files in pot but not dirs

`find pot -type f -name '*.txt' -exec rm {} \;`
