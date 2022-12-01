---
title: "Introduction"
---

### RED HAT SYSTEM ADMINISTRATION I  :fontawesome-brands-redhat:{ .redhat }

Red Hat System Administration I (RH124) is designed for IT professionals
without previous Linux system administration experience. The course is
intended to provide students with Linux administration "survival skills" by
focusing on core administration tasks.

### SYSTEM-WIDE DEFAULT LANGUAGE SETTINGS

The system's default language is set to US English, using the UTF-8 encoding of Unicode as its
character set (**en_US.utf8**), but this can be changed during or after installation.

From the command line, the root user can change the system-wide locale settings with the
**localectl** command. If localectl is run with no arguments, it displays the current system-
wide locale settings.

To set the system-wide default language, run the command localectl set-locale
LANG=locale, where locale is the appropriate value for the LANG environment variable from the
"Language Codes Reference" table in this chapter. The change will take effect for users on their
next login, and is stored in /etc/locale.conf.
 
``` bash
[root@host ~]# localectl set-locale LANG=fr_FR.utf8
```

### WHY SHOULD YOU LEARN ABOUT LINUX?

Linux is a critical technology for IT professionals to understand.

Linux is in widespread use, and if you use the internet at all, you are probably already interacting
with Linux systems in your daily life. Perhaps the most obvious way in which you interact with
Linux systems is through browsing the World Wide Web and using e-commerce sites to buy and sell
products.

However, Linux is in use for much more than that. Linux manages point-of-sale systems and the
world's stock markets, and also powers smart TVs and in-flight entertainment systems. It powers
most of the top 500 supercomputers in the world. Linux provides the foundational technologies
powering the cloud revolution and the tools used to build the next generation of container-based
microservices applications, software-based storage technologies, and big data solutions.

In the modern data center, Linux and Microsoft Windows are the major players, and Linux is a
growing segment in that space. Some of the many reasons to learn Linux include:

- A Windows user needs to interoperate with Linux.

- In application development, Linux hosts the application or its runtime.

- In cloud computing, the cloud instances in the private or public cloud environment use Linux as
the operating system.

- With mobile applications or the Internet of Things (IoT), the chances are high that the operating
system of your device uses Linux.

- If you are looking for new opportunities in IT, Linux skills are in high demand.

### WHAT MAKES LINUX GREAT?

There are many different answers to the question "What makes Linux great?", however, three of
them are:

- Linux is *open source* software.

Being open source does not just mean that you can see how the system works. You can also
experiment with changes and share them freely for others to use. The open source model means
that improvements are easier to make, enabling faster innovation.

- Linux provides easy access to a powerful and scriptable command-line interface (CLI).

Linux was built around the basic design philosophy that users can perform all administration
tasks from the CLI. It enables easier automation, deployment, and provisioning, and simplifies
both local and remote system administration. Unlike other operating systems, these capabilities
have been built in from the beginning, and the assumption has always been to enable these
important capabilities.

- Linux is a modular operating system that allows you to easily replace or remove components.

Components of the system can be upgraded and updated as needed. A Linux system can be a
general-purpose development workstation or an extremely stripped-down software appliance.

### WHAT IS OPEN SOURCE SOFTWARE?

***Open source*** software is software with source code that anyone can use, study, modify, and share.

***Source code*** is the set of human-readable instructions that are used to make a program. It may be
interpreted as a script or compiled into a binary executable which the computer runs directly. Upon
creating source code, it gets copyrighted, and the copyright holder controls the terms under which
the software can be copied, adapted, and distributed. Users can use this software under a software
license.

Some software has source code that only the person, team, or organization that created it can
see, or change, or distribute. This software is sometimes called "proprietary" or "closed source"
software. Typically the license only allows the end user to run the program, and provides no access,
or tightly limited access, to the source.

Open source software is different. When the copyright holder provides software under an open
source license, they grant the user the right to run the program and also to view, modify, compile,
and redistribute the source royalty-free to others.

Open source promotes collaboration, sharing, transparency, and rapid innovation because it
encourages people beyond the original developers to make modifications and improvements to the
software and to share it with others.

Just because the software is open source does not mean it is somehow not able to be used
or provided commercially. Open source is a critical part of many organizations' commercial
operations. Some open source licenses allow code to be reused in closed source products. One can
sell open source code, but the terms of true open source licenses generally allow the customer to
redistribute the source code. Most commonly, vendors such as Red Hat provide commercial help
with deploying, supporting, and extending solutions based on open source products.

Open source has many benefits for the user:

- *Control*: See what the code does and change it to improve it.

- *Training*: Learn from real-world code and develop more useful applications.

- *Security*: Inspect sensitive code, fix with or without the original developers' help.

- *Stability*: Code can survive the loss of the original developer or distributor.

