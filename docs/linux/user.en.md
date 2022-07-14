---
title: "User Management"
---

A user is an entity, in a Linux operating system, that can manipulate files and perform several other operations. Each user is assigned an ID that is unique for each user in the operating system. After installation of the operating system, the ID 0 is assigned to the root user and the IDs 1 to 999 (both inclusive) are assigned to the system users and hence the ids for local user begins from 1000 onwards. 

In a single directory, we can create 60,000 users. Now we will discuss the important commands to manage users in Linux.

**Useradd**
Reference: [Link](https://www.tecmint.com/add-users-in-linux/)

**Usermod**
Reference: [Link](https://www.tecmint.com/usermod-command-examples/)

* To list out all the users in Linux, use the awk command with -F option. Here, we are accessing a file and printing only first column with the help of print $1 and awk.

```bash
awk -F':' '{ print $1}' /etc/passwd
```

**Using id command, you can get the ID of any username.**

```bash
id username
```

**The command to add a user**

```bash
sudo useradd username
```

**Add description/comment to a user account.**

```bash
sudo useradd -c "John Wise" john
```

**Set the home directory for the specified user**
```bash
sudo useradd -d /mnt/home/john
```

**For instance some of parameters to add a new user**
```bash
useradd -g tech -G apple,linux -s /bin/zsh -c "James Adem" adem
```
`-g: Allows you to set the primary group of a user. The user will be added to a group by default if you don't add one during the creation process.`

`-G: Adds the user to multiple groups.`

`-s: Sets Zsh as the default shell for the user `

**You can also view/modify the default settings using the useradd command.**
```bash
useradd -D
```
`-b  Modifies the default home directory (/home) for new user accounts.`

`-g  Modifies the default new user primary group (username) with another default group.`

`-s  Replaces the default /bin/bash shell with another default shell.`

`-e  Modifies the default expiration date to disable a user account in YYYY-MM-DD format.`

`-f  Allows to set inactive days before the account is disabled and after password expiration`

**For instance**
```bash
useradd -D -b /home/new -s /bin/sh
```

**User with Account Expiry Date**
```bash
useradd -e 2024-03-27 divi
```

**Create a User with Password Expiry Date**

We will set an account password expiry date i.e. 45 days on a user ‘mansi‘ using ‘-e‘ and ‘-f‘ options.
```bash
useradd -e 2024-04-27 -f 45 mansi
```

**Using passwd command to assign a password to a user.**
```bash
passwd username
```

**Accessing a user configuration file.**
```bash
cat /etc/passwd
```
`username:password:UID:GID:comment:home:shell`

**Create user with specific UID and GID**
```bash
useradd -u 1000 -g 500 divi
```

**To change the user ID for a user.**
```bash
sudo usermod -u 1982 test
```

**Modify the group ID of a user.**

This command can change the group ID of a user and hence it can even be used to move a user to an already existing group.
```bash
sudo usermod -g 1005 test
```

**Change the user login name**
```bash
sudo usermod -l new_login_name old_login_name
```

**Command to change the home directory**
```bash
usermod -d new_home_directory_path username
```

**Delete a user**

Make sure that the user is not part of a group. If the user is part of a group then it will not be deleted directly, hence we will have to first remove him from the group and then we can delete him.
```bash
userdel -r username
```

**To include adem in the sales group**

You'll need to use the -aG flag as a simple -G flag will remove the user from the previously added supplementary groups
```bash
usermod -aG sales adem
```

**Discovering the versatile passwd command**

Reference: [Link](https://www.redhat.com/sysadmin/linux-user-account-management)

**Verify the age of account**
```bash
chage -l divi
```




```bash

```
```bash

```
```bash

```
