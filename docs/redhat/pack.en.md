---
title: "Installing And Updating Software Packages"
---

### REGISTERING SYSTEMS FOR RED HAT SUPPORT ###

#### RED HAT SUBSCRIPTION MANAGEMENT ####

Red Hat Subscription Management provides tools that can be used to entitle machines to product
subscriptions, allowing administrators to get updates to software packages and track information
about support contracts and subscriptions used by the systems. Standard tools such as *PackageKit*
and **yum** can obtain software packages and updates through a content distribution network
provided by Red Hat.

There are four basic tasks performed with Red Hat Subscription Management tools:

- **Register** a system to associate that system to a Red Hat account. This allows Subscription
Manager to uniquely inventory the system. When no longer in use, a system may be
unregistered.

- **Subscribe** a system to entitle it to updates for selected Red Hat products. Subscriptions have
specific levels of support, expiration dates, and default repositories. The tools can be used to
either auto-attach or select a specific entitlement. As needs change, subscriptions may be
removed.

- **Enable repositories** to provide software packages. Multiple repositories are enabled by default
with each subscription, but other repositories such as updates or source code can be enabled or
disabled as needed.

- **Review and track** entitlements that are available or consumed. Subscription information can
be viewed locally on a specific system or, for an account, in either the Red Hat Customer Portal
Subscriptions page or the Subscription Asset Manager (SAM).

**Registering a System**

There are a number of different ways to register a system with Red Hat Customer Portal. There is
a graphical interface that you can access with a GNOME application or through the Web Console
service, and there is a command-line tool.

To register a system with the GNOME application, launch Red Hat Subscription Manager
by selecting Activities. Type subscription in the Type to search... field and click on Red Hat
Subscription Manager. Enter the appropriate password when prompted to authenticate. This
displays the following Subscriptions window:

