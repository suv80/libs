I installed CentOS 7 with minimal configuration (os + dev tools). I am trying to open 80 port for httpdservice, but something wrong with my iptables service ... what's wrong with it? What am I doing wrong?

```
$ifconfig/sbin/service iptables save
bash: ifconfig/sbin/service: No such file or directory
```

```
$/sbin/service iptables save
```

The service command supports only basic LSB actions (start, stop, restart, try-restart, reload, force-reload, status). For other actions, please try to use systemctl.

```
$sudo service iptables status
```

Redirecting to /bin/systemctl status  iptables.service

iptables.service

Loaded: not-found (Reason: No such file or directory)

Active: inactive (dead)

```
$/sbin/service iptables save
```

The service command supports only basic LSB actions (start, stop, restart, try-restart, reload, force-reload, status). For other actions, please try to use systemctl.

```
$sudo service iptables start
```

Redirecting to /bin/systemctl start  iptables.service

Failed to issue method call: Unit iptables.service failed to load: No such file or directory.

With RHEL 7 / CentOS 7, firewalld was introduced to manage iptables. IMHO, firewalld is more suited for workstations than for server environments.

It is possible to go back to a more classic iptables setup. First, stop and mask the firewalld service:

```
$systemctl stop firewalld
$systemctl mask firewalld

```
  
Then, install the iptables-services package:

```
$yum install iptables-services
```
  
Enable the service at boot-time:

```
$systemctl enable iptables
```

Managing the service

```
$systemctl [stop|start|restart] iptables
```
  
Systemctl doesn't seem to manage the save action like you were able to do in the past with service: 

```
$/usr/libexec/iptables/iptables.init save  
```

This fixed it: 

```
$yum install iptables-services
$systemctl mask firewalld
$systemctl enable iptables
$systemctl enable ip6tables
$systemctl stop firewalld
$systemctl start iptables
$systemctl start ip6tables
```
