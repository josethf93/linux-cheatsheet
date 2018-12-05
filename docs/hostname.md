## hostname

A computer **hostname** is a unique name that gets assigned to a computer in a local network in order to uniquely identify that computer in that network.

There are many ways to check the computer hostname. For example:

```bash
$ hostname
$ uname -n
$ cat /proc/sys/kernel/hostname
$ sysctl kernel.hostname
```

Historically, there also have been multiple ways to set a computer’s hostname which also varied across different distributions, but [systemd](systemd.md) has greatly simplified the process and made this process look the same on any system that runs [systemd](systemd.md).

`systemd` provides the **hostnamectl** utility for managing the hostname of the system.

### How to check computer's hostname

If run without arguments, `hostnamectl` will print the current system hostname(s) and some extra info about the system such as OS and kernel names:

```bash
$ hostnamectl
   Static hostname: localhost.localdomain
         Icon name: computer-vm
           Chassis: vm
        Machine ID: f39619d17a194e15aa3138c77b093bbc
           Boot ID: 5f60ffd1ea4d460f8a1282ecac4d6048
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.2.3.el7.x86_64
      Architecture: x86-64
```

I want to specifically emphasize that the `hostnamectl` command will print all the **hostnames types** that the computer has, and it can have several types of hostnames:

```bash
$ hostnamectl
   Static hostname: localhost.localdomain
   Pretty hostname: Local Host
Transient hostname: localhst
         Icon name: computer-vm
           Chassis: vm
        Machine ID: f39619d17a194e15aa3138c77b093bbc
           Boot ID: 5f60ffd1ea4d460f8a1282ecac4d6048
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-862.2.3.el7.x86_64
      Architecture: x86-64
```

Let's briefly explain what each of these 3 different hostnames mean:

* `Static hostname`: This is a traditional hostname which can be set by a user. Static hostname is stored in the file `/etc/hostname`. The word "static" in this case means persistent as this type of hostname survives reboots. The static hostname is the default hostname the kernel references during boot.
* `Transient hostname`:  This is a dynamic hostname received from network configuration (e.g. DHCP). This is a _fallback hostname_, when a static hostname is not set. If a static hostname is set, and is valid (something other than localhost), then the transient hostname is not used. The transient hostname is maintained by the running kernel and does not survive reboots.
* `Pretty hostname`: This is a high-level hostname which can include special characters. It has little restrictions on the characters and length used, while the static and transient hostnames are limited to the usually accepted characters of Internet domain names, and 64 characters at maximum (the latter being a Linux limitation). The pretty hostname is stored in the file `/etc/machine-info`.

**Note:** commands that we mentioned in the [beginning of this post](#hostname) will all show the transient hostname if it's set and won't show the static hostname which is the hostname the system will default to when it boots the next time. That's why it's recommended to use `hostnamectl` for checking the hostname to get full information about the system hostnames:

```bash
$ hostname
localhst
$ sysctl kernel.hostname
kernel.hostname = localhst
$ hostnamectl | head -3
   Static hostname: localhost.localdomain
   Pretty hostname: Local Host
   Transient hostname: localhst
```

### How to change a hostname

To set the system static hostame, use the `hostnamectl set-hostname` command:

```bash
$ sudo hostnamectl set-hostname testhostname
$ hostnamectl | head -3   # check the hostname
   Static hostname: testhostname
         Icon name: computer-vm
           Chassis: vm
```

In addition to running this command, you may want to edit a couple of more files.

The first one is [/etc/hosts](dns-lookup-on-linux.md) which contains a simple mapping of IP addresses to human-readable hostnames and is used by some programs as a source for a hostname resolution. This file normally contains the mapping of the host system's possible hostnames to local host address (127.0.0.1):

```bash
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 testhostname
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6 testhostname
```

If the `cloud-init` package is installed, you may also need to [edit the cloud.cfg file](https://linuxize.com/post/how-to-change-hostname-on-ubuntu-18-04/#3-edit-the-cloud-cfg-file). This package is usually installed by default in the images provided by the cloud providers such as AWS and it is used to handle the initialization of the cloud instances.

P.S. The `transient` and `pretty` hostnames can be set by providing `--transient` and `--pretty` options correspondigly to the `set-hostname` command.

### Resources used to create this document:
* https://www.tecmint.com/set-change-hostname-in-centos-7/
* https://www.freedesktop.org/software/systemd/man/hostnamectl.html
* https://linuxize.com/post/how-to-change-hostname-on-ubuntu-18-04/
* https://linuxconfig.org/how-to-change-hostname-on-ubuntu-18-04-bionic-beaver-linux
* https://www.maketecheasier.com/hostname-in-linux/
