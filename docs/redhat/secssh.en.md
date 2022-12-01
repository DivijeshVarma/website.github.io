---
title: "Configuring And Securing Ssh"
---

### ACCESSING THE REMOTE COMMAND LINE WITH SSH ###

#### WHAT IS OPENSSH? ####

*OpenSSH* implements the Secure Shell or SSH protocol in the Red Hat Enterprise Linux systems.
The SSH protocol enables systems to communicate in an encrypted and secure fashion over an
insecure network.

You can use the **ssh** command to create a secure connection to a remote system, authenticate as a
specific user, and get an interactive shell session on the remote system as that user. You may also
use the **ssh** command to run an individual command on the remote system without running an
interactive shell.

#### SECURE SHELL EXAMPLES ####

The following **ssh** command would log you in on the remote server remotehost using the
same user name as the current local user. In this example, the remote system prompts you to
authenticate with that user's password.

``` bash
[user01@host ~]$ ssh remotehost
user01@remotehost's password: redhat
...output omitted...
[user01@remotehost ~]$ 
```

You can the **exit** command to log out of the remote system.

``` bash
[user01@remotehost ~]$ exit
logout
Connection to remotehost closed.
[user01@host ~]$ 
```

The next **ssh** command would log you in on the remote server remotehost using the user name
*user02*. Again, you are prompted by the remote system to authenticate with that user's password.

``` bash
[user01@host ~]$ ssh user02@remotehost
user02@remotehost's password: shadowman
...output omitted...
[user02@remotehost ~]$ 
```

This **ssh** command would run the **hostname** command on the remotehost remote system as the
user02 user without accessing the remote interactive shell.

``` bash
[user01@host ~]$ ssh user02@remotehost hostname
user02@remotehost's password: shadowman
remotehost.lab.example.com
[user01@host ~]$ 
```

Notice that the preceding command displayed the output in the local system's terminal.

#### IDENTIFYING REMOTE USERS ####

The **w** command displays a list of users currently logged into the computer. This is especially useful
to show which users are logged in using **ssh** from which remote locations, and what they are doing.

``` bash
[user01@host ~]$ ssh user01@remotehost
user01@remotehost's password: redhat
[user01@remotehost ~]$ w
 16:13:38 up 36 min,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
user02   pts/0    172.25.250.10    16:13    7:30   0.01s  0.01s -bash
user01   pts/1    172.25.250.10    16:24    3.00s  0.01s  0.00s w
[user02@remotehost ~]$ 
```

The preceding output shows that the user02 user has logged in to the system on the pseudo-
terminal 0 at **16:13** today from the host with the 172.25.250.10 IP address, and has been idle
at a shell prompt for seven minutes and thirty seconds. The preceding output also shows that the
user01 user has logged in to the system on the pseudo-terminal 1 and has been idle since since
last three seconds after executing the **w** command.

#### SSH HOST KEYS ####

SSH secures communication through public-key encryption. When an SSH client connects to an
SSH server, the server sends a copy of its public key to the client before the client logs in. This is
used to set up the secure encryption for the communication channel and to authenticate the server
to the client.

When a user uses the ssh command to connect to an **SSH** server, the command checks to see if it
has a copy of the public key for that server in its local known hosts files. The system administrator
may have pre-configured it in **/etc/ssh/ssh_known_hosts**, or the user may have a **~/.ssh/
known_hosts** file in their home directory that contains the key.

If the client has a copy of the key, **ssh** will compare the key from the known hosts files for that
server to the one it received. If the keys do not match, the client assumes that the network traffic
to the server could be hijacked or that the server has been compromised, and seeks the user's
confirmation on whether or not to continue with the connection.

!!! info "NOTE"

	Set the StrictHostKeyChecking parameter to **yes** in the user-specific **~/.ssh/
	config** file or the system-wide **/etc/ssh/ssh_config** to cause the **ssh**
	command to always abort the SSH connection if the public keys do not match.

If the client does not have a copy of the public key in its known hosts files, the **ssh** command
will ask you if you want to log in anyway. If you do, a copy of the public key will be saved in your
**~/.ssh/known_hosts** file so that the server's identity can be automatically confirmed in the
future.

``` bash
[user01@host ~]$ ssh newhost
The authenticity of host 'remotehost (172.25.250.12)' can't be established.
ECDSA key fingerprint is SHA256:qaS0PToLrqlCO2XGklA0iY7CaP7aPKimerDoaUkv720.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'newhost,172.25.250.12' (ECDSA) to the list of known
 hosts.
user01@newhost's password: redhat
...output omitted...
[user01@newhost ~]$ 
```

