## yum commands

**yum** is a primary tool for installing rpm packages. Unlike, [rpm command](rpm-commands.md) which requires you to resolve package dependencies before installation, `yum` provides automatic dependency resoultion by using [yum repositories](yum-repo.md).

### Package naming convention

When working with [rpm packages](rpm-package.md), it's important to know the **package naming convention**.

[RPM package](rpm-package.md) names have the following format:

```
name-version-release.architecture.rpm
```

where:

* `name` is the name of the package.
* `version` is version of the package.
* `release` specifies the release, i.e. the number of times this version of the software has been packaged. The release information also often includes OS details which are provided after the release number separated by a dot `.`.
* `architecture` specifies the architecture, i.e. the type of computer hardware the packaged software is meant to run on.
* `rpm` extention indicates that this is an [rpm package](rpm-package.md).

Let's look at the example of the package name:

![rpm-package-name-example](../pics/rpm-package-name-example.jpg)

In this example, name of the package is `httpd` which is followed by the version number `2.4.6`. After that goes the release information (`80.el7.centos`) which shows that this version of the package has been released (i.e. packaged) `80` times (probably for bug fixes) and was built for Centos 7 (`el7.centos` where `el7` stands for Enterprise Linux 7). The last piece of information included in the package name is the architecture (`x86_64)` that indicates that this package was built for 64-bit PC-type architecture.

### Useful commands

There is a great variety of yum commands, below I will provide examples of those that I find particularly useful.

#### Check information about a package

```bash
# syntax: yum info <package_name>[-<version>][-<release>]
$ yum info httpd -q
Available Packages
Name        : httpd
Arch        : x86_64
Version     : 2.4.6
Release     : 80.el7.centos
Size        : 2.7 M
Repo        : base/7/x86_64
Summary     : Apache HTTP Server
URL         : http://httpd.apache.org/
License     : ASL 2.0
Description : The Apache HTTP Server is a powerful, efficient, and extensible
            : web server.
```

If the package is not installed but available through some [yum repository](yum-repo.md), the package will be shown under the `Available Packages` section. Important to note here is the `Repo` field (`base/7/x86_64`) which shows the [ID of the yum repository](yum-repo.md) where this package is avaiable.

If some version of the package is already installed, it will be show under the `Installed Packages` section. If newer version of the packages are available for installation, they will be again shown under the `Available Packages` section:

```bash
$ yum info influxdb -q
Installed Packages
Name        : influxdb
Arch        : x86_64
Version     : 1.4.2
Release     : 1
Size        : 63 M
Repo        : installed
From repo   : influxdb
Summary     : Distributed time-series database.
URL         : https://influxdata.com
License     : Proprietary
Description : Distributed time-series database.

Available Packages
Name        : influxdb
Arch        : x86_64
Version     : 1.5.2
Release     : 1
Size        : 21 M
Repo        : influxdb/7/x86_64
Summary     : Distributed time-series database.
URL         : https://influxdata.com
License     : Proprietary
Description : Distributed time-series database.
```

#### List all available versions of the package

```bash
$ yum list available --showduplicates influxdb -q
Available Packages

...
influxdb.x86_64       1.3.5-1       influxdb
influxdb.x86_64       1.3.7-1       influxdb
influxdb.x86_64       1.4.2-1       influxdb
influxdb.x86_64       1.5.0-1       influxdb
influxdb.x86_64       1.5.1-1       influxdb
influxdb.x86_64       1.5.2-1       influxdb
```

where the columns of the output have the following format:

* the first column: `<package_name>.<architecture>`
* the second column: `<version>–<release>`
* the third column shows the [ID of the yum repository](yum-repo.md) in which the package is available.

#### Install a package

To install the latest version of the package, simply pass its name to the `install` command:

```bash
$ yum install influxdb
```

To install a specific version of the package, follow the [package naming converntion](#package-naming-convention) to specify the package you need.

For example:

```bash
# syntax: yum install <package name>[-<version>][-<release>]
$ yum install influxdb-1.4.2
$ yum install influxdb-1.5.0-1
```

#### Download a package without installing

If you want to only download a package without installing it, you can use `--downloadonly` and `--downloaddir` options:

```bash
$ yum install -q --downloadonly --downloaddir ./apache-packages httpd
```

Note that because [yum automatically resolves package dependencies](yum-repo.md), it will also download all of the dependent packages (which are currently not present on the system) as well:

```bash
$ ls ./apache-packages/
apr-1.4.8-3.el7_4.1.x86_64.rpm   httpd-2.4.6-80.el7.centos.1.x86_64.rpm        mailcap-2.1.41-2.el7.noarch.rpm
apr-util-1.5.2-6.el7.x86_64.rpm  httpd-tools-2.4.6-80.el7.centos.1.x86_64.rpm
```

#### Install a package from a local file, http or ftp

To install a package available locally or remotely via `http` or `ftp` protocols, use the **localinstall** command which instead of searching packages in the [enabled yum repositories](#get-a-list-of-avaliable-repositories), searches for packages in the given path. For example:

```bash
# syntax: yum localinstall <path-to-rpm-package>
$ yum localinstall ./apache-packages/httpd-2.4.6-80.el7.centos.1.x86_64.rpm
$ yum localinstall https://repos.influxdata.com/centos/7/x86_64/stable/influxdb-1.4.2.x86_64.rpm
```

#### Check if a package is installed

If you know the name of the package, use [yum info command](#check-information-about-a-package). If it's installed, then it will be shown under the `Installed Packages` section:

```bash
$ yum info influxdb -q
Installed Packages
Name        : influxdb
Arch        : x86_64
Version     : 1.4.2
Release     : 1
Size        : 63 M
Repo        : installed
Summary     : Distributed time-series database.
URL         : https://influxdata.com
License     : Proprietary
Description : Distributed time-series database.
```

If you don't know the exact package name, you can use `yum list installed` command and grep for a name pattern:

```bash
$ yum list installed | grep influx
influxdb.x86_64                 1.4.2-1                         installed
```

**Note** that in the [repository ID](yum-repo.md) field (third column) it says "installed". This indicates that the package wasn't installed from a repository (e.g. it was installed by using [localinstall command](#install-a-package-from-a-local-file-http-or-ftp)). If a package was installed from a repository, we would see a repository ID in the third colum. For example:

```bash
$ yum list installed | grep http
httpd.x86_64                    2.4.6-80.el7.centos.1           @updates
httpd-tools.x86_64              2.4.6-80.el7.centos.1           @updates
```

#### Check what packages are installed from a repository

The `yum list installed` command shows from which repository each package was installed. Thus, you can filter this output with [grep](grep.md) to see all the packages that were installed from a particular repo:

```bash
$ yum list installed | grep @influxdb
influxdb.x86_64                 1.6.4-1                         @influxdb
telegraf.x86_64                 1.8.2-1                         @influxdb
```

#### Upgrade a package

To upgrade a package to the latest version, just provide the package name to the `yum update` command:

```bash
# syntax: yum update <package_name(s)>
$ yum update influxdb
```

If package name(s) is not provided, all installed packaged will be upgraded if the upgrade is available:

```bash
$ yum update # will update all packages
```

#### Downgrade a package

Let's assume that I have the latest version of the influxdb package installed:

```bash
$ yum list available --showduplicates influxdb | tail -3 # check three most recent versions available
influxdb.x86_64                       1.6.2-1                          influxdb
influxdb.x86_64                       1.6.3-1                          influxdb
influxdb.x86_64                       1.6.4-1                          influxdb
$ yum info influxdb -q # check if a package is installed
Installed Packages
Name        : influxdb
Arch        : x86_64
Version     : 1.6.4
```

To downgrade to the previous version (`1.6.3` in this case), it's enough to provide the package name to the `yum downgrade` command:

```bash
$ yum downgrade influxdb -q
$ yum info influxdb -q
Installed Packages
Name        : influxdb
Arch        : x86_64
Version     : 1.6.3
```

To downgrade to a specific earlier version, you will need to add a version to the package name:

```bash
$ yum downgrade influxdb-1.6.2 -q
$ yum info influxdb -q
Installed Packages
Name        : influxdb
Arch        : x86_64
Version     : 1.6.2
```

#### Get a list of avaliable repositories

To see a list of **enabled repositories** (i.e. those repositories currently used by `yum` as source of rpm packages), use the following command:

```bash
$ yum repolist -q
repo id                                    repo name                                             status
base/7/x86_64                              CentOS-7 - Base                                       9,911
extras/7/x86_64                            CentOS-7 - Extras                                       432
influxdb/7/x86_64                          InfluxDB Repository - RHEL 7                            135
updates/7/x86_64                           CentOS-7 - Updates                                    1,602
```

The first 2 columns are self-explanatory and the `status` column shows the total number of packages avaliable in each repo.

To list all available repositories, i.e. enabled and disabled, use the following command:

```bash
$ yum repolist all
```

To get a more detailed information about a specific repository, use the `repoinfo` command:

```bash
$ yum repoinfo influxdb -q

Repo-id      : influxdb/7/x86_64
Repo-name    : InfluxDB Repository - RHEL 7
Repo-status  : enabled
Repo-revision: 1539807463
Repo-updated : Wed Oct 17 20:17:44 2018
Repo-pkgs    : 135
Repo-size    : 1.6 G
Repo-baseurl : https://repos.influxdata.com/centos/7/x86_64/stable/
Repo-expire  : 21,600 second(s) (last: Sun Oct 28 18:47:55 2018)
  Filter     : read-only:present
Repo-filename: /etc/yum.repos.d/influxdb.repo
```

From this output you can learn:

* `Repo-updated`: the time the repository was last updated.
* `Repo-pkgs`: the number of packages in the repo.
* `Repo-size`: the total size of the repository.
* `Repo-baseurl`: the repository URL from where the data is fetched. Note that it shows the URL with expanded [$release and $basearch variables](yum-repo.md) which you will often see in the configuration file for a repository.
* `Repo-expire:` the time the repository metadata was fetched the last time.
* `Repo-filename`: the location of configuration file for this repository.

#### --enablerepo and --disablerepo flags

Among all the options, I often find very useful `--enablerepo` and `--disablerepo` options. They allow to disable or enable repositories for a single command.

For example, I have the `httpd` package available in 2 repositories (`base` and `updates`):

```bash
$ yum list available --showduplicates httpd -q
Available Packages
httpd.x86_64                        2.4.6-80.el7.centos                          base
httpd.x86_64                        2.4.6-80.el7.centos.1                        updates
```

To make sure, it's installed from `base` repository, I could run the following command:

```bash
$ yum --disablerepo "updates" install httpd -q

========================================================================================
 Package              Arch            Version                       Repository     Size
========================================================================================
Installing:
 httpd                x86_64          2.4.6-80.el7.centos           base          2.7 M
Installing for dependencies:
 apr                  x86_64          1.4.8-3.el7_4.1               base          103 k
 apr-util             x86_64          1.5.2-6.el7                   base           92 k
 httpd-tools          x86_64          2.4.6-80.el7.centos           base           89 k
 mailcap              noarch          2.1.41-2.el7                  base           31 k
```

Without it, it would try to install the package from `updates` repo:

```bash
$ yum install httpd -q

========================================================================================
 Package             Arch           Version                       Repository       Size
========================================================================================
Installing:
 httpd               x86_64         2.4.6-80.el7.centos.1         updates         2.7 M
Installing for dependencies:
 apr                 x86_64         1.4.8-3.el7_4.1               base            103 k
 apr-util            x86_64         1.5.2-6.el7                   base             92 k
 httpd-tools         x86_64         2.4.6-80.el7.centos.1         updates          90 k
 mailcap             noarch         2.1.41-2.el7                  base             31 k
```

#### Working with a specific repository

You're likely to have multiple repositories enabled on your system, but in case you want to run a few commands against a specific repository, you can use `yum repo-pkgs` command which has the following syntax:

```bash
yum repo-pkgs repo_id <list|install|remove|upgrade|reinstall> [pkg]
```

For example, to list packages from the influxdb repository, I can run the following command:

```bash
$ yum repo-pkgs influxdb list -q
Installed Packages
influxdb.x86_64                                       1.6.4-1                                      @influxdb
telegraf.x86_64                                       1.8.2-1                                      @influxdb
Available Packages
chronograf.x86_64                                     1.6.2-1                                      influxdb
kapacitor.x86_64                                      1.5.1-1                                      influxdb
```

To remove all packages installed from this repository, I could run the following command:

```bash
$ yum repo-pkgs influxdb remove -q

============================================================================================================
 Package                  Arch                   Version                    Repository                 Size
============================================================================================================
Removing:
 influxdb                 x86_64                 1.6.4-1                    @influxdb                  77 M
 telegraf                 x86_64                 1.8.2-1                    @influxdb                  46 M

Transaction Summary
============================================================================================================
Remove  2 Packages

Is this ok [y/N]:
```

#### Which package a file belongs to?

`yum provides` command allows to find out which package a specific file belongs to. For example, if you would like to know the name of the package that creates `/etc/httpd/conf/httpd.conf` file, you could use the following command:

```bash
$ yum provides /etc/httpd/conf/httpd.conf -q
httpd-2.4.6-80.el7.centos.x86_64 : Apache HTTP Server
Repo        : base
Matched from:
Filename    : /etc/httpd/conf/httpd.conf


httpd-2.4.6-80.el7.centos.1.x86_64 : Apache HTTP Server
Repo        : updates
Matched from:
Filename    : /etc/httpd/conf/httpd.conf
```

This command especially comes in handy, when you want to install a program that's part of a package with a different name than the name of the program. For example, if I want to install [netstat](netstat.md) and run `yum install netstat`, I will get an error, because there is no package available with such name:

```bash
$ yum install netstat -q
Error: Nothing to do
```

To see if `netstat` can be installed as part of a different package, I can use the `yum provides` command:

```bash
$ yum provides netstat -q
net-tools-2.0-0.22.20131004git.el7.x86_64 : Basic networking tools
Repo        : base
Matched from:
Filename    : /bin/netstat
```

The above output showed me that the `netstat` executable file (`/bin/netstat`) comes as part of the `net-tools` package. So in order to install `netstat`, I need to run `yum install net-tools`.

#### Remove a package

Removing packages is a bit tricky since there are several ways you can do it.

As an example, let's install the apache package:

```bash
$ yum install -q httpd

============================================================================================================
 Package                  Arch                Version                            Repository            Size
============================================================================================================
Installing:
 httpd                    x86_64              2.4.6-80.el7.centos.1              updates              2.7 M
Installing for dependencies:
 apr                      x86_64              1.4.8-3.el7_4.1                    base                 103 k
 apr-util                 x86_64              1.5.2-6.el7                        base                  92 k
 httpd-tools              x86_64              2.4.6-80.el7.centos.1              updates               90 k
 mailcap                  noarch              2.1.41-2.el7                       base                  31 k

Transaction Summary
============================================================================================================
Install  1 Package (+4 Dependent packages)
```

As you can see from the output, in addition to the apache package itself (`httpd`), it also installs 4 dependent packages (`apr`, `apr-util`, `httpd-tools`, `mailcap`) that apache package requires to be installed.

Let's see what happens, if I now try to remove the apache package using `yum remove` command:

```bash
$ yum remove -q httpd

============================================================================================================
 Package             Arch                 Version                              Repository              Size
============================================================================================================
Removing:
 httpd               x86_64               2.4.6-80.el7.centos.1                @updates               9.4 M

Transaction Summary
============================================================================================================
Remove  1 Package

Is this ok [y/N]:
```

It tries to remove only one package, i.e. the apache package itself, whereas we just installed 5 packages as part of the apache installation...

There are a few ways we can deal with this problem.

To remove apache package and its dependencies that are not longer needed by other packages, we can use `yum autoremove` command:

```bash
$ yum autoremove -q httpd

---> Marking apr-util to be removed - no longer needed by httpd
---> Marking apr to be removed - no longer needed by httpd
---> Marking mailcap to be removed - no longer needed by httpd
---> Marking httpd-tools to be removed - no longer needed by httpd

============================================================================================================
 Package                 Arch               Version                              Repository            Size
============================================================================================================
Removing:
 httpd                   x86_64             2.4.6-80.el7.centos.1                @updates             9.4 M
Removing for dependencies:
 apr                     x86_64             1.4.8-3.el7_4.1                      @base                221 k
 apr-util                x86_64             1.5.2-6.el7                          @base                194 k
 httpd-tools             x86_64             2.4.6-80.el7.centos.1                @updates             169 k
 mailcap                 noarch             2.1.41-2.el7                         @base                 62 k

Transaction Summary
============================================================================================================
Remove  1 Package (+4 Dependent packages)

Is this ok [y/N]:
```

You can also make this the default behavior of the `yum remove` command, if you enable [clean_requirements_on_remove](https://www.tecmint.com/remove-package-with-dependencies-using-yum/) option in the [yum main configuration](yum-repo.md).

```bash
$ fgrep 'clean_requirements' /etc/yum.conf
clean_requirements_on_remove=1
$ yum remove -q httpd

---> Marking apr-util to be removed - no longer needed by httpd
---> Marking apr to be removed - no longer needed by httpd
---> Marking mailcap to be removed - no longer needed by httpd
---> Marking httpd-tools to be removed - no longer needed by httpd

============================================================================================================
 Package                 Arch               Version                              Repository            Size
============================================================================================================
Removing:
 httpd                   x86_64             2.4.6-80.el7.centos.1                @updates             9.4 M
Removing for dependencies:
 apr                     x86_64             1.4.8-3.el7_4.1                      @base                221 k
 apr-util                x86_64             1.5.2-6.el7                          @base                194 k
 httpd-tools             x86_64             2.4.6-80.el7.centos.1                @updates             169 k
 mailcap                 noarch             2.1.41-2.el7                         @base                 62 k

Transaction Summary
============================================================================================================
Remove  1 Package (+4 Dependent packages)

Is this ok [y/N]:
```

Finally, the easy way to revert a yum transaction would be to use the [yum history undo command](#revertining-and-repeating-transactions):

```bash
$ yum history undo last -q
Undoing transaction 28, from Mon Oct 29 00:00:55 2018
    Dep-Install apr-1.4.8-3.el7_4.1.x86_64               @base
    Dep-Install apr-util-1.5.2-6.el7.x86_64              @base
    Install     httpd-2.4.6-80.el7.centos.1.x86_64       @updates
    Dep-Install httpd-tools-2.4.6-80.el7.centos.1.x86_64 @updates
    Dep-Install mailcap-2.1.41-2.el7.noarch              @base

============================================================================================================
 Package                 Arch               Version                              Repository            Size
============================================================================================================
Removing:
 apr                     x86_64             1.4.8-3.el7_4.1                      @base                221 k
 apr-util                x86_64             1.5.2-6.el7                          @base                194 k
 httpd                   x86_64             2.4.6-80.el7.centos.1                @updates             9.4 M
 httpd-tools             x86_64             2.4.6-80.el7.centos.1                @updates             169 k
 mailcap                 noarch             2.1.41-2.el7                         @base                 62 k

Transaction Summary
============================================================================================================
Remove  5 Packages

Is this ok [y/N]:
```

### yum history & logs

#### Listing yum history

The `yum history` command allows to review information about a timeline of yum transactions, the dates and times they occurred, the number of packages affected, whether these transactions succeeded or were aborted. All history data is stored in the history DB in the `/var/lib/yum/history/` directory.

```bash
# show 20 most recent transactions
$ yum history
$ yum history list
# show all transactions
$ yum history list all
# show transactions in a given range
# syntax: yum history list <start_id>..<end_id>
$ yum history list 1..3
```

Let's look at the example of the output these commands produce:

```bash
$ yum history list -q
ID     | Command line             | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------
    21 | install telnet           | 2018-10-28 18:48 | Install        |    1
    20 | install lsof             | 2018-10-28 18:47 | Install        |    1
    19 | install httpd            | 2018-10-28 01:11 | Install        |    5
    18 | install telegraf         | 2018-10-28 01:04 | Install        |    1 EE
    17 | history undo 7           | 2018-10-27 04:41 | Erase          |    4
    16 | remove httpd             | 2018-10-27 04:34 | Erase          |    1
    15 | update influxdb          | 2018-10-27 04:06 | Update         |    1
    14 | downgrade influxdb-1.6.2 | 2018-10-26 06:01 | Downgrade      |    1
    13 | update influxdb          | 2018-10-26 06:01 | Update         |    1
    12 | downgrade influxdb -q    | 2018-10-26 05:59 | Downgrade      |    1
```

In the output, the most recent transaction is displayed at the top of the list. Let's look at the meaning of each column:
* `ID`: an integer value that identifies a particular transaction.
* `Command line`: the yum command that was run.
* `Date and time`: the date and time of a transaction.
* `Action(s)`: a [list of actions](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-Yum-Transaction_History#tabl-Yum-Transaction_History-Actions) that were performed during a transaction.
* `Altered`: the number of packages that were affected by a transaction, possibly followed by additional information as described [here](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-Yum-Transaction_History#tabl-Yum-Transaction_History-Altered).

`yum` also enables you to display a summary of all past transactions:

```bash
$ yum history summary -q
Login user                 | Time                | Action(s)        | Altered
-------------------------------------------------------------------------------
vagrant <vagrant>          | Last day            | Install          |        8
vagrant <vagrant>          | Last week           | D, E, I, R, U    |       23
System <unset>             | Last 6 months       | Install          |      306
```

#### Tracing history of a specific package

You can also list only transactions regarding a particular package or packages. To do so, use the command with a package name or a glob expression:

```bash
$ yum history list httpd\* -q
ID     | Command line             | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------
    19 | install httpd            | 2018-10-28 01:11 | Install        |    5
    17 | history undo 7           | 2018-10-27 04:41 | Erase          |    4
    16 | remove httpd             | 2018-10-27 04:34 | Erase          |    1
     7 | install httpd            | 2018-10-25 05:57 | Install        |    5
```

Note that the wildcard symbols (e.g. `*`) has to be escaped with `\` sign.

To also see full names of the packages, use the `yum history package-list` command:

```bash
$ yum history package-list httpd\* -q
ID     | Action(s)      | Package
-------------------------------------------------------------------------------
    19 | Install        | httpd-2.4.6-80.el7.centos.1.x86_64
    19 | Dep-Install    | httpd-tools-2.4.6-80.el7.centos.1.x86_64
    17 | Erase          | httpd-tools-2.4.6-80.el7.centos.1.x86_64
    16 | Erase          | httpd-2.4.6-80.el7.centos.1.x86_64
     7 | Install        | httpd-2.4.6-80.el7.centos.1.x86_64
     7 | Dep-Install    | httpd-tools-2.4.6-80.el7.centos.1.x86_64
```

#### Examining transactions

To examine a particular transaction or transactions in more detail, use the following command:

```bash
# show info about the last transaction
$ yum history info
# show info about transaction identified by id
$ yum history info 17
# show info about a range of trasactions
$ yum history info 17..18
```

For example, this is a sample output for two transactions, each installing one new package:

```bash
$ yum history info 20..21 -q

Transaction ID : 20..21
Begin time     : Sun Oct 28 18:47:59 2018
Begin rpmdb    : 316:f4c5603cd57dca8a16fa82e10ea9a84b8f9a9c40
End time       :            18:48:07 2018 (8 seconds)
End rpmdb      : 318:0225de8f9a551287a258d6652bfb366cf126c2d3
User           : vagrant <vagrant>
Return-Code    : Success
Command Line   : install lsof
Command Line   : install telnet
Transaction performed with:
    Installed     rpm-4.11.3-32.el7.x86_64                      @anaconda
    Installed     yum-3.4.3-158.el7.centos.noarch               @anaconda
    Installed     yum-plugin-fastestmirror-1.1.31-45.el7.noarch @anaconda
Packages Altered:
    Install lsof-4.87-5.el7.x86_64      @base
    Install telnet-1:0.17-64.el7.x86_64 @base
```

You can also view additional information, such as what configuration options were used at the time of the transaction, or from what repository and why were certain packages installed with [addon-info command](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-Yum-Transaction_History#tabl-Yum-Transaction_History-examining).

#### Revertining and repeating transactions

Apart from reviewing the transaction history, the yum history command provides means to revert or repeat a selected transaction.

```bash
# revert a transaction
$ yum history undo <id>
# redo a transaction
$ yum history redo <id>
```

Note that both `undo` and `redo` commands only revert or repeat the steps that were performed during a transaction. If the transaction installed a new package, the `undo` command will uninstall it, and if the transaction uninstalled a package the command will again install it. This command also attempts to downgrade all updated packages to their previous version, if these older packages are still available.

#### yum logs

`yum` also has a log file where it writes info about the packages being installed, updated, removed, etc.

```bash
$ tail -3 /var/log/yum.log
Oct 28 18:47:59 Installed: lsof-4.87-5.el7.x86_64
Oct 28 18:48:07 Installed: 1:telnet-0.17-64.el7.x86_64
Oct 28 19:37:00 Updated: yum-utils-1.1.31-46.el7_5.noarch
```

### Working with yum cache

By default, `yum` deletes downloaded data files when they are no longer needed after a successful operation. This minimizes the amount of storage space that `yum` uses. However, you can [enable caching](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-working_with_yum_cache), so that the package files downloaded by yum stay in cache directories. By using cached data, you can carry out certain operations without a network connection.

```bash
# check if repo caching is configured globally
$ fgrep keepcache /etc/yum.conf
keepcache=1
```

If caching is enabled, `yum` stores temporary files in the `/var/cache/yum/$basearch/$releasever/` directory, where `$basearch` and `$releasever` are [yum variables](yum-repo.md) referring to base architecture of the system and the release version. You can find the values for the `$basearch` and `$releasever` variables in the output of the yum version command:

```bash
$ yum version -q
Installed: 7/x86_64 # shows $releasever/$basearch
Group-Installed: yum
```

Each configured repository has one subdirectory. For example, the directory `/var/cache/yum/$basearch/$releasever/influxdb/packages/` will hold packages downloaded from the influxdb repository:

```bash
$ ls /var/cache/yum/x86_64/7/influxdb/packages/
$ yum install influxdb -q -y
$ ls /var/cache/yum/x86_64/7/influxdb/packages/
influxdb-1.6.4.x86_64.rpm
```

Besides, it will also download [repository metadata](yum-repo.md) which contains all the information about the packages available in the repository. This means that if you have caching enabled, you won't be able to see repository updates instantly (e.g. when a new version of the package was added), because yum will be using cache data until it is expired. For example:

```bash
$ yum update influxdb
...
No Packages marked for Update
```

By default, `yum` will hold cache data for 6 hours before it invalidates/removes it. You can overwrite this with a [metadata_expire config option](https://linux.die.net/man/5/yum.conf) or use one of the [yum clean commands](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-working_with_yum_cache). In most cases, **expire-cache** options is enough to use if you want to see the repo updates:

```bash
$ yum clean expire-cache
```

([read more about yum caching](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-working_with_yum_cache))

### Further reading

I found particularly useful the links below which you can also check out:

* [yum command cheatsheat](https://access.redhat.com/sites/default/files/attachments/rh_yum_cheatsheet_1214_jcs_print-1.pdf).
* [How to use yum command on CentOS/RHEL](https://www.cyberciti.biz/faq/rhel-centos-fedora-linux-yum-command-howto/)

### Resources used to create this document:

* https://www.systutorials.com/2346/installing-specific-old-versions-of-packages-in-yum/
* https://kerneltalks.com/howto/how-to-list-yum-repositories-in-rhel-centos/
* https://www.tecmint.com/install-particular-package-version-in-centos-ubuntu-debian/
* https://kerneltalks.com/tools/understanding-package-naming-convention-rpm-deb/
* http://ftp.rpm.org/max-rpm/ch-rpm-file-format.html
* https://access.redhat.com/sites/default/files/attachments/rpm_tutorial_20120831.pdf
* https://access.redhat.com/sites/default/files/attachments/rh_yum_cheatsheet_1214_jcs_print-1.pdf
* https://unix.stackexchange.com/questions/304348/how-to-check-when-yum-metadata-will-expire
* https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-working_with_yum_cache
* https://www.tecmint.com/20-linux-yum-yellowdog-updater-modified-commands-for-package-mangement/
* https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-yum-transaction_history
* https://www.tecmint.com/remove-package-with-dependencies-using-yum/
* https://www.certdepot.net/rhel7-how-to-keep-your-system-clean/
* https://www.cyberciti.biz/faq/rhel-centos-fedora-linux-yum-command-howto/
* https://access.redhat.com/sites/default/files/attachments/rh_yum_cheatsheet_1214_jcs_print-1.pdf
