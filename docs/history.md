## Command history

The bash shell stores the history of commands you’ve run in a **history file**. Each user account has its own history file with a separate command history.

The location of this file is controlled by the **HISTFILE** shell variable which is by default set to the **.bash_history** file in the user's home directory:

```bash
$ echo $HISTFILE
/home/vagrant/.bash_history
```

To view a user's command history, you can open the user's history file directly with [less](less.md) or run a [history](#the-history-command) command as that user:

```bash
$ history 4
  346  echo $PROMPT_COMMAND
  347  echo $HISTFILE
  348  echo $HISTSIZE
  349  history 4
```

### Customizing history

You can use a number of [shell variables](shell-and-environment-variables.md) to customize how bash will store the command history for the user. Changing the values of these variables for a single user is done by redefining them in [.bashrc file](shell-and-environment-variables.md) located in the user's home directory (see [examples](https://www.shellhacks.com/tune-command-line-history-bash/)).

#### Increase history size

Bash only remembers a limited number of commands by default, preventing the history file from growing too large. The number of history entries bash remembers is controlled by the [HISTSIZE](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html) variable. The default is usually `500` or `1000` entries:

```bash
$ echo $HISTSIZE
1000
```

#### Store time and date of the commands

It's often useful to also be able to see when certain commands were run, i.e. the exact date and time. To make bash to save that information in the history file, you need to define [HISTTIMEFORMAT varialbe in your ~/.bashrc file](https://howchoo.com/g/yjk5yzm1odz/how-to-display-the-timestamp-in-your-bash-history).

#### Ignore certain commands

The [HISTCONTROL](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html) variable allows to ignore duplicate commands and commands that start with a space. It can be assigned a colon-separated list of the following values:

* `ignorespace`: don’t save commands which begin with a space character
* `ignoredups`: don’t save commands matching the previous command in the history
* `ignoreboth`: use both `ignorespace` and `ignoredups`
* `erasedups`: causes all commands in the history matching the current command to be removed from the history list before the current command is saved.

Consider the following example:

```bash
$ echo $HISTFILE
/home/vagrant/.bash_history
$ echo $HISTSIZE
1000
$ echo $HISTCONTROL
ignoredups
$ echo $HISTCONTROL
ignoredups
$ history 4
  367  echo $HISTFILE
  368  echo $HISTSIZE
  369  echo $HISTCONTROL
  370  history 4
```

The second time I ran `echo $HISTCONTROL` command it wasn't saved because `$HISTCONTROL` states not to save duplicate commands in the history.

If you want to ingore a specific set of commands, you need to list them in the [HISTIGNORE](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html) variable.

`HISTIGNORE` is assigned a colon-separated list of command _patterns_. For example:

```bash
$ HISTIGNORE="ls*:history*"
$ echo $HISTFILE
/home/vagrant/.bash_history
$ ls > /dev/null
$ echo $HISTSIZE
1000
$ history 4
  401  echo $HISTSIZE
  402  HISTIGNORE="ls*:history*"
  403  echo $HISTFILE
  404  echo $HISTSIZE
```

As you can see, after I define `HISTIGNORE` variable, all commands that start with `ls` or `history` do not get saved in the history. Note that in this example I set `HISTIGNORE` for the current shell only ([see how to persist shell variables](shell-and-environment-variables.md)).

### Working with the history

#### The history command

The `history` command can be used to view a list of commands from history:

```bash
$ history
```

The history can be pretty big, so it makes sense to pipe history command output to [less](less.md) or [vim](vim.md) for easier viewing and searching:

```bash
$ history | less
```

Or if you're interested in a few recent commands, you can pass a number as an argument to the history command and it will print only that number of the most recent commands in the history:

```bash
$ history 3  # show 3 last commands
  418  exit
  419  history 7
  420  history 3
```

If you need to search for a certian command in history, you can either pipe history command output into vim or less as I've mentioned before and then use the built-in search mechanism of those tools. Another option would be to directly [grep](grep.md) the history command output for specific command pattern:

```bash
$ history | fgrep usermod
  231  sudo usermod -aG wheel test-user3
  422  history | fgrep usermod
```

#### Scrolling through command history

There are a few ways that you can scroll through your bash history putting each successive command on the command line to edit:

* `UP arrow key`: Scroll one command backwards in history
* `CTRL+p`: Scroll one command backwards in history
* `DOWN arrow key`: Scroll one command forwards in history
* `CTRL+n`: Scroll one command forwards in history

#### Searching through command history

If you press `CTRL+r` in the shell, you activate a feature called **reverse-i-search**. Reverse-i-search'ing lets you search through your command history.

Here is a set of instructions on how to do a reverse-i-search:

1. To start searching, press `CTRL+r`. You will see in the terminal that reverse-i-search feature was activated:
    ```bash
    (reverse-i-search)`':
    ```

2. Then type the part of the command you are looking for. It will suggest you the most recent command that matches what you typed:

    ```bash
    (reverse-i-search)`usermod': history | fgrep usermod
    ```

3. If the first result isn't what you want, press `CTRL+r` again to see the next most recent command in history that matches your search:

    ```bash
    (reverse-i-search)`usermod': sudo usermod -aG wheel test-user3
    ```

    Note: sometimes I have to press the `Delete` key to see the next match, whereas the `CTRL+r` won't let me. To be honest, I'm not sure why.

    P.S. just to see where in the history these two commands are located (i.e. their line numbers in the history file), I ran the following command:

    ```bash
    $ history | fgrep usermod
      231  sudo usermod -aG wheel test-user3
      422  history | fgrep usermod
    ```

4. If you found the command that you want:
    * press `Enter` to execute the command
    * use the right or left arrow keys to place the command on an actual command-line so you can edit it.

   If you didn't find the command:
    * you can delete characters that you typed in the search and type new ones to search for a different pattern
    * you can exit the search by pressing `CTRL+g`

**NOTE:** If you feel the command will be used frequently, [you can tag that command by adding](https://unix.stackexchange.com/a/166927) a comment after the `#` sign:

```bash
(reverse-i-search)`# user2': sudo -i -u test-user2 # user2
```

#### History expansions

**History expansions** introduce words from the history list into the input stream, making it easy to repeat commands, insert the arguments to a previous command into the current input line, or fix errors in previous commands quickly.

`History expansion` takes place in two parts:

* The first is to determine which line from the history list should be used during substitution. The line selected from the history is called the **event**. You specify the event (i.e. the line in this history list) using [event designators](https://www.gnu.org/software/bash/manual/html_node/Event-Designators.html#Event-Designators).
* The second is to select portions of that line for inclusion into the current one. The portions of that line are called **words**. [Word designators](https://www.gnu.org/software/bash/manual/html_node/Word-Designators.html#Word-Designators) are used to select desired words from the event. Besides, various [modifiers](https://www.gnu.org/software/bash/manual/html_node/Modifiers.html#Modifiers) can be used to manipulate the selected words.

Expressions used for history expansions start with the **history expansion character**, which is **!** by default. Only `\` and `'` may be used to escape the history expansion character.

Below I will list a few examples of using history expansions. Note, if you want to learn more about history expansions, you can read the documentation on [event designators]((https://www.gnu.org/software/bash/manual/html_node/Event-Designators.html#Event-Designators)), [word designators](https://www.gnu.org/software/bash/manual/html_node/Word-Designators.html#Word-Designators) and [modifiers](https://www.gnu.org/software/bash/manual/html_node/Modifiers.html#Modifiers). Also, here are a few posts ([1](https://www.thegeekstuff.com/2011/08/bash-history-expansion/), [2](https://www.digitalocean.com/community/tutorials/how-to-use-bash-history-commands-and-expansions-on-a-linux-vps), [3](https://sanctum.geek.nz/arabesque/bash-history-expansion/)) which describe some practical examples of using history expansions.

##### Rerun the previous commands

The double exlamation marks sequence (`!!`) refers to the last run command:

```bash
$ yum install net-tools
Loaded plugins: fastestmirror
You need to be root to perform this command.
$ sudo !!
sudo yum install net-tools
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.keystealth.org
 * extras: mirror.keystealth.org
 * updates: mirror.keystealth.org
...
```

Bash also allows you to rerun the previous command and specify something that should be changed. This can be useful for correcting a typo in the last command. For example, the following command will rerun the previous command, replacing the text `abc` in it with the text `xyz`:

```
^abc^xyz
```

Consider the following examples of rerunning comming while fixing misspellings:

```bash
$ cat /etc/hosst
cat: /etc/hosst: No such file or directory
$ ^hosst^hosts
cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
$ systemclt status sshd
-bash: systemclt: command not found
$ ^clt^ctl
systemctl status sshd
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2018-12-03 02:36:53 UTC; 18min ago
```

##### Rerun a particular line from history

`!n` refers to the n-th command in history list:

```bash
$ history 7
    7  ls /tmp/file1.txt
    8  ls -l /tmp/file1.txt
    9  chmod 0444 /tmp/file1.txt
   10  history
   11  history -d 9
   12  history
   13  history 7
$ !8  # run 8th line from the history command list
ls -l /tmp/file1.txt
-r--r--r--. 1 vagrant vagrant 0 Dec  3 02:41 /tmp/file1.txt
```

##### Reuse arguments from history

[Word designators](https://www.gnu.org/software/bash/manual/html_node/Word-Designators.html#Word-Designators) are very helpful when you want to type a new command, but use the argument(s) from one of the previous commands.

A word desginator is used to select a portion of the command from history. It does this by dividing the command into "words", which are defined as any set of characters separated by a whitespace. Words are numbered from the beginning of the line, with the first word being denoted by `0`. Words are inserted into the current line separated by single spaces.

A word designator follows the [event designator](https://www.gnu.org/software/bash/manual/html_node/Event-Designators.html#Event-Designators) and is separated from it by a colon (`:`).

For instance, we could create a file and then decide to long list that file using history expansions:

```bash
$ touch /tmp/file2.txt
$ ls -l !!:1
ls -l /tmp/file2.txt
-rw-rw-r--. 1 vagrant vagrant 0 Dec  3 04:10 /tmp/file2.txt
```

The `!!:1` expansion consists of two parts separated by colon:

* history expansion character (the first `!`) plus event designator which specifies the last run command (the second `!`)
* word designator `1` which refers to the first word of the command (remember word count starts from `0`)

**Note 1:** the colon (`:`) may be omitted if the word designator begins with a [^ (first argument, i.e. word 1), $ (the last word), * (all of the words, except the 0th), - or %](https://www.gnu.org/software/bash/manual/html_node/Word-Designators.html#Word-Designators).

```bash
$ touch /tmp/file2.txt
$ ls -l !!$
ls -l /tmp/file2.txt
-rw-rw-r--. 1 vagrant vagrant 0 Dec  3 04:10 /tmp/file2.txt
```

**Note 2:** Plus, if a word designator is supplied without an event specification, the last run command is used as the event:

```bash
$ touch /tmp/file2.txt
$ ls -l !$
ls -l /tmp/file2.txt
-rw-rw-r--. 1 vagrant vagrant 0 Dec  3 04:10 /tmp/file2.txt
```

You can also change the "words" using [modifiers](https://www.gnu.org/software/bash/manual/html_node/Modifiers.html#Modifiers):

```bash
$ pwd
/home/vagrant
$ touch /tmp/file3.txt
$ cd !$:h   # will remove a trailing pathname component, leaving only the directory name
cd /tmp
$ pwd
/tmp
$ ls -l !touch:$:t  # will remove all leading pathname components leaving only the filename
ls -l file3.txt
-rw-rw-r--. 1 vagrant vagrant 0 Dec  3 04:33 file3.txt
```

#### Clearing history

To delete a specific line from history list, use the **-d** option:

```bash
$ history 6
  113  mkdir testdir
  114  touch testdir/file.txt
  115  ls -ld testdir
  116  ld -l testdir/file.txt
  117  ls -l testdir/file.txt
  118  history 6
$ history -d 116  # remove misspelled command
$ history 6
  114  touch testdir/file.txt
  115  ls -ld testdir
  116  ls -l testdir/file.txt
  117  history 6
  118  history -d 116
  119  history 6
```

And the following example shows how to run a single command without saving it to history:

```bash
$ history 2
  119  history 6
  120  history 2
$ echo "non-secret command"
non-secret command
$ echo "secret command"; history -d $(history 1)
secret command
$ history 2
  121  echo "non-secret command"
  122  history 2
```

Additionally, if you want to run multiple commands without saving them to history, you can unset the history file variable (`$HISTFILE`) for the current bash session which will prevent all history for the current session from being saved:

```bash
$ echo $HISTFILE
/home/vagrant/.bash_history
$ unset HISTFILE
$ echo $HISTFILE

```

The history is saved into a file upon a logout, i.e. when a shell with history enabled exits, the last `$HISTSIZE` lines are copied from the history list to the file named by `$HISTFILE`. This means, that you can also run the above command of unsetting the `$HISTFILE` variable after you're done with running all the commands and are about to exit the shell session:

```bash
$ unset HISTFILE && exit  # these will prevent saving history for the current shell session
```

#### View history of another user

To view command history of another user, you can start a login shell for that user (using commands like [sudo](sudo.md) and [su](su.md)) and then run the `history` command:

```bash
$ sudo -i -u test-user  # switch to the test-user
$ history | less
```

Another option would be to open for reading that user's history file:

```bash
$ sudo less -N /home/test-user/.bash_history
```

**Note:** if you want to check someone's command history, you also have to be aware of the fact that that user could've spawned a shell for another user and run commands as that user. For example, many people use commands like `sudo -i` or `sudo su -` to start a root login shell in order to be able run multiple commands as root without being asked for password. In this case, you may also want to check the shell history of the root user.

### Resources used to create this document:

* https://www.gnu.org/software/bash/manual/html_node/Bash-History-Facilities.html#Bash-History-Facilities
* https://www.gnu.org/software/bash/manual/html_node/History-Interaction.html#History-Interaction
* https://www.shellhacks.com/tune-command-line-history-bash/
* https://www.howtogeek.com/howto/44997/how-to-use-bash-history-to-improve-your-command-line-productivity/
* https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html
* https://howchoo.com/g/yjk5yzm1odz/how-to-display-the-timestamp-in-your-bash-history
* https://www.tldp.org/LDP/GNU-Linux-Tools-Summary/html/x1712.htm
* https://www.rootusers.com/17-bash-history-command-examples-in-linux/
* https://unix.stackexchange.com/a/166927
* http://notes.jerzygangi.com/reverse-i-search-in-the-linux-shell/
* https://unix.stackexchange.com/questions/42930/how-to-exit-bash-history-search-mode
* https://www.thegeekstuff.com/2011/08/bash-history-expansion/
* https://www.digitalocean.com/community/tutorials/how-to-use-bash-history-commands-and-expansions-on-a-linux-vps
* https://sanctum.geek.nz/arabesque/bash-history-expansion/
* https://sanctum.geek.nz/arabesque/better-bash-history/