**SSH Known Hosts Key Management**

If a server's public key is changed because the key was lost due to hard drive failure, or replaced for
some legitimate reason, you will need to edit the known hosts files to make sure the entry for the
old public key is replaced with an entry with the new public key in order to log in without errors.

Public keys are stored in the **/etc/ssh/ssh_known_hosts** and each users' **~/.ssh/
known_hosts** file on the SSH client. Each key is on one line. The first field is a list of hostnames
and IP addresses that share that public key. The second field is the encryption algorithm for the
key. The last field is the key itself.

``` bash
[user01@host ~]$ cat ~/.ssh/known_hosts
remotehost,172.25.250.11 ecdsa-sha2-nistp256
 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOsEi0e+FlaNT6jul8Ag5Nj
+RViZl0yE2w6iYUr+1fPtOIF0EaOgFZ1LXM37VFTxdgFxHS3D5WhnIfb+68zf8+w=
```

Each remote SSH server that you conect to stores its public key in the **/etc/ssh** directory in files
with the extension **.pub**.

``` bash
[user01@remotehost ~]$ ls /etc/ssh/*key.pub
/etc/ssh/ssh_host_ecdsa_key.pub  /etc/ssh/ssh_host_ed25519_key.pub  /etc/ssh/
ssh_host_rsa_key.pub
```

!!! info "NOTE"

	It is a good practice to add entries matching a server's ssh_host_*key.pub
	files to your ~/.ssh/known_hosts file or the system-wide /etc/ssh/
	ssh_known_hosts file.

### CONFIGURING SSH KEY-BASED AUTHENTICATION ###

#### SSH KEY-BASED AUTHENTICATION ####

You can configure an SSH server to allow you to authenticate without a password by using key-
based authentication. This is based on a private-public key scheme.

To do this, you generate a matched pair of cryptographic key files. One is a private key, the other
a matching public key. The private key file is used as the authentication credential and, like a
password, must be kept secret and secure. The public key is copied to systems the user wants to
connect to, and is used to verify the private key. The public key does not need to be secret.

You put a copy of the public key in your account on the server. When you try to log in, the SSH
server can use the public key to issue a challenge that can only be correctly answered by using the
private key. As a result, your ssh client can automatically authenticate your login to the server
with your unique copy of the private key. This allows you to securely access systems in a way that
doesn't require you to enter a password interactively every time.

**Generating SSH Keys**

To create a private key and matching public key for authentication, use the **ssh-keygen**
command. By default, your private and public keys are saved in your **~/.ssh/id_rsa** and
**~/.ssh/id_rsa.pub** files, respectively.

``` bash
[user@host ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa): Enter
Created directory '/home/user/.ssh'.
Enter passphrase (empty for no passphrase): Enter
Enter same passphrase again: Enter
Your identification has been saved in /home/user/.ssh/id_rsa.
Your public key has been saved in /home/user/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:vxutUNPio3QDCyvkYm1oIx35hmMrHpPKWFdIYu3HV+w user@host.lab.example.com
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|   .     .       |
|  o o     o      |
| . = o   o .     |
|  o + = S E .    |
| ..O o + * +     |
|.+% O . + B .    |
|=*oO . . + *     |
|++.     . +.     |
+----[SHA256]-----+
```

If you do not specify a passphrase when **ssh-keygen** prompts you, the generated private key
is not protected. In this case, anyone with your private key file could use it for authentication. If
you set a passphrase, then you will need to enter that passphrase when you use the private key
for authentication. (Therefore, you would be using the private key's passphrase rather than your
password on the remote host to authenticate.)

You can run a helper program called **ssh-agent** which can temporarily cache your private key
passphrase in memory at the start of your session to get true passwordless authentication. This
will be discussed later in this section.

The following example of the **ssh-keygen** command shows the creation of the passphrase-
protected private key alongside the public key.

``` bash
[user@host ~]$ ssh-keygen -f .ssh/key-with-pass
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in .ssh/key-with-pass.
Your public key has been saved in .ssh/key-with-pass.pub.
The key fingerprint is:
SHA256:w3GGB7EyHUry4aOcNPKmhNKS7dl1YsMVLvFZJ77VxAo user@host.lab.example.com
The key's randomart image is:
+---[RSA 2048]----+
|    . + =.o ...  |
|     = B XEo o.  |
|  . o O X =....  |
| = = = B = o.    |
|= + * * S .      |
|.+ = o + .       |
|  + .            |
|                 |
|                 |
+----[SHA256]-----+
```

