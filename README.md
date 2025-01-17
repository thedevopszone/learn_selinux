# Learn selinux

## Links

- https://www.youtube.com/watch?v=tXNr3gOgrn8
- https://www.youtube.com/watch?v=O8YQVpneH5w
- https://www.youtube.com/watch?v=ZW80ASpXYiQ


## The 3 modes

- Enforcing: Look at /var/log/audit/audit.log
- Permissive: Look at: /var/log/audit/audit.log
- Disabled

How to get the actual mode:
```
getenforce

setenforce 1
```


This file must be present in root directory /
```
ls -al /.autorelabel
```


## Policies

- Policies is a set of rules that tells Linux how to operate.
- Policies are defined in the file: /etc/sysconfig/selinux

There are 3 types that can be set in: /etc/sysconfig/selinux
```
SELINUXTYPE=targeted  # Most secure, protects all services
SELINUXTYPE=minimum   # Only protects some of the services
SELINUXTYPE=mls
```

## Labels


- A label can also be seen as a tag
- The labels are also known as types
- Every object that can be handled by SeLinux has a tag (files, ports, directories, users, processes etc.)
- This labels are used to create rules that defines which object can communicate with other objects
- Example: Can the webservice communcate with the user home directory


```
ls -l

#And now with types
ls -lZ

... system_u:object_r:user_home_t:s0    # This is the SELinux context

system_u = SELinux user  _u
object_r = SELinux role   _r
user_home_t = SELinux type (label)   _t
```

Apache types as an example
```
ls -lZ /usr/sbin/httpd
system_u:object_r:httpd_exec_t:s0    # httpd_exec_t => httpd executetable type

ls -dZ /etc/httpd
system_u:object_r:httpd_config_t

ls -dZ /var/log/httpd
system_u:object_r:httpd_log_t:s0

ls -dZ /var/www/html
system_u:object_r:httpd_sys_content_t:s0

# Ports
netstat -ntlpZ
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name     Security Context
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      20882/sshd: /usr/sb  system_u:system_r:sshd_t:s0-s0:c0.c1023
tcp6       0      0 :::80                   :::*                    LISTEN      50452/httpd          system_u:system_r:httpd_t:s0
tcp6       0      0 :::22                   :::*                    LISTEN      20882/sshd: /usr/sb  system_u:system_r:sshd_t:s0-s0:c0.c1023

#or
netstat -lZ | grep httpd
unix  2      [ ACC ]     STREAM     LISTENING     82834    50453/httpd          system_u:system_r:httpd_t:s0


# Process
ps axZ | grep httpd
system_u:system_r:httpd_t:s0      50452 ?        Ss     0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0      50453 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0      50454 ?        Sl     0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0      50455 ?        Sl     0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0      50456 ?        Sl     0:00 /usr/sbin/httpd -DFOREGROUND
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 50903 pts/0 S+   0:00 grep --color=auto httpd


```

Important!!!! Diferent types can non communicate wich each other. For example: httpd_config_t => httpd_exec_t. We have to fix that.



## Type Enforcement

```
ls -lZ /etc/passwd
system_u:object_r:passwd_file_t:s0

```

## Change SELinux contexts

```
chcon -t httpd_sys_content_t /var/www/html/index.html

# Reference a known god type (for example the directory the file is in)
chcon --reference /var/www/html /var/www/html/index.html

# Set to SELinux default again
restorecon -vR /var/www/html   # v=verbose R=recursive

```

## Preserve (bewahren) SELinux contexts

When you copy or move files, there SELinux contexts can change. Use --preserve to keep their context
```
cp --preserve=context file1 /tmp/
```



## Permanent changes

Note: 
- Label changes made by chcon will not survive a labeling process.
- To make permanent use: semanage fcontext, folowed by: restorecon
```
semanage fcontext -a [option] [filename] [directory]
restorecon -v [filename] [directory]
```