![The main window of Red Hat Subscription Manager](https://course.btech.id/media/markdownx/111a996b-7a21-4377-96b9-e9422177ee57.png "The main window of Red Hat Subscription Manager")

To register the system, click the Register button in the Subscriptions window. This displays the
following dialog:

![The service location and account information dialog of Red Hat Subscription Manager](https://course.btech.id/media/markdownx/84090ec2-44e7-4cf4-bc72-fca1468b8c7c.png "The service location and account information dialog of Red Hat Subscription Manager")

This dialog box registers a system with a subscription server. By default, it registers the server
to Red Hat Customer Portal. Provide the Login and Password for the Red Hat Customer Portal
account to which the system should be registered, and click the Register button.

When registered, the system automatically has a subscription attached if one is available.

After the system is registered and a subscription has been assigned, close the Subscriptions
window. The system is now properly subscribed and ready to receive updates or install new
software from Red Hat.

#### REGISTRATION FROM THE COMMAND LINE ####

Use the **subscription-manager(8)** to register a system without using a graphical environment.
The **subscription-manager** command can automatically attach a system to the best-matched
compatible subscriptions for the system.

- Register a system to a Red Hat account:

```bash
[user@host ~]$ subscription-manager register --username=yourusername --
password=yourpassword
```

- View available subscriptions:

``` bash
[user@host ~]$ subscription-manager list --available | less
```

- Auto-attach a subscription:

``` bash
[user@host ~]$ subscription-manager attach --auto
```

- Alternatively, attach a subscription from a specific pool from the list of available subscriptions:

``` bash
[user@host ~]$ subscription-manager attach --pool=poolID
```

- View consumed subscriptions:

``` bash
[user@host ~]$ subscription-manager list --consumed
```

- Unregister a system:

``` bash
[user@host ~]$ subscription-manager unregister
```

!!! info "NOTE"

	**subscription-manager** can also be used in conjunction with activation keys,
	allowing registration and assignment of predefined subscriptions, without using a
	username or password. This method of registration can be very useful for automated
	installations and deployments. Activation keys are often issued by an on-premise
	subscription management service, such as Subscription Asset Manager or Red Hat
	Satellite, and are not discussed in detail in this course.

#### ENTITLEMENT CERTIFICATES ####

An entitlement is a subscription that has been attached to a system. Digital certificates are used
to store current information about entitlements on the local system. Once registered, entitlement
certificates are stored in **/etc/pki** and its subdirectories.

- **/etc/pki/product** contains certificates indicating which Red Hat products are installed on
the system.

- **/etc/pki/consumer** contains certificates identifying the Red Hat account to which the system
is registered.

- **/etc/pki/entitlement** contains certificates indicating which subscriptions are attached to
the system.

The certificates can be inspected with the **rct** utility directly, but the **subscription-manager**
tools provide easier ways to examine the subscriptions attached to the system.

### EXPLAINING AND INVESTIGATING RPM SOFTWARE PACKAGES ###

#### SOFTWARE PACKAGES AND RPM ####

The RPM Package Manager, originally developed by Red Hat, provides a standard way to package
software for distribution. Managing software in the form of RPM packages is much simpler than
working with software that has simply been extracted into a file system from an archive. It lets
administrators track which files were installed by the software package and which ones need to be
removed if it is uninstalled, and check to ensure that supporting packages are present when it is
installed. Information about installed packages is stored in a local RPM database on each system.
All software provided by Red Hat for Red Hat Enterprise Linux is provided as an RPM package.

RPM package files names consist of four elements (plus the **.rpm** suffix): **name-version-
release.architecture**:

![RPM file name elements](https://course.btech.id/media/markdownx/98c5233a-3127-40c2-9730-75b182090127.png "RPM file name elements")

- NAME is one or more words describing the contents (coreutils).

- VERSION is the version number of the original software (8.30).

- RELEASE is the release number of the package based on that version, and is set by the packager,
who might not be the original software developer (4.el8).

- ARCH is the processor architecture the package was compiled to run on. **noarch** indicates
that this package's contents are not architecture-specific (as opposed to **x86_64** for 64-bit,
**aarch64** for 64-bit ARM, and so on).

Only the package name is required for installing packages from repositories. If multiple versions
exist, the package with the higher version number is installed. If multiple releases of a single
version exist, the package with the higher release number is installed.

Each RPM package is a special archive made up of three components:

- The files installed by the package.

- Information about the package (metadata), such as the name, version, release, and arch; a
summary and description of the package; whether it requires other packages to be installed;
licensing; a package change log; and other details.

- Scripts that may run when this package is installed, updated, or removed, or are triggered when
other packages are installed, updated, or removed.

Typically, software providers digitally sign RPM packages using GPG keys (Red Hat digitally
signs all packages it releases). The RPM system verifies package integrity by confirming that the
package was signed by the appropriate GPG key. The RPM system refuses to install a package if
the GPG signature does not match.

**Updating Software with RPM Packages**

Red Hat generates a complete RPM package to update software. An administrator installing that
package gets only the most recent version of the package. Red Hat does not require that older
packages be installed and then patched. To update software, RPM removes the older version of the
package and installs the new version. Updates usually retain configuration files, but the packager
of the new version defines the exact behavior.

In most cases, only one version or release of a package may be installed at a time. However, if a
package is built so that there are no conflicting file names, then multiple versions may be installed.
The most important example of this is the **kernel** package. Since a new kernel can only be tested
by booting to that kernel, the package is specifically designed so that multiple versions may be
installed at once. If the new kernel fails to boot, the old kernel is still available and bootable.

#### EXAMINING RPM PACKAGES ####

The **rpm** utility is a low-level tool that can get information about the contents of package files
and installed packages. By default, it gets information from the local database of installed
packages. However, you can use the **-p** option to specify that you want to get information about a
downloaded package file. You might want to do this in order to inspect the contents of the package
file before installing it.

The general form of a query is:

- **rpm -q [select-options] [query-options**]

**RPM queries: General information about installed packages**

- **rpm -qa:** List all installed packages

- **rpm -qf FILENAME:** Find out what package provides FILENAME

``` bash
[user@host ~]$ rpm -qf /etc/yum.repos.d
redhat-release-8.0-0.39.el8.x86_64
```

**RPM queries: Information about specific packages**

- **rpm -q:** List what version of the package is currently installed

``` bash
[user@host ~]$ rpm -q yum
yum-4.0.9.2-4.el8.noarch
```

- **rpm -qi:** Get detailed information about the package

- **rpm -ql:** List the files installed by the package

``` bash
[user@host ~]$ rpm -ql yum
/etc/yum.conf
/etc/yum/pluginconf.d
/etc/yum/protected.d
/etc/yum/vars
/usr/bin/yum
/usr/share/man/man1/yum-aliases.1.gz
/usr/share/man/man5/yum.conf.5.gz
/usr/share/man/man8/yum-shell.8.gz
/usr/share/man/man8/yum.8.gz
```

- **rpm -qc:** List just the configuration files installed by the package

``` bash
[user@host ~]$ rpm -qc openssh-clients
/etc/ssh/ssh_config
/etc/ssh/ssh_config.d/05-redhat.conf
```

- **rpm -qd:** List just the documentation files installed by the package

``` bash
[user@host ~]$ rpm -qd openssh-clients
/usr/share/man/man1/scp.1.gz
/usr/share/man/man1/sftp.1.gz
/usr/share/man/man1/ssh-add.1.gz
/usr/share/man/man1/ssh-agent.1.gz
/usr/share/man/man1/ssh-copy-id.1.gz
/usr/share/man/man1/ssh-keyscan.1.gz
/usr/share/man/man1/ssh.1.gz
/usr/share/man/man5/ssh_config.5.gz
/usr/share/man/man8/ssh-pkcs11-helper.8.gz
```

- **rpm -q --scripts:** List shell scripts that run before or after the package is installed or
removed

``` bash
[user@host ~]$ rpm -q --scripts openssh-server
preinstall scriptlet (using /bin/sh):
getent group sshd >/dev/null || groupadd -g 74 -r sshd || :
getent passwd sshd >/dev/null || \
  useradd -c "Privilege-separated SSH" -u 74 -g sshd \
  -s /sbin/nologin -r -d /var/empty/sshd sshd 2> /dev/null || :
postinstall scriptlet (using /bin/sh):
if [ $1 -eq 1 ] ; then 
        # Initial installation 
        /usr/bin/systemctl preset sshd.service sshd.socket >/dev/null 2>&1 || : 
fi
preuninstall scriptlet (using /bin/sh):
if [ $1 -eq 0 ] ; then 
        # Package removal, not upgrade 
        /usr/bin/systemctl --no-reload disable sshd.service sshd.socket > /dev/
null 2>&1 || : 
        /usr/bin/systemctl stop sshd.service sshd.socket > /dev/null 2>&1 || : 
fi
postuninstall scriptlet (using /bin/sh):
/usr/bin/systemctl daemon-reload >/dev/null 2>&1 || : 
if [ $1 -ge 1 ] ; then 
        # Package upgrade, not uninstall 
        /usr/bin/systemctl try-restart sshd.service >/dev/null 2>&1 || : 
fi
```

- **rpm -q --changelog:** list change information for the package

``` bash
[user@host ~]$ rpm -q --changelog audit
* Wed Jan 09 2019 Steve Grubb <sgrubb@redhat.com> 3.0-0.10.20180831git0047a6c
resolves: rhbz#1655270] Message "audit: backlog limit exceeded" reported
- Fix annobin failure
* Fri Dec 07 2018 Steve Grubb <sgrubb@redhat.com> 3.0-0.8.20180831git0047a6c
resolves: rhbz#1639745 - build requires go-toolset-7 which is not available
resolves: rhbz#1643567 - service auditd stop exits prematurely
resolves: rhbz#1616428 - Update git snapshot of audit package
- Remove static libs subpackage
...output omitted...
```

Querying local package files:

``` bash
[user@host ~]$ ls -l wonderwidgets-1.0-4.x86_64.rpm
-rw-rw-r--. 1 user user 257 Mar 13 20:06 wonderwidgets-1.0-4.x86_64.rpm
[user@host ~]$ rpm -qlp wonderwidgets-1.0-4.x86_64.rpm
/etc/wonderwidgets.conf
/usr/bin/wonderwidgets
/usr/share/doc/wonderwidgets-1.0
/usr/share/doc/wonderwidgets-1.0/README.txt
```

### INSTALLING RPM PACKAGES ###

The **rpm** command can also be used to install an RPM package that you have downloaded to your
local directory.

``` bash
[root@host ~]# rpm -ivh wonderwidgets-1.0-4.x86_64.rpm
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:wonderwidgets-1.0-4              ################################# [100%]
[root@host ~]# 
```

However, the next section of this chapter will discuss a more powerful tool for managing RPM
installation and updates from the command line, **yum**.

!!! danger "WARNING"

	Be careful when installing packages from third parties, not just because of the
	software that they may install, but because the RPM package may include arbitrary
	scripts that run as the root user as part of the installation process.

!!! info "NOTE"

	You can extract files from an RPM package file without installing the package. The
	**rpm2cpio** utility can pass the contents of the RPM to a special archiving tool called
	**cpio**, which can extract all files or individual files.
	Pipe the output of **rpm2cpio PACKAGEFILE.rpm** into **cpio -id**, to extract all
	files stored in the RPM package. Subdirectory trees are created as needed, relative
	to the current working directory.
	
	``` bash	
	[user@host tmp-extract]$ rpm2cpio wonderwidgets-1.0-4.x86_64.rpm | cpio -id
	```

	Individual files are extracted by specifying the path of the file:
	
	``` bash	
	[user@host ~]$ rpm2cpio wonderwidgets-1.0-4.x86_64.rpm | cpio -id "*txt"
	11 blocks
	[user@host ~]$ ls -l usr/share/doc/wonderwidgets-1.0/
	total 4
	-rw-r--r--. 1 user user 76 Feb 13 19:27 README.txt
	```

#### SUMMARY OF RPM QUERY COMMANDS ####

Installed packages can be queried directly with the **rpm** command. Add the **-p** option to query a
package file before installation.

| COMMAND                 | TASK                                                                       |
|:------------------------|:---------------------------------------------------------------------------|
| rpm -qa                 | List all RPM packages currently installed                                  |
| rpm -q NAME             | Display the version of NAME installed on the system                        |
| rpm -qi NAME            | Display detailed information about a package                               |
| rpm -ql NAME            | List all files included in a package                                       |
| rpm -qc NAME            | List configuration files included in a package                             |
| rpm -qd NAME            | List documentation files included in a package                             |
| rpm -q --changelog NAME | Show a short summary of the reason for a new package release               |
| rpm -q --scripts NAME   | Display the shell scripts run on package installation, upgrade, or removal |

### INSTALLING AND UPDATING SOFTWARE PACKAGES WITH YUM ###

#### MANAGING SOFTWARE PACKAGES WITH YUM ####

The low-level **rpm** command can be used to install packages, but it is not designed to work with
package repositories or resolve dependencies from multiple sources automatically.

Yum is designed to be a better system for managing RPM-based software installation and updates.
The **yum** command allows you to install, update, remove, and get information about software
packages and their dependencies. You can get a history of transactions performed and work with
multiple Red Hat and third-party software repositories.

**Finding Software with Yum**

- **yum help** displays usage information.

- **yum list** displays installed and available packages.

``` bash
[user@host ~]$ yum list 'http*'
Available Packages
http-parser.i686              2.8.0-2.el8                        rhel8-appstream
http-parser.x86_64            2.8.0-2.el8                        rhel8-appstream
httpcomponents-client.noarch  4.5.5-4.module+el8+2452+b359bfcd   rhel8-appstream
httpcomponents-core.noarch    4.4.10-3.module+el8+2452+b359bfcd  rhel8-appstream
httpd.x86_64                  2.4.37-7.module+el8+2443+605475b7  rhel8-appstream
httpd-devel.x86_64            2.4.37-7.module+el8+2443+605475b7  rhel8-appstream
httpd-filesystem.noarch       2.4.37-7.module+el8+2443+605475b7  rhel8-appstream
httpd-manual.noarch           2.4.37-7.module+el8+2443+605475b7  rhel8-appstream
httpd-tools.x86_64            2.4.37-7.module+el8+2443+605475b7  rhel8-appstream
```

- **yum search KEYWORD** lists packages by keywords found in the name and summary fields only.

To search for packages that have “web server” in their name, summary, and description fields,
use **search all**:

``` bash
[user@host ~]$ yum search all 'web server'
================= Summary & Description Matched: web server ====================
pcp-pmda-weblog.x86_64 : Performance Co-Pilot (PCP) metrics from web server logs
nginx.x86_64 : A high performance web server and reverse proxy server
======================== Summary Matched: web server ===========================
libcurl.x86_64 : A library for getting files from web servers
libcurl.i686 : A library for getting files from web servers
libcurl.x86_64 : A library for getting files from web servers
====================== Description Matched: web server =========================
httpd.x86_64 : Apache HTTP Server
git-instaweb.x86_64 : Repository browser in gitweb
...output omitted...
```

- **yum info PACKAGENAME** returns detailed information about a package, including the disk
space needed for installation.

To get information on the Apache HTTP Server:

``` bash
[user@host ~]$ yum info httpd
Available Packages
Name         : httpd
Version      : 2.4.37
Release      : 7.module+el8+2443+605475b7
Arch         : x86_64
Size         : 1.4 M
Source       : httpd-2.4.37-7.module+el8+2443+605475b7.src.rpm
Repo         : rhel8-appstream
Summary      : Apache HTTP Server
URL          : https://httpd.apache.org/
License      : ASL 2.0
Description  : The Apache HTTP Server is a powerful, efficient, and extensible
             : web server.
```

- **yum provides PATHNAME** displays packages that match the path name specified (which often
include wildcard characters).

To find packages that provide the **/var/www/html** directory, use:

``` bash
[user@host ~]$ yum provides /var/www/html
httpd-filesystem-2.4.37-7.module+el8+2443+605475b7.noarch : The basic directory
 layout for the Apache HTTP server
Repo        : rhel8-appstream
Matched from:
Filename    : /var/www/html
```

**Installing and removing software with yum**

- **yum install PACKAGENAME** obtains and installs a software package, including any
dependencies.

``` bash
[user@host ~]$ yum install httpd
Dependencies resolved.
================================================================================
 Package                  Arch       Version             Repository        Size
================================================================================
Installing:
 httpd                    x86_64     2.4.37-7.module...  rhel8-appstream   1.4 M
Installing dependencies:
 apr                      x86_64     1.6.3-8.el8         rhel8-appstream   125 k
 apr-util                 x86_64     1.6.1-6.el8         rhel8-appstream   105 k
...output omitted...
Transaction Summary
================================================================================
Install  9 Packages
Total download size: 2.0 M
Installed size: 5.4 M
Is this ok [y/N]: y
Downloading Packages:
(1/9): apr-util-bdb-1.6.1-6.el8.x86_64.rpm           464 kB/s |  25 kB     00:00
(2/9): apr-1.6.3-8.el8.x86_64.rpm                    1.9 MB/s | 125 kB     00:00
(3/9): apr-util-1.6.1-6.el8.x86_64.rpm               1.3 MB/s | 105 kB     00:00
...output omitted...
Total                                                8.6 MB/s | 2.0 MB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                         1/1
  Installing       : apr-1.6.3-8.el8.x86_64                                  1/9
  Running scriptlet: apr-1.6.3-8.el8.x86_64                                  1/9
  Installing       : apr-util-bdb-1.6.1-6.el8.x86_64                         2/9
...output omitted...
Installed:
  httpd-2.4.37-7.module+el8+2443+605475b7.x86_64 apr-util-bdb-1.6.1-6.el8.x86_64
  apr-util-openssl-1.6.1-6.el8.x86_64            apr-1.6.3-8.el8.x86_64
...output omitted...
Complete!
```

- **yum update PACKAGENAME** obtains and installs a newer version of the specified package,
including any dependencies. Generally the process tries to preserve configuration files in place,
but in some cases, they may be renamed if the packager thinks the old one will not work after the
update. With no PACKAGENAME specified, it installs all relevant updates.

``` bash
[user@host ~]$ sudo yum update
```

Since a new kernel can only be tested by booting to that kernel, the package is specifically
designed so that multiple versions may be installed at once. If the new kernel fails to boot, the
old kernel is still available. Using **yum update kernel** will actually install the new kernel. The
configuration files hold a list of packages to always install even if the administrator requests an
update.

!!! info "NOTE"

	Use **yum list kernel** to list all installed and available kernels. To view the
	currently running kernel, use the **uname** command. The -r option only shows
	the kernel version and release, and the **-a** option shows the kernel release and
	additional information.

	[user@host ~]$ yum list kernel
	Installed Packages
	kernel.x86_64         4.18.0-60.el8         @anaconda
	kernel.x86_64         4.18.0-67.el8         @rhel-8-for-x86_64-baseos-htb-rpms
	[user@host ~]$ uname -r
	4.18.0-60.el8.x86_64
	[user@host ~]$ uname -a
	Linux host.lab.example.com 4.18.0-60.el8.x86_64 #1 SMP Fri Jan 11 19:08:11 UTC
	2019 x86_64 x86_64 x86_64 GNU/Linux
 
- **yum remove PACKAGENAME** removes an installed software package, including any supported
packages.

``` bash
[user@host ~]$ sudo yum remove httpd
```

!!! danger "WARNING"

	The **yum remove** command removes the packages listed and any package that
	requires the packages being removed (and packages which require those packages,
	and so on). This can lead to unexpected removal of packages, so carefully review the
	list of packages to be removed.

**Installing and removing groups of software with yum**

- **yum** also has the concept of groups, which are collections of related software installed together
for a particular purpose. In Red Hat Enterprise Linux 8, there are two kinds of groups. Regular
groups are collections of packages. Environment groups are collections of regular groups. The
packages or groups provided by a group may be **mandatory** (they must be installed if the group
is installed), **default** (normally installed if the group is installed), or **optional** (not installed
when the group is installed, unless specifically requested).

Like **yum list**, the **yum group list** command shows the names of installed and available
groups.

``` bash
[user@host ~]$ yum group list
Available Environment Groups:
   Server with GUI
   Minimal Install
   Server
...output omitted...
Available Groups:
   Container Management
   .NET Core Development
   RPM Development Tools
...output omitted...
```

Some groups are normally installed through environment groups and are hidden by default. List
these hidden groups with the **yum group list hidden** command.

- **yum group info** displays information about a group. It includes a list of mandatory, default,
and optional package names.

``` bash
[user@host ~]$ yum group info "RPM Development Tools"
Group: RPM Development Tools
 Description: These tools include core development tools such rpmbuild.
  Mandatory Packages:
    redhat-rpm-config
    rpm-build
  Default Packages:
    rpmdevtools
  Optional Packages:
    rpmlint
```

- **yum group install** installs a group that installs its mandatory and default packages and the
packages they depend on.

``` bash
[user@host ~]$ sudo yum group install "RPM Development Tools"
...output omitted...
Installing Groups:
 RPM Development Tools
 Transaction Summary
 ===============================================================================
```

!!! danger "IMPORTANT"

	The behavior of Yum groups changed starting in Red Hat Enterprise Linux 7. In
	RHEL 7 and later, groups are treated as objects, and are tracked by the system. If an
	installed group is updated, and new mandatory or default packages have been added
	to the group by the Yum repository, those new packages are installed upon update.
	
	RHEL 6 and earlier consider a group to be installed if all its mandatory packages
	have been installed, or if it had no mandatory packages, or if any default or optional
	packages in the group are installed. Starting in RHEL 7, a group is considered to be
	installed only if **yum group install** was used to install it. The command **yum
	group mark install GROUPNAME** can be used to mark a group as installed, and
	any missing packages and their dependencies are installed upon the next update.
	
	Finally, RHEL 6 and earlier did not have the two-word form of the **yum group**
	commands. In other words, in RHEL 6 the command **yum grouplist** existed, but
	the equivalent RHEL 7 and RHEL 8 command **yum group list** did not.

**Viewing transaction history**

- All install and remove transactions are logged in **/var/log/dnf.rpm.log**.

``` bash
[user@host ~]$ tail -5 /var/log/dnf.rpm.log
2019-02-26T18:27:00Z SUBDEBUG Installed: rpm-build-4.14.2-9.el8.x86_64
2019-02-26T18:27:01Z SUBDEBUG Installed: rpm-build-4.14.2-9.el8.x86_64
2019-02-26T18:27:01Z SUBDEBUG Installed: rpmdevtools-8.10-7.el8.noarch
2019-02-26T18:27:01Z SUBDEBUG Installed: rpmdevtools-8.10-7.el8.noarch
2019-02-26T18:38:40Z INFO --- logging initialized ---
```

- **yum history** displays a summary of install and remove transactions.

``` bash
[user@host ~]$ sudo yum history
ID     | Command line             | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------
     7 | group install RPM Develo | 2019-02-26 13:26 | Install        |   65
     6 | update kernel            | 2019-02-26 11:41 | Install        |    4
     5 | install httpd            | 2019-02-25 14:31 | Install        |    9
     4 | -y install @base firewal | 2019-02-04 11:27 | Install        |  127 EE
     3 | -C -y remove firewalld - | 2019-01-16 13:12 | Removed        |   11 EE
     2 | -C -y remove linux-firmw | 2019-01-16 13:12 | Removed        |    1
     1 |                          | 2019-01-16 13:05 | Install        |  447 EE
```

- The **history undo** option reverses a transaction.

``` bash
[user@host ~]$ sudo yum history undo 5
Undoing transaction 7, from Tue 26 Feb 2019 10:40:32 AM EST
    Install apr-1.6.3-8.el8.x86_64                              @rhel8-appstream
    Install apr-util-1.6.1-6.el8.x86_64                         @rhel8-appstream
    Install apr-util-bdb-1.6.1-6.el8.x86_64                     @rhel8-appstream
    Install apr-util-openssl-1.6.1-6.el8.x86_64                 @rhel8-appstream
    Install httpd-2.4.37-7.module+el8+2443+605475b7.x86_64      @rhel8-appstream
...output omitted...
```

#### SUMMARY OF YUM COMMANDS ####

Packages can be located, installed, updated, and removed by name or by package groups.

| TASK:                                         | COMMAND:                    |
|:----------------------------------------------|:----------------------------|
| List installed and available packages by name | yum list [NAME-PATTERN]     |
| List installed and available groups           | yum group list              |
| Search for a package by keyword               | yum search KEYWORD          |
| Show details of a package                     | yum info PACKAGENAME        |
| Install a package                             | yum install PACKAGENAME     |
| Install a package group                       | yum group install GROUPNAME |
| Update all packages                           | yum update                  |
| Remove a package                              | yum remove PACKAGENAME      |
| Display transaction history                   | yum history                 |

### ENABLING YUM SOFTWARE REPOSITORIES ###

#### ENABLING RED HAT SOFTWARE REPOSITORIES ####

Registering a system to the subscription management service automatically configures access to
software repositories based on the attached subscriptions. To view all available repositories:

``` bash
[user@host ~]$ yum repolist all
Loaded plugins: langpacks
repo id                                    repo name                              
          status
rhel-8-server-debug-rpms/8Server/x86_64    Red Hat Enterprise Linux 8 Server
 (Debug RPMs)   disabled
rhel-8-server-rpms/8Server/x86_64          Red Hat Enterprise Linux 8 Server
 (RPMs)         enabled: 5,071
rhel-8-server-source-rpms/8Server/x86_64   Red Hat Enterprise Linux 8 Server
 (Source RPMs)  disabled
repolist: 5,071
```

The **yum-config-manager** command can be used to enable or disable repositories. To enable a
repository, the command sets the enabled parameter to **1**. For example, the following command
enables the **rhel-8-server-debug-rpms** repository:

``` bash
[user@host ~]$ yum-config-manager --enable rhel-8-server-debug-rpms
Loaded plugins: langpacks
===================== repo: rhel-8-server-debug-rpms ======================
[rhel-8-server-debug-rpms]
async = True
bandwidth = 0
base_persistdir = /var/lib/yum/repos/x86_64/8Server
baseurl = https://cdn.redhat.com/content/dist/rhel/server/8/8Server/x86_64/debug
cache = 0
cachedir = /var/cache/yum/x86_64/8Server/rhel-8-server-debug-rpms
check_config_file_age = True
cost = 1000
deltarpm_percentage =
enabled = 1
...output omitted...
```

Non-Red Hat sources provide software through third-party repositories, which can be accessed
by the **yum** command from a website, FTP server, or the local file system. For example, Adobe
provides some of its software for Linux through a Yum repository. In a Red Hat classroom, the
content.example.com classroom server hosts Yum repositories.

To enable support for a new third-party repository, create a file in the **/etc/yum.repos.d/**
directory. Repository configuration files must end with a **.repo** extension. The repository
definition contains the URL of the repository, a name, whether to use GPG to check the package
signatures, and if so, the URL pointing to the trusted GPG key.

**Creating Yum Repositories**

Create Yum repositories with the yum-config-manager command.
The following command creates a file named /etc/yum.repos.d/
dl.fedoraproject.org_pub_epel_8_x86_64_.repo with the output shown.

``` bash
[user@host ~]$ yum-config-manager --add-repo="http://dl.fedoraproject.org/pub/
epel/8/x86_64/"
Loaded plugins: langpacks
adding repo from: http://dl.fedoraproject.org/pub/epel/8/x86_64/
[dl.fedoraproject.org_pub_epel_8_x86_64_]
name=added from: http://dl.fedoraproject.org/pub/epel/8/x86_64/
baseurl=http://dl.fedoraproject.org/pub/epel/8/x86_64/
enabled=1
```

Modify this file to provide customized values and location of a GPG key. Keys are stored in various
locations on the remote repository site, such as, http://dl.fedoraproject.org/pub/
epel/RPM-GPG-KEY-EPEL-8. Administrators should download the key to a local file rather than
allowing **yum** to retrieve the key from an external source. For example:

``` bash
[EPEL]
name=EPEL 8
baseurl=http://dl.fedoraproject.org/pub/epel/8/x86_64/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
```

**RPM Configuration Packages for Local Repositories**

Some repositories provide a configuration file and GPG public key as part of an RPM package
that can be downloaded and installed using the **yum localinstall** command. For example,
the volunteer project called Extra Packages for Enterprise Linux (EPEL) provides software not
supported by Red Hat but compatible with Red Hat Enterprise Linux.

The following command installs the Red Hat Enterprise Linux 8 EPEL repo package:

``` bash
[user@host ~]$ rpm --import http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-
EPEL-8
[user@host ~]$ yum install http://dl.fedoraproject.org/pub/epel/8/x86_64/e/epel-
release-8-2.noarch.rpm
```

Configuration files often list multiple repository references in a single file. Each repository
reference begins with a single-word name in square brackets.

``` bash
[user@host ~]$ cat /etc/yum.repos.d/epel.repo
[epel]
name=Extra Packages for Enterprise Linux 8 - $basearch
#baseurl=http://download.fedoraproject.org/pub/epel/8/$basearch
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-8&arch=$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
[epel-debuginfo]
name=Extra Packages for Enterprise Linux 8 - $basearch - Debug
#baseurl=http://download.fedoraproject.org/pub/epel/8/$basearch/debug
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-8&arch=
$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
gpgcheck=1
[epel-source]
name=Extra Packages for Enterprise Linux 8 - $basearch - Source
#baseurl=http://download.fedoraproject.org/pub/epel/8/SRPMS
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-source-8&arch=
$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
gpgcheck=1
```

To define a repository, but not search it by default, insert the enabled=0 parameter. Repositories
can be enabled and disabled persistently with **yum-config-manager** command or temporarily
with yum command options, **--enablerepo=PATTERN** and **--disablerepo=PATTERN**.

!!! danger "WARNING"

	Install the RPM GPG key before installing signed packages. This verifies that the
	packages belong to a key which has been imported. Otherwise, the **yum** command
	fails due to a missing key. The **--nogpgcheck** option can be used to ignore missing
	GPG keys, but this could cause forged or insecure packages to be installed on the
	system, potentially compromising its security.

### MANAGING PACKAGE MODULE STREAMS ###

#### INTRODUCTION TO APPLICATION STREAM ####

Red Hat Enterprise Linux 8.0 introduces the concept of Application Streams. Multiple versions of
user space components shipped with the distribution are now delivered at the same time. They
may be updated more frequently than the core operating system packages. This provides you with
greater flexibility to customize Red Hat Enterprise Linux without impacting the underlying stability
of the platform or specific deployments.

Traditionally, managing alternate versions of an application's software package and its related
packages meant maintaining different repositories for each different version. For developers who
wanted the latest version of an application and administrators who wanted the most stable version
of the application, this created a situation that was tedious to manage. This process is simplified
in Red Hat Enterprise Linux 8 using a new technology called *Modularity*. Modularity allows a single
repository to host multiple versions of an application's package and its dependencies.

Red Hat Enterprise Linux 8 content is distributed through two main software repositories: BaseOS
and Application Stream (AppStream).

**BaseOS**

The BaseOS repository provides the core operating system content for Red Hat Enterprise Linux
as RPM packages. BaseOS components have a life cycle identical to that of content in previous
Red Hat Enterprise Linux releases.

**Application Stream**

The Application Stream repository provides content with varying life cycles as both modules and
traditional packages. Application Stream contains necessary parts of the system, as well as a wide
range of applications previously available as a part of Red Hat Software Collections and other
products and programs.

!!! danger "IMPORTANT"

	Both BaseOS and AppStream are a necessary part of a Red Hat Enterprise Linux 8
	system.

The Application Stream repository contains two types of content: Modules and traditional RPM
packages. A module describes a set of RPM packages that belong together. Modules can contain
several streams to make multiple versions of applications available for installation. Enabling a
module stream gives the system access to the RPM packages within that module stream.

#### MODULES ####

A module is a set of RPM packages that are a consistent set that belong together. Typically, this
is organized around a specific version of a software application or programming language. A
typical module can contain packages with an application, packages with the application’s specific
dependency libraries, packages with documentation for the application, and packages with helper
utilities.

**Module Streams**

Each module can have one or more module streams, which hold different versions of the content.
Each of the streams receives updates independently. Think of the module stream as a virtual
repository in the Application Stream physical repository.

For each module, only one of its streams can be enabled and provide its packages.

**Module Profiles**

Each module can have one or more profiles. A profile is a list of certain packages to be installed
together for a particular use-case such as for a server, client, development, minimal install, or
other.

Installing a particular module profile simply installs a particular set of packages from the module
stream. You can subsequently install or uninstall packages normally. If you do not specify a profile,
the module will install its default profile.

#### MANAGING MODULES USING YUM ####

Yum version 4, new in Red Hat Enterprise Linux 8, adds support for the new modular features of
Application Stream.

For handling the modular content, the **yum module** command has been added. Otherwise, **yum**
works with modules much like does with regular packages.

**Listing Modules**

To display a list of available modules, use yum module list:

``` bash
[user@host ~]$ yum module list
   Red Hat Enterprise Linux 8.0 AppStream - AppStream HTB (RPMs)
   Name                    Stream        Profiles   Summary
   389-ds                  1.4           default    389 Directory Server (base)
   ant                     1.10 [d]      common [d] Java build tool
   container-tools         1.0 [d]       common [d] Common tools and dependencies
```

!!! info "NOTE"

	Use the Hint at the end of the output to help determine which streams and profiles
	are enabled, disabled, installed, as well as which ones are the defaults.
	
To list the module streams for a specific module and retrieve their status:

``` bash
[user@host ~]$ yum module list perl
Red Hat Enterprise Linux 8 for x86_64 - AppStream HTB (RPMs)
Name  Stream       Profiles             Summary
perl  5.24         common [d], minimal  Practical Extraction and Report Language
perl  5.26 [d]     common [d], minimal  Practical Extraction and Report Language
```

To list the packages installed by the default module stream:

``` bash
[user@host ~]$ yum module info perl
    Name             : perl
    Stream           : 5.26 [d][a]
    Version          : 820181219174508
    Context          : 9edba152
    Profiles         : common, minimal
    Default profiles : default
    Repo             : rhel-8-for-x86_64-appstream-htb-rpms
    Summary          : Practical Extraction and Report Language
    ...output omitted...
    Artifacts   : perl-4:5.24.4-401.module+el8+2464+d274aed1.x86_64
                : perl-Algorithm-Diff-0:1.1903-9.module+el8+2464+d274aed1.noarch
                : perl-Archive-Tar-0:2.30-1.module+el8+2464+d274aed1.noarch
    ...output omitted...
```
    
!!! info "NOTE"

	Without specifying a module stream, **yum module info** shows details of the
	default module stream. Use the *module-name:stream* format to view a specific
	module stream. Add the **--profile** option to display information about packages
	installed by each of the module's profiles. For example:
	
	``` bash	
	[user@host ~]$ yum module info --profile perl:5.24
	```

**Enabling Module Streams and Installing Modules**

Module streams must be enabled in order to install their module. To simplify this process, when
a module is installed it enables its module stream if necessary. Module streams can be enabled
manually using **yum module enable** and providing the name of the module stream.

!!! danger "IMPORTANT"

	Only one module stream may be enabled for a given module. Enabling an additional
	module stream will disable the original module stream.

Install a module using the default stream and profiles:

``` bash
[user@host ~]$ sudo yum module install -y perl
Dependencies resolved.
================================================================================
 Package         Arch   Version      Repository                            Size
================================================================================
Installing group/module packages:
 perl            x86_64 4:5.26.3-416.el8
                                     rhel-8-for-x86_64-appstream-htb-rpms  72 k
Installing dependencies:
...output omitted...
Running transaction
  Preparing        :                                                        1/1
  Installing       : perl-Exporter-5.72-396.el8.noarch                    1/155
  Installing       : perl-Carp-1.42-396.el8.noarch                        2/155
...output omitted...
Installed:
  perl-4:5.26.3-416.el8.x86_64
  perl-Encode-Locale-1.05-9.el8.noarch
...output omitted...
Complete!
```

!!! info "NOTE"

	The same results could have been accomplished by running **yum install @perl**.
	The @ notation informs **yum** that the argument is a module name instead of a
	package name.

To verify the status of the module stream and the installed profile:

``` bash
[user@host ~]$ yum module list perl
Red Hat Enterprise Linux 8.0 AppStream - AppStream HTB (RPMs)
Name  Stream       Profiles             Summary
perl  5.24         common, minimal      Practical Extraction and Report Language
perl  5.26 [d][e]  common [i], minimal  Practical Extraction and Report Language
Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```
  
**Removing Modules and Disabling Module Streams**

Removing a module removes all of the packages installed by profiles of the currently enabled
module stream, and any further packages and modules that depend on these. Packages installed
from this module stream not listed in any of its profiles remain installed on the system and can be
removed manually.

!!! danger "WARNING"

	Removing modules and switching module streams can be a bit tricky. Switching
	the stream enabled for a module is equivalent to resetting the current stream and
	enabling the new stream. It does not automatically change any installed packages.
	You have to do that manually.
	
	Directly installing a module stream that is different than the one that is currently
	installed is not recommended, because upgrade scripts might run during the
	installation that would break things with the original module stream. That could lead
	to data loss or other configuration issues.
	
	Proceed with caution.

To remove an installed module:

``` bash
[user@host ~]$ sudo yum module remove -y perl
  Dependencies resolved.
  ================================================================================
   Package                        ArchVersion            Repository               
             Size
  ================================================================================
  Removing:
   perl                           x86_644:5.26.3-416.el8   @rhel-8-for-x86_64-
appstream-htb-rpms 0
  Removing unused dependencies:
  ...output omitted...
  Running transaction
  Preparing        :                                                        1/1
  Erasing          : perl-4:5.26.3-416.el8.x86_64                         1/155
  Erasing          : perl-CPAN-2.18-397.el8.noarch                        2/155
  ...output omitted...
  Removed:
  perl-4:5.26.3-416.el8.x86_64
  dwz-0.12-9.el8.x86_64
  ...output omitted...
  Complete!
```
  
After the module is removed, the module stream is still enabled. To verify the module stream is still
enabled:

``` bash
[user@host ~]$ yum module list perl
Red Hat Enterprise Linux 8.0 AppStream - AppStream HTB (RPMs)
Name  Stream       Profiles             Summary
perl  5.24         common [d], minimal  Practical Extraction and Report Language
perl  5.26 [d][e]   common [d], minimal  Practical Extraction and Report Language
Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```
  
To disable the module stream:

``` bash
[user@host ~]$ sudo yum module disable perl
  ...output omitted...
Dependencies resolved.
=================================================================================
 Package           Arch             Version              Repository         Size
=================================================================================
Disabling module streams:
 perl                               5.26
Is this ok [y/N]: y
Complete!
```
  
**Switching Module Streams**
Switching module streams generally requires upgrading or downgrading the content to a different
version.

To ensure a clean switch, you should remove the modules provided by the module stream first.
That will remove all the packages installed by the profiles of the module, and any modules and
packages that those packages have dependencies on.

To enable a different module stream and install the module:

``` bash
[user@host ~]$ sudo yum module install perl:5.24
```

The new module stream will be enabled and the current stream disabled. It may be necessary to
update or downgrade packages from the previous module stream that are not listed in the new
profile. Use the **yum distro-sync** to perform this task if required. There may also be packages
that remain installed from the previous module stream. Remove those using **yum remove**.

### SUMMARY ###

In this chapter, you learned:

- Red Hat Subscription Management provides tools to entitle machines to product subscriptions,
get updates to software packages, and track information about support contracts and
subscriptions used by the systems.

- Software is provided as RPM packages, which make it easy to install, upgrade, and uninstall
software from the system.

- The **rpm** command can be used to query a local database to provide information about the
contents of installed packages and install downloaded package files.

- **yum** is a powerful command-line tool that can be used to install, update, remove, and query
software packages.

- Red Hat Enterprise Linux 8 uses Application Streams to provide a single repository to host
multiple versions of an application's packages and its dependencies.