The **-f** option with the **ssh-keygen** command determines the files where the keys are saved. In
the preceding example, the private and public keys are saved in the **/home/user/.ssh/key-
with-pass /home/user/.ssh/key-with-pass.pub** files, respectively.

!!! danger "WARNING"

	During further SSH keypair generation, unless you specify a unique file name, you
	are prompted for permission to overwrite the existing id_rsa and id_rsa.pub
	files. If you overwrite the existing id_rsa and id_rsa.pub files, then you must
	replace the old public key with the new one on all the SSH servers that have your old
	public key.

Once the SSH keys have been generated, they are stored by default in the **.ssh/** directory of
the user's home directory. The permission modes must be 600 on the private key and 644 on the
public key.

**Sharing the Public Key**

Before key-based authentication can be used, the public key needs to be copied to the destination
system. The **ssh-copy-id** command copies the public key of the SSH keypair to the destination
system. If you omit the path to the public key file while running **ssh-copy-id**, it uses the default
**/home/user/.ssh/id_rsa.pub** file.

``` bash
[user@host ~]$ ssh-copy-id -i .ssh/key-with-pass.pub user@remotehost
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/user/.ssh/
id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter
 out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted
 now it is to install the new keys
user@remotehost's password: redhat
Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'user@remotehost'"
and check to make sure that only the key(s) you wanted were added.
```

After the public key is successfully transferred to a remote system, you can authenticate to the
remote system using the corresponding private key while logging in to the remote system over
SSH. If you omit the path to the private key file while running the ssh command, it uses the default
**/home/user/.ssh/id_rsa** file.

``` bash
[user@host ~]$ ssh -i .ssh/key-with-pass user@remotehost
Enter passphrase for key '.ssh/key-with-pass': redhatpass
...output omitted...
[user@remotehost ~]$ exit
logout
Connection to remotehost closed.
[user@host ~]$ 
```

**Using ssh-agent for Non-interactive Authentication**

If your SSH private key is protected with a passphrase, you normally have to enter the passphrase
to use the private key for authentication. However, you can use a program called **ssh-agent** to
temporarily cache the passphrase in memory. Then any time that you use SSH to log in to another
system with the private key, **ssh-agent** will automatically provide the passphrase for you. This
is convenient, and can improve security by providing fewer opportunities for someone "shoulder
surfing" to see you type the passphrase in.

Depending on your local system's configuration, if you initially log in to the GNOME graphical
desktop environment, the **ssh-agent** program might automatically be started and configured for
you.

If you log in on a text console, log in using **ssh**, or use **sudo** or **su**, you will probably need to start
**ssh-agent** manually for that session. You can do this with the following command:

``` bash
[user@host ~]$ eval $(ssh-agent)
Agent pid 10155
[user@host ~]$ 
```

!!! info "NOTE"

	When you run **ssh-agent**, it prints out some shell commands. You need to run
	these commands to set environment variables used by programs like **ssh-add** to
	communicate with it. The **eval $(ssh-agent)** command starts **ssh-agent** and
	runs those commands to automatically set those environment variables for that shell
	session. It also displays the PID of the **ssh-agent** process.

Once **ssh-agent** is running, you need to tell it the passphrase for your private key or keys. You
can do this with the **ssh-add** command.

The following **ssh-add** commands add the private keys from **/home/user/.ssh/id_rsa** (the
default) and **/home/user/.ssh/key-with-pass** files, respectively.

``` bash
[user@host ~]$ ssh-add
Identity added: /home/user/.ssh/id_rsa (user@host.lab.example.com)
[user@host ~]$ ssh-add .ssh/key-with-pass
Enter passphrase for .ssh/key-with-pass: redhatpass
Identity added: .ssh/key-with-pass (user@host.lab.example.com)
```

After successfully adding the private keys to the ssh-agent process, you can invoke an SSH
connection using the **ssh** command. If you are using any private key file other than the default
**/home/user/.ssh/id_rsa** file, then you must use the **-i** option with the **ssh** command to
specify the path to the private key file.

The following example of the **ssh** command uses the default private key file to authenticate to an
SSH server.

``` bash
[user@host ~]$ ssh user@remotehost
Last login: Fri Apr  5 10:53:50 2019 from host.example.com
[user@remotehost ~]$ 
```

