## date

The **date** command is used to print system time and date. It can also set system time and date, but you should generally use [ntp](ntp.md) daemon to handle this for you.

When used without options, the `date` command displays the current system time and date:

```bash
$ date
Sun Sep 23 03:55:04 UTC 2018
```

As you can see, the default output includes the day of the week (`Sun`), the day of the month (`Sep 23`), time (`03:55:04`), timezone (`UTC`), and year (`2018`).

### Useful options

#### Operate on a specific date

By default, the `date` displays information about the current date and time, but you can make it show information about a specific date in the past or the future with the **-d** option.

For example, to find out what day of the week will be October 26 of this year, I could use one of the following commands:

```bash
$ date -d 'October 26'
Fri Oct 26 00:00:00 UTC 2018
$ date -d '2018-10-26'
Fri Oct 26 00:00:00 UTC 2018
$ date -d '10/26/2018'
Fri Oct 26 00:00:00 UTC 2018
```

`date` also accepts words like day of the week, "today", and "yesterday":

```bash
$ date -d yesterday
Sat Sep 22 04:11:12 UTC 2018
$ date -d friday
Fri Sep 28 00:00:00 UTC 2018
$ date -d sun
Sun Sep 23 00:00:00 UTC 2018
```

Notice how for future dates the time is set to be `00:00:00` and for past dates it displays the current system time.

Other valid date strings include: `last-week`, `next-week`, `last-month`, `next-month`, `last-year`, and `next-year`.

#### Count seconds from epoch

`date` can display given date/time in **Unix epoch time format**, i.e the number of seconds elapsed since 00:00:00, Jan 1, 1970. For this to work, you need to use `%s` output [format specifier](#specify-output-format).

The following example will show you the seconds from epoch to the current time:

```bash
$ date +%s
1537676319
```

And again, use can use the [-d option](#operate-on-a-specific-date) to count the number of seconds since the epoch to a specific date.

```bash
$ date -d '10/26/2018' +%s
1540512000
```

#### Convert seconds from the epoch to a human readable date

To convert seconds from the epoch to a human readable date, use the **-d** option and prepend the number of seconds with the `@` sign:

```bash
$ date -d @1540512000
Fri Oct 26 00:00:00 UTC 2018
```

Note that the command above won't work on Mac OSX, use the following command instead:

```bash
$ date -r 1540512000
Thu Oct 25 17:00:00 PDT 2018
```

Notice how it showed a different date because of the [timezone difference](#specify-a-timezone).

#### Specify a timezone

To specify a different [timezone](timezones.md) other than the system default, set the `TZ` environment variable when running the `date` command:

```bash
$ date
Sun Sep 23 04:55:20 UTC 2018
$ TZ=Europe/Moscow date
Sun Sep 23 07:55:22 MSK 2018
$ date -d @1540512000
Fri Oct 26 00:00:00 UTC 2018
$ TZ=US/Pacific date -d @1540512000
Thu Oct 25 17:00:00 PDT 2018
```

You can find available timezones by listing `/usr/share/zoneinfo` directory:

```bash
$ ls /usr/share/zoneinfo/
Africa      Brazil   Egypt    GB-Eire    HST          Japan        MST7MDT   posix       Singapore  WET
America     Canada   Eire     GMT        Iceland      Kwajalein    Navajo    posixrules  Turkey     W-SU
Antarctica  CET      EST      GMT0       Indian       leapseconds  NZ        PRC         tzdata.zi  zone1970.tab
Arctic      Chile    EST5EDT  GMT-0      Iran         Libya        NZ-CHAT   PST8PDT     UCT        zone.tab
Asia        CST6CDT  Etc      GMT+0      iso3166.tab  MET          Pacific   right       Universal  Zulu
Atlantic    Cuba     Europe   Greenwich  Israel       Mexico       Poland    ROC         US
Australia   EET      GB       Hongkong   Jamaica      MST          Portugal  ROK         UTC
```

There is also the **-u** option which makes the `date` command act as if the `TZ` environment variable was set to `UTC`.

```bash
$ date -u
Sun Sep 23 05:10:21 UTC 2018
```

#### Specify output format

You can overwrite the default output of the `date` command using format specifiers. Examples of format specifiers include:

* `%F`: displays the date `YYYY-MM-DD` format;
* `%T`: displays the time;
* `%A`: displays full weekday name;

To see a full list of specifiers, user the `date --help` or `man date` command.

You specify output format after the `+` sign:

```bash
$ date +'%F %T'
2018-09-23 05:31:21
$ date +%A
Sunday
```

You will often see the `date` command to be used as part of the scripts for creating scheduled backups.

For example, the following command will create a backup of `/home/vagrant/` directory and include the current date in the name of the resulting archive file:

```bash
$ tar cfz /backup-`date +%F`.tar.gz /home/vagrant/
$ ls /backup-2018-09-23.tar.gz
/backup-2018-09-23.tar.gz
```

### Resources used to create this document:

* https://www.linode.com/docs/tools-reference/tools/use-the-date-command-in-linux/
* https://www.howtoforge.com/linux-date-command/
* man date
