# Learn selinux

## The 3 modes

- Enforcing: Look at /var/log/audit/audit.log
- Permissive: Look at: /var/log/audit/audit.log
- Disabled

How to get the actual mode:
```
getenforce
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
SELINUX=enforcing
```










```
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