The following example of the **ssh** command uses the **/home/user/.ssh/key-with-pass** (non-
default) private key file to authenticate to an SSH server. The private key in the following example
has already been decrypted and added to its parent **ssh-agent** process, so the **ssh** command
does not prompt you to decrypt the private key by interactively entering its passphrase.

``` bash
[user@host ~]$ ssh -i .ssh/key-with-pass user@remotehost
Last login: Mon Apr  8 09:44:20 2019 from host.example.com
[user@remotehost ~]$ 
```

When you log out of the session that started **ssh-agent**, the process will exit and your the
passphrases for your private keys will be cleared from memory.

### CUSTOMIZING OPENSSH SERVICE CONFIGURATION ###

#### CONFIGURING THE OPENSSH SERVER ####

OpenSSH service is provided by a daemon called sshd. Its main configuration file is **/etc/ssh/
sshd_config**.

The default configuration of the OpenSSH server works well. However, you might want to make
some changes to strengthen the security of your system. There are two common changes you
might want to make. You might want to prohibit direct remote login to the root account, and you
might want to prohibit password-based authentication (in favor of SSH private key authentication).

#### PROHIBIT THE SUPERUSER FROM LOGGING IN USING SSH ####

It is a good practice to prohibit direct login to the root user account from remote systems. Some
of the risks of allowing direct login as root include:

- The user name root exists on every Linux system by default, so a potential attacker only has
to guess the password, instead of a valid user name and password combination. This reduces
complexity for an attacker.

- The root user has unrestricted privileges, so its compromise can lead to maximum damage to
the system.

- From an auditing perspective, it can be hard to track which authorized user logged in as root
and made changes. If users have to log in as a regular user and switch to the root account, this
generates a log event that can be used to help provide accountability.

The OpenSSH server uses the **PermitRootLogin** configuration setting in the **/etc/ssh/
sshd_config** configuration file to allow or prohibit users logging in to the system as root.

`PermitRootLogin yes`

With the **PermitRootLogin** parameter to **yes**, as it is by default, people are permitted to log in as
root. To prevent this, set the value to **no**. Alternatively, to prevent password-based authentication
but allow private key-based authentication for root, set the **PermitRootLogin** parameter to
**without-password**.

The SSH server (sshd) must be reloaded for any changes to take effect.

``` bash
[root@host ~]# systemctl reload sshd
```

!!! danger "IMPORTANT"

	The advantage of using systemctl reload sshd command is that it tells sshd
	to re-read its configuration file rather than completely restarting the service. A
	systemctl restart sshd command would also apply the changes, but would
	also stop and start the service, breaking all active SSH connections to that host.

#### PROHIBITING PASSWORD-BASED AUTHENTICATION FOR SSH ####

Allowing only private key-based logins to the remote command line has various advantages:

- Attackers cannot use password guessing attacks to remotely break into known accounts on the
system.

- With passphrase-protected private keys, an attacker needs both the passphrase and a copy of
the private key. With passwords, an attacker just needs the password.

- By using passphrase-protected private keys in conjunction with ssh-agent, the passphrase is
exposed less frequently since it is entered less frequently, and logging in is more convenient for
the user.

The OpenSSH server uses the PasswordAuthentication parameter in the **/etc/ssh/
sshd_config** configuration file to control whether users can use password-based authentication
to log in to the system.

`PasswordAuthentication yes`

The default value of **yes** for the PasswordAuthentication parameter in the **/etc/ssh/
sshd_config** configuration file causes the SSH server to allow users to use password-based
authentication while logging in. The value of no for PasswordAuthentication prevents users
from using password-based authentication.

Keep in mind that whenever you change the **/etc/ssh/sshd_config** file, you must reload the
sshd service for changes to take effect.

!!! danger "IMPORTANT"

	Remember, if you turn off password-based authentication for ssh, you need to have
	a way to ensure that the user's ~/.ssh/authorized_keys file on the remote
	server is populated with their public key, so that they can log in.

### SUMMARY ###

In this chapter, you learned:

- The **ssh** command allows users to access remote systems securely using the SSH protocol.

- A client system stores remote servers' identities in **~/.ssh/known_hosts** and **/etc/ssh/
ssh_known_hosts**.

- SSH supports both password-based and key-based authentication.

- The **ssh-keygen** command generates an SSH key pair for authentication. The **ssh-copy-id**
command exports the public key to remote systems.

- The sshd service implements the SSH protocol on Red Hat Enterprise Linux systems.

- It is a recommended practice to configure sshd to disable remote logins as root and to require
public key authentication rather than password-based authentication.

### LAB

pdf - 359