## yum repository

Red Hat Enterprise Linux, CentOS, Fedora, and many other Linux distributions group their software together in
[RPM packages](rpm-package.md).

At first, the [rpm](rpm-commands.md) command was the primary tool created to install and manage RPM packages. Now the [yum](yum-commands.md)
command and related **yum repositories** are more commonly used.

### yum versus rpm

There are two commonly used tools related to RPM package management, [yum](yum-commands.md) and [rpm](rpm-commands.md). (Recent Fedora versions have replaced yum with **dnf**, a rewrite with similar functionality.)

The **yum** (stands for `Yellowdog Updater, Modified`) utility was created because [rpm tool](rpm-commands.md) lacked some important features the most prominent of which was management of package dependencies. Although each RPM package stored a list of components it depended on, there was no way for the rpm command to satisfy those dependencies automatically. You had to hunt down each dependent package yourself and make sure you had them all available in a local directory before you could install the package you wanted.

_`yum` changed that by making installation and management of package dependencies the responsibility of the package management tool._ When Red Hat or another RPM-based Linux disto created a set of RPM packages, they would store those packages in a **software repository** (**yum repository**) that was accessible on the network. The developers would make sure that all packages needed by other packages in the repository were in that repository as well. So when someone used the [yum command](yum-commands.md) to install a package, `yum` would download that package from the repository along with any dependent packages needed to make the requested package work. Then all the necessary packages could be installed together.

_Summary:_
* [rpm](rpm-commands.md) is the low-level tool which operates on explicit set of RPM packages. Unlike [yum](yum-command.md) utility, [rpm](rpm-commands.md) cannot automatically resolve package dependencies and requires you to download all dependent packages yourself.
* [yum](yum-command.md) uses [rpm](rpm-commands.md) as a library to install packages.

### yum repositories

A **yum repository** is a collection of RPM packages with metadata that is readable by the [yum command line tool](yum-commands.md).

Information about repositories is stored in configuration files.

#### Configuration files: [main] section

The configuration file for yum and related utilities is located at `/etc/yum.conf`. This file contains one mandatory **[main] section**, which allows you to set `yum` command options that have global effect, and can also contain one or more **[repository] sections**, which allow you to set repository-specific options.

```bash
$ cat /etc/yum.conf
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=0
debuglevel=2
logfile=/var/log/yum.log
exactarch=1
obsoletes=1
gpgcheck=1
plugins=1
installonly_limit=5
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum
distroverpkg=centos-release
```

(see [explanation of the most commonly-used options in the [main] section](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-configuring_yum_and_yum_repositories) & [yum variables](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-Using_Yum_Variables))

#### Configuration files: [repository] sections

