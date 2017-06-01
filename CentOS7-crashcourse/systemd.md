systemctl
active/inactive/other states, like failed

● named-chroot.service                                     loaded failed failed    Berkeley Internet Name Domain (DNS)

systemctl status named-chroot
 named-chroot.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named-chroot.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Qua 2017-05-31 20:45:24 PDT; 1h 1min ago
  Process: 945 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-checkconf -t /var/named/chroot -z /etc/named.conf; else echo "Checking of zone files is disabled"; fi (code=exited, status=1/FAILURE)

Mai 31 20:45:24 gehenna.hell bash[945]: zone cliente.serpro/IN: not loaded due to errors.

systemd: processes in cgroups
$ ps xawf -eo pid,user,cgroup,args

systemd-cgls

systemd units: after

 Note that a dependency of type After= only encodes the suggested ordering, but does not actually cause syslog to be started when abrtd is -- and this is exactly what we want, since abrtd actually works fine even without syslog being around. However, if both are started (and usually they are) then the order in which they are is controlled with this dependency.
 
 systemctl kill -s vs systemctl stop
kill vai direto, stop usa o caminho configurado na unit como execstop e etc.
 
 
 services: 
 
 - stop: para, mas ele pode ser disparado novamente manualmente, via dbus, via socket ou boot
 - disable: desliga a possível ativação do serviço de maneira futura. não para o em execução. aceita execução manual.
 - mask: desliga de vez, e nem manualente pode chamar.
 
  Unit files stored in /etc/systemd/system override those from /lib/systemd/system that carry the same name. The former directory is administrator territory, the latter terroritory of your package manager. By installing your symlink in /etc/systemd/system/ntpd.service you hence make sure that systemd will never read the upstream shipped service file /lib/systemd/system/ntpd.service.
  
  systemd velocidade: 
  systemd-analyze blame
  $ systemd-analyze plot > plot.svg


Things sytemd do:

Checking and mounting of all file systems
Updating and enabling quota on all file systems
Setting the host name
Configuring the loopback network device
Loading the SELinux policy and relabelling /run and /dev as necessary on boot
Registering additional binary formats in the kernel, such as Java, Mono and WINE binaries
Setting the system locale
Setting up the console font and keyboard map
Creating, removing and cleaning up of temporary and volatile files and directories
Applying mount options from /etc/fstab to pre-mounted API VFS
Applying sysctl kernel settings
Collecting and replaying readahead information
Updating utmp boot and shutdown records
Loading and saving the random seed
Statically loading specific kernel modules
Setting up encrypted hard disks and partitions
Spawning automatic gettys on serial kernel consoles
Maintenance of Plymouth
Machine ID maintenance
Setting of the UTC distance for the system clock

