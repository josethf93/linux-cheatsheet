## Log rotation

One of the most important things that you have to aware about when dealing with logs as a system administrator is that logs require **rotation**.

Logs are written constantly as they reflect every event that happens on the system. To prevent logs from growing too big and taking up all the disk space (which can lead to services' failures), you need to develop a proper **rotation policy**, i.e. a plan on how to archive/delete old logs that become less valuable with the time.

Below we're going to cover some of the popular tools used for log rotation.

### Logrotate

**Logrotate** is one of the most popular tools for rotating log files. In particular, it's often used to rotate log files produced by a [syslog daemon](logging.md) like [rsyslog](logging.md).

`logrotate’s` behavior is determined by options set in its configuration files. The main configuration file that logrotate reads by default is **/etc/logrotate.conf**. Here is how a default configuration file looks like on my test Centos 7 box:

```bash
$ cat /etc/logrotate.conf

# Global options
weekly
rotate 4
create
dateext
#compress

include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    # Local options defined for specific log files (override global options above)
    monthly
    create 0664 root utmp
    minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}
```

The first few lines in this file set `global options` which apply by default to all rotated files unless they are overriden by `local options`.

Next follows a special directive which tells logrotate to also read configuration files under **/etc/logrotate.d** directory. This is a directory where individual packages that you install will place logrotate configuration for their logs. On a default OS installation, you will already have files for basic system applications like syslog and a package manager (e.g. yum, apt):

```bash
$ ls -1 /etc/logrotate.d/
bootlog
chrony
syslog
wpa_supplicant
yum
```

Each of these application-specific files define a list of log files to rotate and how to rotate them:

```bash
$ cat /etc/logrotate.d/syslog
/var/log/cron
/var/log/maillog
/var/log/messages
/var/log/secure
/var/log/spooler
{
    missingok
    sharedscripts
    postrotate
	    /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
    endscript
}
```

Some log files may not be part of a some specific service or package and their logrotate configuration may be defined in the main configuration file, i.e. `/etc/logrotate.conf`. Notice how in the sample `/etc/logrotate.conf` above, configuration options for `/var/log/wtmp` and `/var/log/btmp` are defined directly in the main config file.

#### Examples of logrotate configuration for log files

Logrotate configuration has the following syntax for defining rotation of a log file:

```
<full-path-to-a-file> { <configuration options> }
```

For example, the following configuration defines rotation of a log file created by yum (P.S. configuration options are explained in [the section below](#configuration-options), right now we're only looking at the syntax):

```bash
$ cat /etc/logrotate.d/yum

/var/log/yum.log {
    missingok
    notifempty
    size 30k
    yearly
    create 0600 root root
}
```

If you want to define the same log rotation configuration for multiple files, you simply separate log file paths by a space:

```bash
$ cat /etc/logrotate.d/syslog
/var/log/cron
/var/log/maillog
/var/log/messages
/var/log/secure
/var/log/spooler
{
    missingok
    sharedscripts
    postrotate
	    /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
    endscript
}
```

You can also use [wildcards](globbing.md) inside the path names. For example, to specify all the files inside the `/var/foo` directory that end in `.log`, and the file `/var/bar/log.txt`, you could use a configuration block like this:

```
/var/foo/*.log /var/bar/log.txt {
    rotate 14
    daily
    compress
}
```

**NOTE:** use wildcards with caution. If you specify `*`, logrotate will rotate all files, including previously rotated ones. A way around this is to use the [olddir option](https://linux.die.net/man/8/logrotate) or a more exact wildcard (such as `*.log`).

#### Configuration options

Below we'll look at some of the common options used in logrotate configuration. For a full list of options, see the [logrotate man page](https://linux.die.net/man/8/logrotate).

#### rotate count

The **rotate** directive determines how many archived logs are kept on the system before logrotate starts deleting the older ones. For example:

```
rotate 4
```

This directive tells logrotate to keep four archived logs at a time. If four archived logs exist when the log is rotated again, the oldest one is deleted to make room for the new archive:

```bash
$ ls -1 /var/log/messages*
/var/log/messages
/var/log/messages-20181217
/var/log/messages-20181223
/var/log/messages-20181230
/var/log/messages-20190106
```

#### rotation frequency

In the configuration file, you can specify a _rotation time interval_ that tells logrotate how often to rotate a particular log file. The possible options are:

* `daily`: Log files are rotated every day.
* `weekly`: Log files are rotated if the current weekday is less than the weekday of the last rotation (e.g., Monday is less than Friday) or if more than a week has passed since the last rotation. This is normally the same as rotating logs on the first day of the week, but it works better if logrotate is not run every night.
* `monthly`: Log files are rotated the first time logrotate is run in a month (this is normally on the first day of the month).
* `yearly`: Log files are rotated if the current year is not the same as the last rotation.

Another option is to rotate log files once they reach a specific _size_. The `size` directive makes logrotate to check the file size and, if it has grown bigger than a given size, to rotate it.

Note that the size directive takes priority over a rotation interval if both are set. For example, the following configuration will make logrotate `/var/log/yum.log` file when it grows bigger than 30 kilobytes even though it has already been previously rotated this same year:

```
$ cat /etc/logrotate.d/yum

/var/log/yum.log {
    missingok
    notifempty
    size 30k
    yearly
    create 0600 root root
}
```

#### compress old versions of logs

If you want old versions of log files to be compressed, you can include the `compress` directive in logrotate configuration file. Compression is normally a good idea, because log files are usually text files and text compresses well. If, however, you have some archived logs that you don’t want to compress, but you still want compression to be on by default, you can include the `nocompress` directive in an application-specific configuration.

By default, logrotate compresses files using the `gzip` command. You can replace this with another compression tool such as `bzip2` or `xz` as an argument to the `compresscmd` directive. For example:

```
compresscmd xz
```

Another important option with regards to compression is the `delaycompress` option. It makes logrotate to postpone the compression of the previous log file to the next rotation cycle. This only has effect when used in combination with `compress`. It can be used when some program cannot be told to close its logfile and thus might continue writing to the previous log file for some time.

An example of using `delaycompress` would be when logrotate is told to restart apache with the "reload" directive. Because old apache processes do not end until their connections are closed, they could potentially try to log to the old file for some time after the restart. Delaying the compression ensures that you won’t lose those extra log entries when the logs are rotated.

```bash
$ cat /etc/logrotate.d/httpd

/var/log/httpd/*log {
    missingok
    notifempty
    sharedscripts
    delaycompress
    postrotate
        /bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
    endscript
}
```

#### Postrotate command

The shell commands you specify between `postrotate` and `endscript` directives are executed (using `/bin/sh`) after the log file is rotated. These directives may only appear inside a log file definition block enclosed in curly brackets (`{}`).

You typically would use it to reload/restart the application after the log rotation so that the app can switch to a new log file. Here is an example of a sample apache configuration file:

```
$ cat /etc/logrotate.d/httpd

/var/log/httpd/*log {
    missingok
    notifempty
    sharedscripts
    delaycompress
    postrotate
        /bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
    endscript
}
```

**NOTE:** normally logrotate runs the postrotate script for each log file it rotates. That means that a single script may be run multiple times for log file entries which match multiple files. For example, the apache server configuration block uses the [star wildcard](globbing.md) to refer to both the access log and the error log:

```bash
$ ls -1 /var/log/httpd/*log
/var/log/httpd/access_log
/var/log/httpd/error_log
```

This basically means that if both files are rotated, logrotate will run the postrotate script twice (once for each rotated file), thus apache server will be reloaded twice.

To keep logrotate from running postrotate commands for every rotated log, add the `sharedscripts` directive to the configuration block. This will make logrotate to run postrotate commands only once for all the files included in that configuration block (also, if none of the logs is rotated, the postrotate script won't run).

#### Create empty log file immediately after rotation

If your daemon process requires that its log file exists to function properly, logrotate may interfere when it rotates logs. It is possible to have logrotate create new, empty log file after rotation with the `create` directive.

If you specify `create` option in your logrotate configuration files, logrotate will create an empty log file (with the same name as the log file just rotated) immediately after rotation (before the postrotate script is run). The new log file will be created with the same attributes ([mode, owner, group](file-permissions.md)) as the original log file.
Alternatively, you can specify the file mode, user and group owner of the file. For example:

```bash
$ cat /etc/logrotate.conf
...
# create new (empty) log files after rotating old ones
create

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}
...
```

#### How to run logrotate

Normally, logrotate is run as a _daily [cron job](cron.md)_. For example, on my standard Centos 7 installation I have the following cron job by default:

```bash
$ cat /etc/cron.daily/logrotate
#!/bin/sh

/usr/sbin/logrotate -s /var/lib/logrotate/logrotate.status /etc/logrotate.conf
EXITVALUE=1
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```

Note how the logrotate command is invoked:

```
/usr/sbin/logrotate -s /var/lib/logrotate/logrotate.status /etc/logrotate.conf
```

When you call a logrotate command, you need to pass one or more configuration files. If called without arguments, it will simply show the usage instructions along with the version information:

```bash
$ logrotate
logrotate 3.8.6 - Copyright (C) 1995-2001 Red Hat, Inc.
This may be freely redistributed under the terms of the GNU Public License

Usage: logrotate [-dfv?] [-d|--debug] [-f|--force] [-m|--mail command] [-s|--state statefile] [-v|--verbose] [-l|--log STRING] [--version]
        [-?|--help] [--usage] [OPTION...] <configfile>
```

Any number of config files may be given on the command line. Later config files may override the options given in earlier files, so the order in which the logrotate config files are listed is important. Normally, a single config file which includes any other config files which are needed should be used (e.g. `/etc/logrotate.conf`).

The **-s** (**--state**) option tells logrotate to use an alternate **state file**. The default state file is `/var/lib/logrotate.status` (not sure about this one, as it seems to default to `/var/lib/logratate/logrotate.conf` on my Centos machine). Logrotate writes to the `state file` the time of the most recent rotation of each log file, it then uses this information the next time it runs to determine whether to rotate a specific log file or not according to the [rotation interval settings](#rotation-frequency):

```bash
$ cat /var/lib/logrotate/logrotate.status
logrotate state -- version 2
"/var/log/nginx/error.log" 2018-12-31-3:0:0
"/var/log/yum.log" 2019-1-1-0:32:27
"/var/log/boot.log" 2019-1-5-21:36:2
"/var/log/httpd/error_log" 2019-1-6-0:13:40
"/var/log/chrony/*.log" 2018-12-3-3:0:0
"/var/log/wtmp" 2018-12-3-3:0:0
"/var/log/spooler" 2019-1-6-0:13:40
"/var/log/btmp" 2019-1-1-0:32:27
"/var/log/maillog" 2019-1-6-0:13:40
"/var/log/wpa_supplicant.log" 2018-12-3-3:0:0
"/var/log/secure" 2019-1-6-0:13:40
"/var/log/nginx/access.log" 2018-12-31-3:0:0
"/var/log/messages" 2019-1-6-0:13:40
"/var/log/cron" 2019-1-6-0:13:40
"/var/log/httpd/access_log" 2019-1-6-0:13:40
```

**NOTE:** the frequency at which you run the cron job is important. Logs will only be rotated when logrotate runs, regardless of configuration. For example, if you configure logrotate to rotate logs daily, but logrotate cron job runs only once a week, the logs will only be rotated once a week.

Another interesting option to note about logrotate command is **-f** (**--force**). This basically tell logrotate to force the rotation, even if it doesn't think this is necessary. For example:

```bash
$ date  # check today's date
Sun Jan  6 18:56:08 UTC 2019
$ fgrep wtmp /var/lib/logrotate/logrotate.status  # check the date of the last rotation of the wtmp log file
"/var/log/wtmp" 2018-12-3-3:0:0
$ logrotate /etc/logrotate.conf
$ fgrep wtmp /var/lib/logrotate/logrotate.status  # check the date of the last rotation again after running logrotate
"/var/log/wtmp" 2018-12-3-3:0:0
$ logrotate /etc/logrotate.conf -f  # force log rotation
$ fgrep wtmp /var/lib/logrotate/logrotate.status
"/var/log/wtmp" 2019-1-6-18:56:57
```

### Journal logs rotation

[Journald](logging.md) not only collects all system logs, but is also automatically performing their rotation. You can control how `journald` does the rotation by changing the following [configuration options](https://www.freedesktop.org/software/systemd/man/journald.conf.html):

| Option | Description |
|------|------|
| `SystemMaxUse` | The total maximum disk space that can be used for logs. Defaults to `10%` of the size of the file system and is capped to `4G`. |
| `SystemMaxUse` | The total maximum disk space that can be used for logs. Defaults to `15%` of the size of the file system and is capped to `4G`. |
| `SystemKeepFree` | The minimum amount of disk space that should be kept free for uses outside of systemd-journald’s logging functions. |
| `SystemMaxFileSize` | The maximum size of an individual journal file. Defaults to 1/8 of the values configured with `SystemMaxUse=` and `RuntimeMaxUse=`, so that usually seven rotated journal files are kept as history. |
| `SystemMaxFiles` | The maximum number of journal files that can be kept on disk. |

`Journald` will respect both `SystemMaxUse` and `SystemKeepFree`, and it will restrict the disk space usage by journal to the smallest of the two values.

These options apply to the journal files when stored on a persistent file system, more specifically `/var/log/journal`. There is parallel group of [configuration options](https://www.freedesktop.org/software/systemd/man/journald.conf.html) for the case when journal is set to only store logs in memory, more specifically `/run/log/journal`: `RuntimeMaxUse`, `RuntimeKeepFree`, `RuntimeMaxFileSize`, and `RuntimeMaxFiles`.

To view the currently used limits, you can run the following command:

```bash
$ journalctl -u systemd-journald -b --no-pager
...
Jan 03 04:40:54 host01.example.com systemd-journal[396]: Permanent journal is using 16.0M (max allowed 3.7G, trying to leave 4.0G free of 36.3G available → current limit 3.7G).
```

And the following command below can be used to simply check the current size of logs:

```bash
$ journalctl --disk-usage
Archived and active journals take up 16.0M on disk.
```

In addition to automatic log rotation, `journal` also provides [a set of commands that you can run to manually rotate the logs](https://www.linode.com/docs/quick-answers/linux/how-to-use-journalctl/#manually-clean-up-archived-logs).

### Resources used to create this document:

* https://support.rackspace.com/how-to/understanding-logrotate-utility/
* https://www.linode.com/docs/uptime/logs/use-logrotate-to-manage-log-files/
* https://linux.die.net/man/8/logrotate
* https://nbsoftsolutions.com/blog/introduction-to-journald-and-structured-logging
* https://www.freedesktop.org/software/systemd/man/journald.conf.html
* https://www.linode.com/docs/quick-answers/linux/how-to-use-journalctl/#control-the-size-of-your-logs-disk-usage