It is recommended to define individual repositories configurations in new or existing **.repo files** in the `/etc/yum.repos.d/` directory. The values you define in individual `[repository] sections` in `.repo` files override values set in the [[main] section](#[main]-section):

```bash
$ ls /etc/yum.repos.d/
CentOS-Base.repo  CentOS-Debuginfo.repo  CentOS-Media.repo    CentOS-Vault.repo
CentOS-CR.repo    CentOS-fasttrack.repo  CentOS-Sources.repo  influxdb.repo
```

Let's look at the definition of the influxdb repository as an example:

```bash
$ cat /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL $releasever
baseurl = https://repos.influxdata.com/centos/$releasever/$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
```

The value inside the square brackets `[]` (i.e.`influxdb`) is the unique repository ID and it indicates the beginning of the configuration definition for the repository. Note, the repository ID cannot contain spaces.

Every `[repository]` section must contain the following directives:

* `name` is a human-readable string describing the repository.
* `baseurl` is a URL to the directory where the [repodata directory](#contents-of-a-yum-repository) of a repository is located:
    * If the repository is available over HTTP, use: `http://path/to/repo`
    * If the repository is available over FTP, use: `ftp://path/to/repo`
    * If the repository is local to the machine, use: `file:///path/to/local/repo`
  
In the `baseurl` value, you will often see [$releasever, $arch, and $basearch yum variables](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-Using_Yum_Variables). For example:

```
baseurl = https://repos.influxdata.com/centos/$releasever/$basearch/stable
```

`yum` always expands these variables in URLs prior to making a request to the repo:

* `$releasever` variable is used to reference the release version of Linux distribution.
* `$arch` variable is used to refer to the system's CPU architecture.
* `$basearch` variable is used to reference the base architecture of the system. For example, i686 machines have a base architecture of i386, and AMD64 and Intel 64 machines have a base architecture of x86_64.

([see how you can check the values of these variables](https://unix.stackexchange.com/questions/19701/yum-how-can-i-view-variables-like-releasever-basearch-yum0))

Other directives that you will often see in `[repository] sections` include:

* `enabled` Either '1' or '0'. This tells yum whether or not use this repository.
* `gpgcheck` Either '1' or '0'. This tells yum whether or not it should perform a [GPG signature check](#package-signing) on the packages gotten from this repository.
* `gpgkey` A URL pointing to the [ASCII-armored GPG key file](gpg.md) for the repository. This option is used if yum needs a public key to verify a package and the required key hasn't been imported into the RPM database.

(see [list of all options for the [repository] section](https://linux.die.net/man/5/yum.conf))

### Contents of a yum repository

To better understand what a yum repository is made of, let's create a local copy of a remote repository. As an example, I will use the influxdb repository configuration that has been discussed above.

First, I will copy packages from a remote repository to a local directory using **reposync** tool:

```bash
$ reposync \
  -r influxdb \
  -p /opt/local-mirror \
  -n \
  /opt/local-repo-mirror
(1/4): chronograf-1.6.2.x86_64.rpm                         |  15 MB   00:06
(2/4): influxdb-1.6.4.x86_64.rpm                           |  24 MB   00:07
(3/4): telegraf-1.8.2-1.x86_64.rpm                         |  13 MB   00:05
(4/4): kapacitor-1.5.1.x86_64.rpm                          |  22 MB   00:07
```

The above command dowloaded the latest rpm packages from influxdb remote repository to my local directory:

```bash
$ ls /opt/local-mirror/
influxdb
$ ls /opt/local-mirror/influxdb/
chronograf-1.6.2.x86_64.rpm  kapacitor-1.5.1.x86_64.rpm
influxdb-1.6.4.x86_64.rpm    telegraf-1.8.2-1.x86_64.rpm
```

To make a `yum repository` out of this folder with packages, I need to build some metadata information using the **createrepo** tool:

```bash
$ ls /opt/local-mirror/
influxdb
$ createrepo /opt/local-mirror/
Spawning worker 0 with 4 pkgs
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
$ ls /opt/local-mirror/
influxdb  repodata
```

As you can see, the `createrepo` command created a **repodata** directory. Let's look at what files it holds.

```bash
$ ls /opt/local-mirror/repodata/
36a6d5e729153eddf636f8b179a93ef717ea09aa66e788fb34c954b7124daebc-primary.sqlite.bz2
53ae985da57cd11e7251af269e20c0735ded9ee97a72fe418e9458d69e067d99-other.xml.gz
762758eb22f55a22f8247a0f8275b7d0998197c7a368419a31bfa9b992be2dc1-primary.xml.gz
8abfa32cc85b5d21d9216e7b322d52ce8af2919438d5ddce6a5c3f69173914c1-filelists.xml.gz
8ba49e9e831dfbdba317c2f21d4a1306804a5b08efe710a539aceaee4e2c3180-filelists.sqlite.bz2
a6044fecfd985e895880c464fa37483dd33191dccd99ac60b5af412d3c6372f7-other.sqlite.bz2
repomd.xml
```

#### filelists files

`filelists` file will contain information about all the files each package in the repository will install on the system. If you dont have `filelists` inside your repository, then you will not be able to use [yum to search for a package that provides/installs a particular file on the system](yum-commands.md).

```bash
$ zcat /opt/local-mirror/repodata/8abfa32cc85b5d21d9216e7b322d52ce8af2919438d5ddce6a5c3f69173914c1-filelists.xml.gz

<?xml version="1.0" encoding="UTF-8"?>
<filelists xmlns="http://linux.duke.edu/metadata/filelists" packages="4">
<package pkgid="e99c74b2112983d815e4b46562ef2bb3c529ce0592d658e615daac06668955d4" name="telegraf" arch="x86_64">
  <version epoch="0" ver="1.8.2" rel="1"/>
  <file>/etc/logrotate.d/telegraf</file>
  <file>/etc/telegraf/telegraf.conf</file>
  <file>/usr/bin/telegraf</file>
  <file>/usr/lib/telegraf/scripts/init.sh</file>
  <file>/usr/lib/telegraf/scripts/telegraf.service</file>
  <file type="dir">/etc/telegraf/telegraf.d</file>
  <file type="dir">/var/log/telegraf</file>
</package>
...
```

Each package in the repository is described inside the `<package>` tags. As you can see from the `telegraf` package example, it specifies the name of the package (`name="telegraf`), architecture (`arch="x86_64"`) this packages was built for, version of the package (`ver="1.8.2"`). It also gives the information about all the files (inside the `<file>` tags) and directories (inside the `<file type="dir">` tags) that are going to be installed on the system when you install this package.

#### primary files

`primary` files contains primary information about each package in the repository.

```bash
$ zcat /opt/local-mirror/repodata/762758eb22f55a22f8247a0f8275b7d0998197c7a368419a31bfa9b992be2dc1-primary.xml.gz

...
<package type="rpm">
  <name>telegraf</name>
  <arch>x86_64</arch>
  <version epoch="0" ver="1.8.2" rel="1"/>
  <checksum type="sha256" pkgid="YES">e99c74b2112983d815e4b46562ef2bb3c529ce0592d658e615daac06668955d4</checksum>
  <summary>Plugin-driven server agent for reporting metrics into InfluxDB.</summary>
  <description>Plugin-driven server agent for reporting metrics into InfluxDB.</description>
  <packager>support@influxdb.com</packager>
  <url>https://github.com/influxdata/telegraf</url>
  <time file="1539807459" build="1539805338"/>
  <size package="13643887" installed="48098460" archive="48099572"/>
  <location href="influxdb/telegraf-1.8.2-1.x86_64.rpm"/>
  <format>
    <rpm:license>MIT</rpm:license>
    <rpm:vendor>InfluxData</rpm:vendor>
    <rpm:group>default</rpm:group>
    <rpm:buildhost>a54239ba414c</rpm:buildhost>
    <rpm:sourcerpm>telegraf-1.8.2-1.src.rpm</rpm:sourcerpm>
    <rpm:header-range start="4392" end="14701"/>
    <rpm:provides>
      <rpm:entry name="telegraf" flags="EQ" epoch="0" ver="1.8.2" rel="1"/>
      <rpm:entry name="telegraf(x86-64)" flags="EQ" epoch="0" ver="1.8.2" rel="1"/>
    </rpm:provides>
    <rpm:requires>
      <rpm:entry name="/bin/sh"/>
      <rpm:entry name="/bin/sh" pre="1"/>
      <rpm:entry name="coreutils"/>
      <rpm:entry name="shadow-utils"/>
    </rpm:requires>
  <file>/etc/logrotate.d/telegraf</file>
  <file>/etc/telegraf/telegraf.conf</file>
  <file>/usr/bin/telegraf</file>
  <file type="dir">/etc/telegraf/telegraf.d</file>
  </format>
</package>
...
```

As you can see from the `telegraf` package example, you can find information like the package name, version, checksum, description, packager contact info (`<packager>support@influxdb.com</packager>`), link to the project page name of the package (`<url>https://github.com/influxdata/telegraf</url>`), size of the package, its built time, location of the package inside the repository (`<location href="influxdb/telegraf-1.8.2-1.x86_64.rpm"/>`), package dependencies (inside the `<rpm:requires>` tags), etc.

#### other files

Contains the changelog entries found in the [RPM SPEC file](rpm-package.md) for each package in the repository.

#### repomd.xml

The `repomd.xml` file is essentially an index that contains the location, checksums, and timestamp of the other XML metadata files described above.

```bash
$ cat /opt/local-mirror/repodata/repomd.xml

...
<data type="filelists">
  <checksum type="sha256">8abfa32cc85b5d21d9216e7b322d52ce8af2919438d5ddce6a5c3f69173914c1</checksum>
  <open-checksum type="sha256">b0f139c3e4d859928fb6671b43bde32e114702571e51af8ee933a880e423e26e</open-checksum>
  <location href="repodata/8abfa32cc85b5d21d9216e7b322d52ce8af2919438d5ddce6a5c3f69173914c1-filelists.xml.gz"/>
  <timestamp>1540164243</timestamp>
  <size>1091</size>
  <open-size>6059</open-size>
</data>
<data type="primary">
  <checksum type="sha256">762758eb22f55a22f8247a0f8275b7d0998197c7a368419a31bfa9b992be2dc1</checksum>
  <open-checksum type="sha256">868050dc2878db3eb9c9c688860b5878afde2005e3064d13e9531deef3fb027e</open-checksum>
  <location href="repodata/762758eb22f55a22f8247a0f8275b7d0998197c7a368419a31bfa9b992be2dc1-primary.xml.gz"/>
  <timestamp>1540164243</timestamp>
  <size>1453</size>
  <open-size>6280</open-size>
</data>
...
```

As you can see it lists the details about all the other repository metadata files (files inside the `repodata` directory).

#### How to enable a local repo?

To conclude our task of creating a local clone of a remote package repository, we need to define yum configuration for this local repo before we can start using it.

```bash
$ cat <<EOF | tee /etc/yum.repos.d/local-mirror.repo
[local-influxdb]
name = InfluxDB Repo Local Mirror
baseurl = file:///opt/local-mirror
enabled = 1
gpgcheck = 0
EOF
```

Now let's see what packages are available in that repository using `yum` command:

```bash
$ yum --disablerepo "*" --enablerepo "local-influxdb" list available
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Available Packages
chronograf.x86_64                 1.6.2-1                     local-influxdb
influxdb.x86_64                   1.6.4-1                     local-influxdb
kapacitor.x86_64                  1.5.1-1                     local-influxdb
telegraf.x86_64                   1.8.2-1                     local-influxdb
```

### Resources used to create this document:

* https://access.redhat.com/sites/default/files/attachments/rpm_tutorial_20120831.pdf
* https://access.redhat.com/blogs/766093/posts/1976693
* https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-setting_repository_options
* https://linux.die.net/man/5/yum.conf
* https://www.slashroot.in/yum-repository-and-package-management-complete-tutorial
* https://blog.packagecloud.io/eng/2015/07/20/yum-repository-internals/