/etc/hostname: the host name for the system. One of the most basic and trivial system settings. Nonetheless previously all distributions used different files for this. Fedora used /etc/sysconfig/network, OpenSUSE /etc/HOSTNAME. We chose to standardize on the Debian configuration file /etc/hostname.
/etc/vconsole.conf: configuration of the default keyboard mapping and console font.
/etc/locale.conf: configuration of the system-wide locale.
/etc/modules-load.d/*.conf: a drop-in directory for kernel modules to statically load at boot (for the very few that still need this).
/etc/sysctl.d/*.conf: a drop-in directory for kernel sysctl parameters, extending what you can already do with /etc/sysctl.conf.
/etc/tmpfiles.d/*.conf: a drop-in directory for configuration of runtime files that need to be removed/created/cleaned up at boot and during uptime.
/etc/binfmt.d/*.conf: a drop-in directory for registration of additional binary formats for systems like Java, Mono and WINE.
/etc/os-release: a standardization of the various distribution ID files like /etc/fedora-release and similar. Really every distribution introduced their own file here; writing a simple tool that just prints out the name of the local distribution usually means including a database of release files to check. The LSB tried to standardize something like this with the lsb_release tool, but quite frankly the idea of employing a shell script in this is not the best choice the LSB folks ever made. To rectify this we just decided to generalize this, so that everybody can use the same file here.
/etc/machine-id: a machine ID file, superseding D-Bus' machine ID file. This file is guaranteed to be existing and valid on a systemd system, covering also stateless boots. By moving this out of the D-Bus logic it is hopefully interesting for a lot of additional uses as a unique and stable machine identifier.
/etc/machine-info: a new information file encoding meta data about a host, like a pretty host name and an icon name, replacing stuff like /etc/favicon.png and suchlike. This is maintained by systemd-hostnamed.

On demand initiation - replacement for xinetd:

Socket activation for parallelization, simplicity, robustness: sockets are bound during early boot and a singleton service instance to serve all client requests is immediately started at boot. This is useful for all services that are very likely used frequently and continously, and hence starting them early and in parallel with the rest of the system is advisable. Examples: D-Bus, Syslog.
On-demand socket activation for singleton services: sockets are bound during early boot and a singleton service instance is executed on incoming traffic. This is useful for services that are seldom used, where it is advisable to save the resources and time at boot and delay activation until they are actually needed. Example: CUPS.
On-demand socket activation for per-connection service instances: sockets are bound during early boot and for each incoming connection a new service instance is instantiated and the connection socket (and not the listening one) is passed to it. This is useful for services that are seldom used, and where performance is not critical, i.e. where the cost of spawning a new service process for each incoming connection is limited. Example: SSH.
  
  systemd security: auto chroot
  RootDirectoryStartOnly=yes

systemd as cron replacement:
Example

No changes to service unit files are needed to schedule them with a timer. The following example schedules foo.service to be run with a corresponding timer called foo.timer.
Monotonic timer
A timer which will start 15 minutes after boot and again every week while the system is running.
/etc/systemd/system/foo.timer
[Unit]
Description=Run foo weekly and on boot

[Timer]
OnBootSec=15min
OnUnitActiveSec=1w 

[Install]
WantedBy=timers.target
Realtime timer
A timer which starts once a week (at 12:00am on Monday). It starts once immediately if it missed the last start time (option Persistent=true), for example due to the system being powered off:
/etc/systemd/system/foo.timer
[Unit]
Description=Run foo weekly

[Timer]
OnCalendar=weekly
Persistent=true     
 
[Install]
WantedBy=timers.target
The format controlling OnCalendar events uses the following format when more specific dates and times are required: DayOfWeek Year-Month-Day Hour:Minute:Second. An asterisk may be used to specify any value and commas may be used to list possible values. Two values separated by .. may be used to indicate a contiguous range. In this example the service is run the first four days of each month at 12:00 PM, but only if that day is also on a Monday or a Tuesday. More information is available in systemd.time(7).
OnCalendar=Mon,Tue *-*-01..04 12:00:00
Tip: Special event expressions like daily and weekly refer to specific start times and thus any timers sharing such calendar events will start simultaneously. Timers sharing start events can cause poor system performance if the timers' services compete for system resources. The  RandomizedDelaySec option avoids this problem by randomly staggering the start time of each timer. See systemd.timer(5).
Transient .timer units

One can use systemd-run to create transient .timer units. That is, one can set a command to run at a specified time without having a service file. For example the following command touches a file after 30 seconds:
# systemd-run --on-active=30 /bin/touch /tmp/foo
One can also specify a pre-existing service file that does not have a timer file. For example, the following starts the systemd unit named someunit.service after 12.5 hours have elapsed:
# systemd-run --on-active="12h 30m" --unit someunit.service
See systemd-run(1) for more information and examples.
As a cron replacement

Although cron is arguably the most well-known job scheduler, systemd timers can be an alternative.
Benefits
The main benefits of using timers come from each job having its own systemd service. Some of these benefits are:
Jobs can be easily started independently of their timers. This simplifies debugging.
Each job can be configured to run in a specific environment (see systemd.exec(5)).
Jobs can be attached to cgroups.
Jobs can be set up to depend on other systemd units.
Jobs are logged in the systemd journal for easy debugging.
Caveats
Some things that are easy to do with cron are difficult to do with timer units alone.
Complexity: to set up a timed job with systemd you create two files and run a couple systemctl commands. Compare that to adding a single line to a crontab.
Emails: there is no built-in equivalent to cron's MAILTO for sending emails on job failure. See the next section for an example of setting up a similar setup using OnFailure=.
MAILTO
You can set up systemd to send an e-mail when a unit fails. Cron sends mail to MAILTO the job outputs to stdout or stderr, but many jobs are setup to only output on error. First you need two files: an executable for sending the mail and a .service for starting the executable. For this example, the executable is just a shell script using sendmail:
/usr/local/bin/systemd-email
#!/bin/bash

/usr/bin/sendmail -t <<ERRMAIL
To: $1
From: systemd <root@$HOSTNAME>
Subject: $2
Content-Transfer-Encoding: 8bit
Content-Type: text/plain; charset=UTF-8

$(systemctl status --full "$2")
ERRMAIL
Whatever executable you use, it should probably take at least two arguments as this shell script does: the address to send to and the unit file to get the status of. The .service we create will pass these arguments:
/etc/systemd/system/status-email-user@.service
[Unit]
Description=status email for %i to user

[Service]
Type=oneshot
ExecStart=/usr/local/bin/systemd-email address %i
User=nobody
Group=systemd-journal
Where user is the user being emailed and address is that user's email address. Although the recipient is hard-coded, the unit file to report on is passed as an instance parameter, so this one service can send email for many other units. At this point you can start status-email-user@dbus.service to verify that you can receive the emails.
Then simply edit the service you want emails for and add OnFailure=status-email-user@%n.service to the [Unit] section. %n passes the unit's name to the template.
Note:
If you set up SSMTP security according to SSMTP#Security the user nobody will not have access to /etc/ssmtp/ssmtp.conf, and the systemctl start status-email-user@dbus.service command will fail. One solution is to use root as the User in the status-email-user@.service unit.
If you try to use mail -s somelogs address in your email script, mail will fork and systemd will kill the mail process when it sees your script exit. Make the mail non-forking by doing mail -Ssendwait -s somelogs address.
Using a crontab
Several of the caveats can be worked around by installing a package that parses a traditional crontab to configure the timers. systemd-cron-nextAUR and systemd-cronAUR are two such packages. These can provide the missing MAILTO feature.
If you like crontabs just because they provide a unified view of all scheduled jobs, systemctl can provide this. See #Management.

advanced systemd:
  systemd instanciated services and templates
  systemd-nspawn: containers com systemd.