Example
```
semanage fcontext -a -t httpd_sys_content_t /var/www/html/index.html

restorecon -v httpd_sys_content_t /var/www/html/index.html

# Note: semanage fcontext writes in /etc/selinux/targeted/contexts/files to save permanent
```


## Booleans (Rules)

List all booleans
```
getsebool -a
# or
semanage boolean -l   # much better with description

# Apache
semanage boolean -l | grep httpd

```


Set boolean
```
setsebool -P ftpd_anon_write on    # -P persistant
```

Example 1: The webserver service want to access user home directories
```
semanage boolean -l | grep http | grep home
httpd_enable_home_dirs (off, off) Allow httpd to enable homedirs

setsebool -P httpd_enable_homedirs on
```

Example 2: The webserver should be able to use sendmail
```
semanage boolean -l | grep http | sendmail
httpd_can_sendmail

setsebool -P httpd_can_sendmail on


semanage boolean -l | grep guest
```


## Troubleshooting SELinux

SeLinux logs to: /var/log/audit/audit.log

You can use tools to troubleshoot SELinux
```
setroubleshoot
setroubleshoot-server

dnf install setroubleshoot
dnf install setroubleshoot-server
ausearch and audit2allow

# Important, you have to restart the auditd service
service auditd restart

# Usage
sealert -a /var/log/audit/audit.log

```


```
# You can disable selinux
setenforce 0

# If its working, you have a SELinux problem

# Enable SELinux again
setenforce 1

# Now get the error messages
grep sealert /var/log/messages

cat /etc/services

semanage port -a -t http_port_t -p tcp 85

```



## Troubleshooting SELinux Non_complient Apps

```
setenfoce permisive
```

Use sealert to create a problem report
```
sealert -a /var/log/audit/audit.log

Fix by copy and paste what is shown in the report
```

You can also look at journalctl
```
journalctl -xe  # x add explanation e jump to the end
```


```
setenforce enforcing
```



## Confined Users, Booleans and sudo

You can map an Linux user to an SELinux user

List the user mappings
```
semanage login -l

Login Name           SELinux User         MLS/MCS Range        Service
__default__          unconfined_u         s0-s0:c0.c1023       *      # __default__ is for all users
root                 unconfined_u         s0-s0:c0.c1023       *
```

There are some predefined SELinux users
```
sysadm_u
staff_u
user_u
guest_u
xguest_u
```

```
semanage login -m -S targeted -s "user_u" -r s0 __default__

semanage login -l

Now every user per deault is mapped to the SELinux user user_u after login

Login Name           SELinux User         MLS/MCS Range        Service
__default__          user_u         s0-s0:c0.c1023       *      # __default__ is for all users
root                 unconfined_u         s0-s0:c0.c1023       *

For the user: user_u there are rules defined

```


Check what SELinux user my Linux user is mapped to
```
id -Z
```

Map an SeLinux user to an Linux user
```
semanage login -a -s staff_u cloud_user

semanage login -l

# Remove again
semanage login -d cloud_user

semanage login -l
```


## sudo

If a user is not in the SELinux role unconfined_r, only sysadm_r and staff_r roles Users are allowed to use sudo

Add a user to a SELinux role
```
semanage login -a -s "staff_u" tmundt
```

Add an role to an existing SELinux user
```
semanage user -m -R "staff_r" "user_u"  # m modify, r role
```

Add sudo rigths 
```
vi /etc/sudoers

tmundt ALL=(ALL) TYPE=administrator_t
ROLE=administrator_t /bin/sh


restorecon -FR -v /home/tmundt
```


```
setsebool -P user_exec_content off
```


## Change Apache Port to 8888

```
Is the port managed by SELinux rules?
semanage port -l | grep 8888

Change Apache Port to listen to port 8888
Listen 8888

systemctl restart httpd

You get an error and because of the installed SELinux helper packets you get an better message in journalctl.

semanage port -a -t PORT_TYPE -p tcp 8888
But how do we get the PORT_TYPE?

What PORT_TYPE does 80 use?
Because thats ok!

semanage port -l | grep http | grep 80
http_port_t

So:
semanage port -a -t http_port_t -p tcp 8888

sstemctl restart httpd
```



