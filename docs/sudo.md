## sudo

The **sudo** command (stands for "superuser do") allows a trusted user to run commands as another user.

Most of the times, it's used to run commands as _root user_. That's why it's the default behaviour of the `sudo` command and it doesn't require you to specify any additional arguments such as a root user name.

For example, see how running the `whoami` command (which shows the user name associated with the current account) with `sudo` changes its output:

```bash
$ whoami
vagrant
$ sudo whoami
root
```

[root](http://www.linfo.org/root.html) is the user name or account that by default has access to all commands and files on a Linux or other Unix-like operating system. It is also referred to as the **root account** and the **superuser**.

Note that `sudo` can also be used to run commands as another user other than root, but that user have to be explicitly specified with the **-u** option:

```bash
$ whoami
vagrant
$ sudo -u test-user whoami
test-user
```

### sudo vs su

When switching a user with [su](su.md), you're asked to enter the password for that user. If multiple system users need to have access to the same account such as root account, you're facing the problem of sharing passwords among multiple people which entails a whole lot problems (e.g. syncronizing the update of the password, making sure it stays secret, etc.).

_The **sudo** command offers a mechanism for providing trusted users with another user account access to a system without sharing the password of that user account._

If `sudo` is configured to use password authentication, then when you run a command with `sudo` you will be asked to type in the password of _your_ user and not the password of another user.

### Configuration file

`sudo` uses **/etc/sudoers** configuration file. This file is used to control the behavior of the `sudo` command and specify a list of trusted users who can use `sudo`.

Changing this file can be pretty risky, that's why by default even the root user doesn't have permissions to change this file:

```bash
$ ls -l /etc/sudoers
-r--r-----. 1 root root 3938 Apr 10  2018 /etc/sudoers
```

The sudoers file is supposed to be edited with a special **visudo** command.

`visudo` allows to edit the sudoers file in a safe fashion. In particular, it locks the sudoers file against multiple simultaneous edits, parses the sudoers file after the edit and will not save the changes if there is a syntax error. To change sudoers file, simply run `visudo` command as a root user. If run without any arguments, `visudo` will automatically open `/etc/sudoers` file for editing.

**Warning:** `sudo` will not work if `/etc/sudoers` contains syntax errors, so you should only ever edit it using `visudo`, which performs basic sanity checks, and installs the new file only if it parses correctly.

#### Defaults

The sudoers file contains a bunch of `Defaults` entries that define the behavior of the `sudo` command.

```bash
$ fgrep -i defaults /etc/sudoers
# Defaults specification
Defaults   !visiblepw
Defaults    always_set_home
Defaults    match_group_by_gid
Defaults    env_reset
Defaults    env_keep =  "COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS"
Defaults    env_keep += "MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
# Defaults   env_keep += "HOME"
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin
```

Let's explain some of these defaults:

* [Defaults always_set_home](https://www.sudo.ws/man/sudoers.man.html#x5355444f455253204f5054494f4e53): will set the HOME environment variable to the home directory of the target user (which is root unless the -u option is used).
* [Defaults env_reset](https://www.sudo.ws/man/sudoers.man.html#x5355444f455253204f5054494f4e53): resets the environment to remove any [user variables](shell-and-environment-variables.md). This is a safety measure used to clear potentially harmful environment variables from the sudo session. The new environment contains the TERM, PATH, HOME, MAIL, SHELL, LOGNAME, USER, USERNAME and SUDO_* variables in addition to variables from the invoking process permitted by the env_check and env_keep options.
* [Defaults env_keep](https://www.sudo.ws/man/sudoers.man.html#x5355444f455253204f5054494f4e53): Environment variables to be preserved in the user's environment when the env_reset option is in effect. The argument may be a double-quoted, space-separated list or a single value without double-quotes. The list can be replaced, added to, deleted from, or disabled by using the =, +=, -=, and ! operators respectively.
* [Defaults secure_path](https://www.sudo.ws/man/sudoers.man.html#x5355444f455253204f5054494f4e53): specifies the PATH environment variable that will be used for sudo operations. This prevents using user paths which may be harmful.

Some other useful defaults options include.

##### Lecture sudo users

```
Defaults  lecture="always"
```

This option enables a short lecture message to be printed when calling sudo command. For example:

```bash
$ sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.
...
```

To customize the lecture message, see the [lecture_file option](https://www.sudo.ws/man/1.8.25/sudoers.man.html).

##### Specify number of sudo authentication attempts

The [passwd_tries](https://www.sudo.ws/man/1.8.25/sudoers.man.html) allows to specify the number of tries a user gets to enter his password before sudo logs the failure and exits. The default is `3`.

```bash
Defaults   passwd_tries=2
```

The above option, will make sudo command fail if a user fails to enter his password `2` times:

```bash
$ sudo -l
...
[sudo] password for test-user:
Sorry, try again.
[sudo] password for test-user:
sudo: 2 incorrect password attempts
```

##### Set password timeout

The [timestamp_timeout](https://www.sudo.ws/man/1.8.25/sudoers.man.html) allows to specify the number of minutes that can elapse before `sudo` will ask user for a password again. The default is `5`. If set to `0`, `sudo` will always prompt for a password. If set to a value less than `0` the user's time stamp will not expire until the system is rebooted.

By default, sudo will ask you for your password after `5` minutes of not using it:

```bash
$ whoami
test-user2
$ date; sudo ls /root
Sun Nov 18 04:10:57 UTC 2018
[sudo] password for test-user2:
anaconda-ks.cfg  file1.txt	  script.sh  test1.txt	test.sh
## run a command one minute after the last sudo command
$ date; sudo ls /root
Sun Nov 18 04:12:06 UTC 2018
anaconda-ks.cfg  file1.txt	  script.sh  test1.txt	test.sh
## run a command five minutes after the last sudo command
$ date; sudo ls /root
Sun Nov 18 04:17:19 UTC 2018
[sudo] password for test-user2:
anaconda-ks.cfg  file1.txt	  script.sh  test1.txt	test.sh
```

As you can see from this example, it didn't ask me for a password when I ran sudo after one minute of the last sudo command, but it did ask me for a password again when I didn't use sudo for more than 5 minutes.

##### Log sudo commands to a file

By default, sudo logs via [syslog](syslog.md). By using the [logfile option](https://www.sudo.ws/man/1.8.25/sudoers.man.html), you can specify a path to the log file to which sudo will log the activity:

```bash
Defaults  log_host, log_year, logfile="/var/log/sudo.log"
```

After this option is enabled, I can see the logs of sudo command usage in `/var/log/sudo.log` file:

```bash
$ sudo cat /var/log/sudo.log
Nov 13 07:52:40 2018 : test-user : HOST=localhost : 2 incorrect password
    attempts ; TTY=pts/0 ; PWD=/home/vagrant ; USER=root ; COMMAND=list
Nov 13 07:52:54 2018 : vagrant : HOST=localhost : TTY=pts/0 ; PWD=/home/vagrant
    ; USER=root ; COMMAND=/bin/cat /var/log/sudo.log
```

#### User privileges

Configuration lines that give system users sudo permissions have the following syntax:

```bash
user host=(run-as) command
```

Where:

* `user`: specifies a trusted user that is allowed to use `sudo` command;
* `host`: specifies a hostname that this line applies to, i.e. only on the host whose hostname matches the one specified in this line the user will be able to run `sudo`;
* `run-as`: specifies the user (and possibly group) to run the command as;
* `command`: specifies the command that is allowed to be run with `sudo`.

This whole line can be read as "User `user` may run command `command` as the `run-as` user on host `host`".

For example, the following line in a sudoers file will allow user named `test-user` run [touch](touch.md) command on any host as a vagrant user:

```
test-user ALL=(vagrant) /bin/touch
```

To verify that test-user indeed can run touch command (and only this command) as a vagrant user, I can run the following test:

```bash
$ whoami
test-user
$ sudo -u vagrant touch /tmp/file1.txt
$ ls -l /tmp/file1.txt   # check the owner and the group of the file
-rw-r--r--. 1 vagrant vagrant 0 Nov 14 05:57 /tmp/file1.txt
$ sudo -u vagrant mkdir /tmp/dir1  # try to run a different command
Sorry, user test-user is not allowed to execute '/bin/mkdir /tmp/dir1' as vagrant on localhost.localdomain.
```

The word **ALL** that was specified in place of a hostname means _any_. It's quite often used when giving sudo permissions.

For example, in your sudoers file you will likely to have the following line:

```bash
root    ALL=(ALL)       ALL
```

This allows the root user to run any command on any host (which has this sudo configuration) as any user.

##### User specifications

[user](#user-privileges) and [run-as-user](#user-privileges) may be not only be user names, but you can also specify group names (must be prefixed with `%`), numeric UIDs (must be prefixed with `#`), or numeric GIDs (must be prefixed with `%#`).

For example, the following command allows users in the `wheel` group to use `sudo`:

```bash
%wheel  ALL=(ALL)       ALL
```

##### Run-as specifications

If the optional [run-as](#user-privileges) clause is omitted, the user will be permitted to run commands only as root.

For example, let's change the previous sudo configuration for the test-user to the following:

```bash
test-user ALL=/bin/touch
```

And run the test [we ran previously](#user-privileges):

```bash
$ whoami
test-user
$ sudo -u vagrant touch /tmp/file1.txt  # run command as vagrant user
Sorry, user test-user is not allowed to execute '/bin/touch /tmp/file1.txt' as vagrant on localhost.localdomain.
$ sudo touch /tmp/file1.txt  # run command as root user
$ ls -l /tmp/file1.txt  # check file owner and group
-rw-r--r--. 1 root root 0 Nov 14 06:24 /tmp/file1.txt
```

You may also see sometimes the [run-as](#user-privileges) specified as **(ALL:ALL)** or `(some-user-name:some-group-name)`. For example:

```
test-user ALL=(ALL:ALL) /bin/touch
```

This enables the test-user to use not only the `-u` option (which allows to run commands as another user), but also the `-g` option (which runs commands with the primary group set to a specified group instead of the primary group specified by the target user's password database entry).

Let's run a few commands to see how it works:

```bash
$ whoami
test-user
$ sudo -u vagrant touch /tmp/file1.txt  # run command as vagrant user
$ ls -l /tmp/file1.txt  # check file owner and group
-rw-r--r--. 1 vagrant vagrant 0 Nov 14 06:41 /tmp/file1.txt
$ sudo -g vagrant touch /tmp/file2.txt # run command with a primary group set to vagrant
$ ls -l /tmp/file2.txt  # check file owner and group
-rw-r--r--. 1 test-user vagrant 0 Nov 14 06:42 /tmp/file2.txt
$ sudo -u root -g vagrant touch /tmp/file3.txt   # run command as root user with a primary group set to vagrant
$ ls -l /tmp/file3.txt  # check file owner and group
-rw-r--r--. 1 root vagrant 0 Nov 14 06:42 /tmp/file3.txt
```

Note, If you specify only a target group in the sudoers file (e.g. `(:vagrant)`), `sudo` will accept and act on `-g vagrant` but run commands only as the invoking user, i.e. you won't be able to run commands as root or another user using the `-u` option.

##### Command specifications

In the simplest case, a command is the full path to an executable, which permits it to be executed with any arguments. You may specify a list of arguments after the path to permit the command only with those exact arguments, or write "" to permit execution only without any arguments.

A command may also be the full path to a directory (including a trailing /). This permits execution of all the files in that directory, but not in any subdirectories.

If multiple commands need to be specified, simply separate them with commas.

```
test-user ALL=/bin/ls, /bin/df -h /, /bin/date "", /usr/bin/, sudoedit /etc/hosts
```

The keyword [sudoedit](http://www.wingtiplabs.com/blog/posts/2013/03/13/sudoedit/) is recognised as a command name, and arguments can be specified as with other commands. [Use it instead of allowing a particular editor to be run with sudo](https://www.howtoforge.com/tutorial/how-to-let-users-securely-edit-files-using-sudoedit/), because it runs the editor as the user that issued the command and only installs the editor's output file into place as root (or other target user).

##### Command tags

Before the command, you can specify zero or more **command tags** to control how it will be executed. The most commonly used tags are:

* `PASSWD` and `NOPASSWD`: by default, `sudo` requires a user to authenticate himself by entering his password before running a command. This behavior can be modified via the `NOPASSWD` tag.

    For example:

    ```
    test-user ALL=(ALL) NOPASSWD: /bin/touch, /bin/ls
    ```

    This configuration will allow test-user to run [touch](touch.md) and [ls](ls.md) commands as any user on any host without entering a password.

    Conversely, the `PASSWD` tag can be used to require user authentication for certain commands. For example:

    ```
    test-user ALL=(ALL) NOPASSWD: /bin/touch, /bin/ls, PASSWD: /bin/mkdir
    ```
    
    Similar to example before, this allows test-user to run [touch](touch.md) and [ls](ls.md) commands without a password, and it also allows to run `mkdir` command if a user's able to authenticate himself.

    ```bash
    $ whoami
    test-user
    $ sudo touch /tmp/test1.txt  # run command as root user
    $ ls -l /tmp/test1.txt  # check owner and group
    -rw-r--r--. 1 root root 0 Nov 14 18:46 /tmp/test1.txt
    $ sudo mkdir /tmp/test-dir  # run command as root user
    [sudo] password for test-user:
    $ ls -ld /tmp/test-dir/  # check owner and group
    drwxr-xr-x. 2 root root 6 Nov 14 18:47 /tmp/test-dir/
    ```

* `EXEC` and `NOEXEC`. The `NOEXEC` tag can be used to prevent **shell escapes**, i.e. situations when an executable can further run commands itself ([see an example of a shell escape using vi editor](https://www.howtoforge.com/tutorial/how-to-let-users-securely-edit-files-using-sudoedit/))

* `SETENV` and `NOSETENV`. The `SETENV` tag allows the user to set environment variables for the command. Note that if `SETENV` has been set for a command, the user may disable the `env_reset` option from the command line via the `-E` option. Additionally, environment variables set on the command line are not subject to the restrictions imposed by `env_check`, `env_delete`, or `env_keep`. If the command matched is `ALL`, the `SETENV` tag is implied for that command; this default may be overridden by use of the NOSETENV tag.

#### Aliases

The sudoers file can be better organized by using `aliases`.

We've already mentioned, that giving a user sudo privileges has the following format:

```
user host=(run-as) command
```

Each of variables involved here can be a list of values. For example, we can allow a user to run a list of commands by separating them with a comma:

```bash
test-user ALL=(ALL) NOPASSWD: /bin/touch, /bin/ls
```

or we can give the same type of sudo access to a list of users:

```bash
test-user, test-user2 ALL=(ALL) NOPASSWD: /bin/touch, /bin/ls
```

These lists of values can be grouped and assigned to a variable. These variables are called **aliases**.

There are four kinds of aliases: 

* `User_Alias`: used to specify a list of trusted users;
* `Runas_Alias`: used to specify a list of users to run commands as;
* `Host_Alias`: used to specify a list of hosts;
* `Cmnd_Alias`: used to specify a list of commands. Note, you cannot include [command tags](#command-tags) like `NOPASSWD:` in command aliases.

Each alias definition has the following form:

```
Alias_Type NAME = item1, item2, ...
```

Note, that aliases are always named in uppercase.

Here is an example of alias definitions that can be defined in a sudoers file:

```bash
User_Alias TRUSTED = %admin, !ams
Runas_Alias LEGACYUSERS = oldapp1, oldapp2
Runas_Alias APPUSERS = app1, app2, LEGACYUSERS
Host_Alias PRODUCTION = www1, www2, \
    192.0.2.1/24, !192.0.2.222
Cmnd_Alias DBA = /usr/pgsql-9.4/bin, \
    /usr/local/bin/pgadmin
```

Any term in a list may be prefixed with `!` to negate it. This can be used to include a group but exclude a certain user, or to exclude certain addresses in a network, and so on. Negation can also be used in command lists, but note the manpage's warning that trying to “subtract” commands from `ALL` using `!` is generally not effective.

_Use aliases whenever you need rules involving multiple users, hosts, or commands._

As an example, let's group lists of values from the previous example into aliases:

```bash
User_Alias TESTUSERS = test-user, test-user2
Cmnd_Alias TESTCOMMANDS = /bin/touch, /bin/ls

TESTUSERS ALL=(ALL) NOPASSWD: TESTCOMMANDS
```

### How to give a user sudo access

#### Simple case

For a simple case, when you need to give a specific user `sudo` access, you can add them to a _general purpose admin group_. For example, on Ubuntu this group is called the **sudo group**, on RedHat based distros it's called the **wheel group**.

By default, in the sudoers file, this general purpose admin group is given privileges to run any commands with `sudo`. For example, on my default Centos 7 installation I have the following lines in the sudoers file:

```bash
$ fgrep wheel /etc/sudoers
## Allows people in group wheel to run all commands
%wheel	ALL=(ALL)	ALL
```

To add user to this group, I can run the following command:

```bash
$ sudo usermod -aG wheel test-user3  # add user to the group
$ sudo groups test-user3  # check what groups this user is part of
test-user3 : test-user3 wheel
```

After this the `test-user3` will be allowed to use `sudo` to run any commands as any user.

#### Advanced case

In a more complex senario, if you need to give a fine-grained access to certain commands to certain users, you could possilbly edit the sudoers file with visudo commands, but there is a better way of doing this.

The sudoers file has an interesting line at the end of the file:

```bash
$ tail -1 /etc/sudoers
#includedir /etc/sudoers.d
```

Although, it does begin with a `#`, which indicates a comment in the sudoers file, this line actually makes `sudo` to read files in the **/etc/sudoers.d** as well as the main sudoers file.

Here are few reasons (taken from [this thread](https://superuser.com/questions/869144/why-does-the-system-have-etc-sudoers-d-how-should-i-edit-it)) why it's better to place files in `/etc/sudoers.d` directory insteand of editing the `/etc/sudoers` file:

*  `/etc/sudoers` is under control of your distribution's package manager. If you have made changes to that file and the package manager may upgrade it. Files in `/etc/sudoers.d` directory will persist during the upgrades.
* the rules on files in `/etc/sudoers.d` directory seem a bit looser than for `/etc/sudoers`. For example, mistakes in the file inside `/etc/sudoers.d` do not cause sudo to fail, howerver the file is ignored (verified myself).
* It's easier for configuration management tools (e.g. Chef or Puppet) to drop individual files into this directory, rather than making changes to `/etc/sudoers`.

Files inside `/etc/sudoers.d` directory follow the same syntax rules as the `/etc/sudoers` file itself (they are basically concatenated to it). And you can use `visudo` command for safer editing files inside `/etc/sudoers.d` as well. Simply specify the path to the file you're adding/changing after the `-f` option:

```bash
$ visudo -f /etc/sudoers.d/test-user
```

Note a few exceptions about naming the files that you put in `/etc/sudoers.d`:

* Files whose names end in `~` are ignored;
* Files whose names contain a `.` character are ingored;

These exceptions allow backup files from editors to be ignored.

### Useful commands

The `sudo` command has the following syntax:

```
sudo [options] command
```

#### Run command as another user

By default, `sudo` will run a command you specify as root user, so you don't need to provide any extra options if you need to run a command as root:

```bash
$ whoami
test-user2
$ sudo whoami
root
```

If you want to run a command as another user other than the root user, you need to pass the **-u** option and the user name:

```bash
$ whoami
test-user2
$ sudo -u vagrant whoami
vagrant
```

#### List user sudo privileges

To list sudo priliveges for the current user, use **-l** or **-ll** option:

```bash
$ sudo -l

User test-user2 may run the following commands on localhost:
    (ALL) ALL

$ sudo -ll

User test-user2 may run the following commands on localhost:
Sudoers entry:
    RunAsUsers: ALL
    Commands:
	ALL
```

To see sudo privileges of another user, specify its name after the **-U** option:

```bash
$ sudo -U vagrant -l

User vagrant may run the following commands on localhost:
    (ALL) NOPASSWD: ALL
```

#### Run the last command with sudo

You may often find yourself in a situation when you try run a command and it fails because you lack privileges. For example, if you try to install a system package, the command will likely to fail:

```bash
$ yum install -y lsof
Loaded plugins: fastestmirror
You need to be root to perform this command.
```

The shortcut for the last entered command is **!!**. You can pass it as a command to sudo:

```bash
$ yum install -y lsof
Loaded plugins: fastestmirror
You need to be root to perform this command.
$ sudo !!
```

#### Run commands in the target user environment

`sudo` command provides a few options which allow not only to run commands as another user, but also use that user's default [shell as a running environment](shell-and-environment-variables.md).

The **-s** option will make `sudo` to run a [non-login shell](shells.md) as the target user. If the option is used and command to run is not provided, it will launch a new [interactive non-login shell](shells.md):

```bash
$ whoami   # check the current user
vagrant
$ echo $$  # check PID of the current shell
2331
$ sudo -s  # start a non-login shell of root user
$ whoami
root
$ echo $$  # check PID of the current shell
2644
$ ps -ef --forest | grep -B3 2644  # check the process tree
vagrant   2330  2327  0 04:42 ?        00:00:00      \_ sshd: vagrant@pts/0
vagrant   2331  2330  0 04:42 pts/0    00:00:00          \_ -bash
root      2643  2331  0 05:17 pts/0    00:00:00              \_ sudo -s
root      2644  2643  0 05:17 pts/0    00:00:00                  \_ /bin/bash
root      2709  2644  0 05:22 pts/0    00:00:00                      \_ ps -ef --forest
```

As you can see, the `sudo -s` command launched a root non-login shell. This means that you can now run any commands in that shell as root without having to type a password.

You can also provide a command to the `sudo -s` command. Your command will run in the target user's non-login shell:

```bash
$ echo $HOME   # check path of the current user home folder
/home/vagrant
$ sudo -s echo \$HOME  # check path of the target user home folder
/root
```

The last command was run in the root shell (I had to escape the shell variable, otherwise [it would expand too early](https://markhneedham.com/blog/2012/07/04/sudo-sudo-i-sudo-su/)).

Because the **-s** option starts a [non-login shell](shells.md), your current working directory won't be changed, and some important variables such as `PATH` won't have the same values as those of the target user:

```bash
$ pwd
/home/vagrant
$ sudo -s pwd
/home/vagrant
$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin
$ sudo -s echo \$PATH
/sbin:/bin:/usr/sbin:/usr/bin
```

To run sudo commands in a [login shell](shells.md), use the **-i** option:

```bash
$ echo $HOME
/home/vagrant
$ sudo -i echo \$HOME
/root
$ pwd
/home/vagrant
$ sudo -i pwd
/root
$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin
$ sudo -i echo \$PATH
/usr/local/sbin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
$ sudo -s echo \$PATH
/sbin:/bin:/usr/sbin:/usr/bin
```

And if run without a command, it will start an [interactive login shell](shells.md):

```bash
$ whoami
vagrant
$ echo $$
2331
$ sudo -i
$ whoami
root
$ echo $$
2798
$ ps -ef --forest | grep -B3 2798
vagrant   2330  2327  0 04:42 ?        00:00:00      \_ sshd: vagrant@pts/0
vagrant   2331  2330  0 04:42 pts/0    00:00:00          \_ -bash
root      2797  2331  0 05:50 pts/0    00:00:00              \_ sudo -i
root      2798  2797  0 05:50 pts/0    00:00:00                  \_ -bash
root      2816  2798  0 05:51 pts/0    00:00:00                      \_ ps -ef --forest
```

The **sudo -i** command is often [the easiest way to swith to a root or any other user](https://www.maketecheasier.com/differences-between-su-sudo-su-sudo-s-sudo-i/).

Another commonly used way to switch to another user is by [running the su command with sudo](https://askubuntu.com/questions/376199/sudo-su-vs-sudo-i-vs-sudo-bin-bash-when-does-it-matter-which-is-used). For example, the following command will achieve [almost the same result](https://serverfault.com/questions/359856/what-is-the-difference-between-sudo-i-and-sudo-su) as the `sudo -i` command:

```bash
$ sudo su -
$ ps -ef --forest | grep -B4 2820
vagrant   2330  2327  0 04:42 ?        00:00:00      \_ sshd: vagrant@pts/0
vagrant   2331  2330  0 04:42 pts/0    00:00:00          \_ -bash
root      2818  2331  0 05:52 pts/0    00:00:00              \_ sudo su -
root      2819  2818  0 05:52 pts/0    00:00:00                  \_ su -
root      2820  2819  0 05:52 pts/0    00:00:00                      \_ -bash
root      2855  2820  0 06:03 pts/0    00:00:00                          \_ ps -ef --forest
```

You may notice that this command has a bit of different process tree. Let's break down this process of switching to the root user:

1. Vagrant user invokes the `sudo su -` command.
2. `su -` is run as root.
3. `su -` tries to switch to the root user by launching a login root shell. The [su](su.md) command normally asks for a password of the target user, but root is allowed to use `su` without a password.
4. `su -` starts a login root shell.

#### Invalidate sudo credentials cache

By default, [sudo will ask you for your password after 5 minutes of not using it](#set-password-timeout). If the password caching is enabled, you can run sudo with the **-k** option to invalidate the cache:

```bash
$ date; sudo touch /tmp/file100.txt
Sun Nov 18 04:54:34 UTC 2018
$ date; sudo ls -l /tmp/file100.txt
Sun Nov 18 04:54:43 UTC 2018
-rw-r--r--. 1 root root 0 Nov 18 04:54 /tmp/file100.txt
$ date; sudo -k
Sun Nov 18 04:54:47 UTC 2018
$ date; sudo ls -l /tmp/file100.txt
Sun Nov 18 04:54:51 UTC 2018
[sudo] password for test-user2:
-rw-r--r--. 1 root root 0 Nov 18 04:54 /tmp/file100.tx
```

As you can see, once I invalidated the sudo password cache, it asked me for a password the next time ran sudo.

### Resources used to create this document:

* https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/2/html/Getting_Started_Guide/ch02s03.html
* http://www.linfo.org/root.html
* https://www.tecmint.com/su-vs-sudo-and-how-to-configure-sudo-in-linux/
* man visudo
* man sudo
* https://www.sudo.ws/man/1.8.25/sudoers.man.html
* http://toroid.org/sudoers-syntax
* http://www.wingtiplabs.com/blog/posts/2013/03/13/sudoedit/
* https://superuser.com/questions/869144/why-does-the-system-have-etc-sudoers-d-how-should-i-edit-it
* https://linux.101hacks.com/unix/sudo/
* https://askubuntu.com/questions/195789/what-are-some-of-the-basic-sudo-commands
* https://bencane.com/2011/08/17/sudo-list-available-commands/
* https://linux.101hacks.com/unix/sudo/
* https://www.maketecheasier.com/differences-between-su-sudo-su-sudo-s-sudo-i/
* https://serverfault.com/questions/359856/what-is-the-difference-between-sudo-i-and-sudo-su
* https://markhneedham.com/blog/2012/07/04/sudo-sudo-i-sudo-su/
* https://askubuntu.com/questions/376199/sudo-su-vs-sudo-i-vs-sudo-bin-bash-when-does-it-matter-which-is-used
