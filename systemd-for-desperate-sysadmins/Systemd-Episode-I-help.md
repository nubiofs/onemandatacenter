# Help! Systemd ate my Linux!

## Introduction

... or systemd for desperate sysadmins.

Hi, I'm a Linux sysadmin. I got frozen on ice Capitain America-style and recently got thawed and hired by an huge Linux organization with a massive amount of servers. They gave me root and a mission: 

"You are... the One Man Datacenter. Operate'em all!"

When I logged on the first system, my surprise.

Something called **Systemd** ate my Linux!

Allright, let's handle it.

## Episode I: All you need to know in 5 minutes (or less)

Okay, maybe not all I need. 

### 'cmd operation daemon' instead of 'cmd daemon operation'

Oh god why.

* Is my service running?

```
# # Instead of: 
# service httpd status

# # Now you type:
# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:httpd(8)
           man:apachectl(8)
```
The weird output above shows the service is *not running*: the Active field is *inactive*. You can also notice it's *disabled*, so it won't start on boot.

* Start my service imediately!

```
# # Instead of: 
# service httpd start

# # Now you type:
# systemctl start httpd
#
```
Yes, systemd does not like you and won't tell you if it worked. It'll tell if it failed, though, so rest assured, everything is fine. You can check it with *systemctl status* (operation first, service later) if you want!

* But what if it reboots?

```
# # Instead of: 
# chkconfig httpd on

# # Now you type:
# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /etc/systemd/system/httpd.service.
``` 
Never liked chkconfig anyways.

Check with status to see the difference:
```
# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sex 2017-06-02 23:36:35 PDT; 6min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 2807 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─2807 /usr/sbin/httpd -DFOREGROUND
           ├─2817 /usr/sbin/httpd -DFOREGROUND
           ├─2818 /usr/sbin/httpd -DFOREGROUND
           ├─2819 /usr/sbin/httpd -DFOREGROUND
           ├─2820 /usr/sbin/httpd -DFOREGROUND
           └─2821 /usr/sbin/httpd -DFOREGROUND
```
Notice the subtle *enabled* in the *Loaded* line.

* Boring... Stop/restart service

```
# # Instead of:
# service httpd stop / service httpd restart

# # It's getting boring. 
# systemctl stop httpd
# systemctl restart httpd
#
```

Well, that's all you need to know about systemd (not really but yeah).

Sysadmin all the things!

## Episode II: Them logs

On Episode I, you learned that systemd ate most of your basic service managing tools. Even though it didn't eat *ps* or *ls* (yet), you'll learn that it's a lot worse than it seems, although it's the better kind of worse. 

Loggin on Linux usually relies on the /dev/
