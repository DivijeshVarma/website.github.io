---
title: "Analyzing Servers And Getting Support"
---

### ANALYZING AND MANAGING REMOTE SERVERS ###

#### DESCRIBING WEB CONSOLE ####

Web Console is a web-based management interface for Red Hat Enterprise Linux 8 designed for
managing and monitoring your servers. It is based on the open source Cockpit service.

You can use Web Console to monitor system logs and view graphs of system performance.
Additionally, you can use your web browser to change settings using graphical tools in the Web
Console interface, including a fully-functional interactive terminal session.

#### ENABLING WEB CONSOLE ####

Red Hat Enterprise Linux 8 installs Web Console by default in all installation variants except a
minimal installation.

Enable and start the cockpit.socket service, which runs a web server. This step is necessary if
you need to connect to the system through the web interface.

```
[user@host ~]$ sudo systemctl enable --now cockpit.socket
Created symlink /etc/systemd/system/sockets.target.wants/cockpit.socket -> /usr/
lib/systemd/system/cockpit.socket.
```

If you are using a custom firewall profile, you need to add the cockpit service to firewalld to
open port 9090 in the firewall:

```
[user@host ~]$ sudo firewall-cmd --add-service=cockpit --permanent
success
[user@host ~]$ sudo firewall-cmd --reload
success
```

#### LOGGING IN TO WEB CONSOLE ####

Web Console provides its own web server. Launch Firefox to log in to Web Console. You can log in
with the user name and password of any local account on the system, including the root user.

Open *https://servername:9090* in your web browser, where servername is the hostname
or IP address of your server. The connection will be protected by a TLS session. Out of the box,
the system will be installed with a self-signed TLS certificate, and when you initially connect your
web browser will probably display a security warning. The cockpit-ws(8) man page provides
instructions on how to replace the TLS certificate with one that is properly signed.

Enter your user name and password at the login screen.

Refer Pdf : 578

### GETTING HELP FROM RED HAT CUSTOMER PORTAL ###

Refer Pdf : 596

### DETECTING AND RESOLVING ISSUES WITH RED HAT INSIGHTS ###

Refer Pdf : 607

### SUMMARY ###

In this chapter, you learned:

- Web Console is a web-based management interface to your server based on the open source
Cockpit service.

- Web Console provides graphs of system performance, graphical tools to manage system
configuration and inspect logs, and an interactive terminal interfaces.

- Red Hat Customer Portal provides you with access to documentation, downloads, optimization
tools, support case management, and subscription and entitlement management for your
Red Hat products.

- **redhat-support-tool** is a command-line tool to query Knowledgebase and work with
support cases from the server's command line.

- Red Hat Insights is a SaaS-based predictive analytics tool to help you identify and remediate
threats to your systems' security, performance, availability, and stability.

### LAB

pdf - 616