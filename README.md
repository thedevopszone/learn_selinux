# Learn selinux

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