The bottom line is that open source allows the creation of better software with a higher return on
investment by collaboration.

### TYPES OF OPEN SOURCE LICENSES

There is more than one way to provide open source software. The terms of the software license
control how the source can be combined with other code or reused, and hundreds of different open
source licenses exist. However, to be open source, licenses must allow users to freely use, view,
change, compile, and distribute the code.

There are two broad classes of open source license that are particularly important:

- *Copyleft* licenses that are designed to encourage keeping code open source.

- *Permissive* licenses that are designed to maximize code reusability.

Copyleft, or "share-alike" licenses, require that anyone who distributes the source code, with or
without changes, must also pass along the freedom for others to also copy, change, and distribute
the code. The basic advantage of these licenses is that they help to keep existing code, and
improvements to that code, open and add to the amount of open source code available. Common
copyleft licenses include the GNU General Public License (GPL) and the Lesser GNU Public License
(LGPL).

Permissive licenses are intended to maximize the reusability of source code. Users can use the
source for any purpose as long as the copyright and license statements are preserved, including
reusing that code under more restrictive or even proprietary licenses. This makes it very easy for
this code to be reused, but at the risk of encouraging proprietary-only enhancements. Several
commonly used permissive open source licenses include the MIT/X11 license, the Simplified BSD
license, and the Apache Software License 2.0.

### WHO DEVELOPS OPEN SOURCE SOFTWARE?

It is a misconception to think that open source is developed solely by an "army of volunteers" or
even an army of individuals plus Red Hat. Open source development today is overwhelmingly
professional. Many developers are paid by their organizations to work with open source projects to
construct and contribute the enhancements they and their customers need.

Volunteers and the academic community play a significant role and can make vital contributions,
especially in new technology areas. The combination of formal and informal development provides
a highly dynamic and productive environment.

### WHO IS RED HAT?

Red Hat is the world's leading provider of open source software solutions, using a community-
powered approach to reliable and high-performance cloud, Linux, middleware, storage, and
virtualization technologies. Red Hat's mission is to be the catalyst in communities of customers,
contributors, and partners creating better technology the open source way.

Red Hat's role is to help customers connect with the open source community and their partners
to effectively use open source software solutions. Red Hat actively participates in and supports
the open source community and many years of experience have convinced the company of the
importance of open source to the future of the IT industry.

Red Hat is most well-known for their participation in the Linux community and the Red Hat
Enterprise Linux distribution. However, Red Hat is also very active in other open source
communities, including middleware projects centered on the JBoss developer community,
virtualization solutions, cloud technologies such as OpenStack and OpenShift, and the Ceph and
Gluster software-based storage projects, among others.

### WHAT IS A LINUX DISTRIBUTION?

A *Linux distribution* is an installable operating system constructed from a Linux kernel and
supporting user programs and libraries. A complete Linux operating system is not developed by
a single organization, but by a collection of independent open source development communities
working with individual software components. A distribution provides an easy way for users to
install and manage a working Linux system.

In 1991, a young computer science student named Linus Torvalds developed a Unix-like kernel he
named Linux, licensed as open source software under the GPL. The kernel is the core component of
the operating system, which manages hardware, memory, and the scheduling of running programs.
This Linux kernel could then be supplemented with other open source software, such as utilities
and programs from the GNU Project, the graphical interface from MIT's X Window System, and
many other open source components, such as the Sendmail mail server or the Apache HTTP web
server, in order to build a complete open source Unix-like operating system.

However, one of the challenges for Linux users was to assemble all these pieces from many
different sources. Very early in its history, Linux developers began working to provide a distribution
of prebuilt and tested tools that users could download and use to set up their Linux systems
quickly.

Many different Linux distributions exist, with differing goals and criteria for selecting and
supporting the software provided by their distribution. However, distributions generally have many
common characteristics:

- Distributions consist of a Linux kernel and supporting user space programs.

- Distributions can be small and single-purpose or include thousands of open source programs.

- Distributions must provide a means of installing and updating the distribution and its
components.

- The provider of the distribution must support that software, and ideally, be participating directly
in the community developing that software.

Red Hat Enterprise Linux is Red Hat's commercialized Linux distribution.

### SUMMARY

In this chapter, you learned:

- Open source software is software with source code that anyone can freely use, study, modify,
and share.

- A Linux distribution is an installable operating system constructed from a Linux kernel and
supporting user programs and libraries.

- Red Hat participates in supporting and contributing code to open source projects, sponsors and
integrates project software into community-driven distributions, and stabilizes the software to
offer it as supported enterprise-ready products.

- Red Hat Enterprise Linux is Red Hat's open source, enterprise-ready, commercially-supported
Linux distribution.

### Quiz Solutions

pdf - 30