## The Labeling-System

SELinux is a Labeling-System. Every process, every file, every directory and every system object (ports. etc) has and defined label set, named SELinux context.  
SELinux-Policies allows and defines access to labeled processes to labelled objects.  


## The two most important konzepts


1. Das Labeling (Dateien, Ports, Prozesse etc.)
2. Type Enforcement (also die Isolation von Prozessen auf Basis ihrer Labels).




Das korrekte Label-Format für einen Kontext ist user:role:type:level.

Um SELinux temporär für das ganze System abzuschalten, setenforce 0 verwenden. Doch Vorsicht: das System ist von da an ungeschützt. Um SELinux zur Laufzeit auf „enforcing“ zu setzen , setenforce 1 nutzen.

So setzt man SELinux dauerhaft auf den Enforcing Mode (was der Standard-Einstellung in RHEL entspricht). Anschliessend ist ein Reboot fällig:

vi /etc/sysconfig/selinux
```
SELINUX=disabled
SELINUX=permissive
SELINUX=enforcing
```



```
semanage permissive --add httpd_t
```






```
getenforce
# or
sestatus

permissive = logs but allows


ausearch -m AVC,USER_AVC -ts recent

vi /etc/selinux/config

or
setenforce 1 # On next reboot it will switch to the value in the config file

audit2allow -a -w

port erlauben:
sudo semanage port -a -t http_port_t -p tcp 8765




yum install policycoreutils-python-utils
semanage port -l

semanage port -a -t http_port_t -p tcp 2030

semanage port -d -p tcp 2030



```



```
SELINUX
chcon -Rt httpd_sys_content_t /home/tmundt/test/logs

ls -lZ /home/tmundt/test/logs

Allow HTTP servers to connect to other backends
setsebool -P httpd_can_network_connect on

Allow HTTP servers to read files from user directory
setsebool -P httpd_enable_homedirs on

Allow HTTP servers to read files from user directory
chcon -Rt httpd_sys_content_t /file/path

man semanage-fcontext

Add another Port for Apache: Listen 9090
semanage port -l | grep http
semanage port --help
semanage port -a -t http_port_t -p 9090

Get error messages
audit2allow -a -w
# Do not execute the commands shown in the log!!!
Better:
semanage port -a -t http_port_t -p tcp 9090


```



```
semanage fcontext -l
semanage port




/web/index.html
httpd_t => default_t

semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?" # Set
restorecon -R -v /web    # Save

# Booleans

List all booleans
getsebool -a
better
semanage boolean -l

# Show what a boolean defines
sesearch -b ftpd_anon_write -ACT

# Without type_transition
sesearch -b ftpd_anon_write -ACT | grep -v type_transit

setsebool -P ftpd_ano_write on


semanage port -a -t http_port_t -p tcp 16700


grep http /var/log/audit/audit.log | audit2why

```

## Schulung von Y

```
ll /root -Z

root root system_u:object_r:admin_home_t:s0
admin_home_t ist der für uns wichtige Teil
```

```
systemctl
labels

labels steuert was erlaubt ist

ss -tulpen -Z

Regeln anzeigen
sesearch -A -s httpd_t http_sys_content_t


Apache
Listen 12345

systemctl restart httpd
Funktioniert nicht

Failed to start the Apache server
Permission denied
```

```
dnf install setroubleshoot

# Gibt eine Anleitung was zu tun ist
sealert -l "*"

semanage port -a -t PORT_TYPE -p tcp 12345

semanage port -l  | grep http

semanage port -a -t http_port_t -p tcp 12345
semanage port -l  | grep http

ll /var/www/html -Z

ll -Z ./root.html

sealert -l "*"

semanage booleans -l

semanage fcontext -l | grep /var/www/html

touch /.autorelable
fixfiles --help

sealert -a /var/log/audit/audit.log

semanage permissive -a httpd_t







```


