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


advanced systemd:
  systemd instanciated services and templates
  systemd-nspawn: containers com systemd.
