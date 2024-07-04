# Learn selinux

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
