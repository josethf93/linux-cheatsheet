## hash command

When you run a command without specifying a path to the executable file, the bash shell searches for the command executable in the directories listed in the **PATH** [environment variable](shell-and-environment-variables.md). When the shell finds the command executable, it remembers its path storing it in a **hash table** (in-memory table which stores paths of executables of commands used in the current shell). The next time your run a command in the same shell, bash will first check the `hash table` for the location of the command executable file instead of searching for it again in the directories listed in `PATH`, making commands run faster.

The `hash table` can be manipulated with the **hash command** which is a [shell built-in](shell-builtins.md). For example, to view the `hash table` for your current shell, simply run `hash` command:

```bash
$ hash
hits	command
   1	/usr/bin/mkdir
   2	/usr/bin/ls
   1	/usr/sbin/ip
```

The hash table has 2 columns:

* `hits` : shows how many times a particular executable has been called
* `command`: shows the path to the executable file

Here is a couple of use cases when knowledge of hash tables and the hash command may come in handy.

### Use cases

#### Executable file is moved after bash has recorded it

If the executable file of a command is moved after bash has recorded its location in a hash table, the next time you run that command, shell won't be able to find it because it will use the old path from the hash table.

Let's consider the following example. I have placed a simple bash script (executable file) in the `bin/` directory in my user's home directory:

```bash
$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin
$ cat /home/vagrant/bin/hello
#!/bin/bash
echo "hello world"
```

If I run a `hello` command, shell will first try to find a path to the command executable in its hash table and if can't find it there, it will search directories listednin my `PATH` variable. If it finds it in one of the directories listed in the `PATH`, it will record command executable path to a hash table:

```bash
$ hello
hello world
$ hash | fgrep hello
   1	/home/vagrant/bin/hello
```

Now, let's see what happens if I decide to move my script to another directory (also listed in `PATH`) and run it again:

```bash
$ sudo mv ~/bin/hello /usr/local/bin
$ hello
-bash: /home/vagrant/bin/hello: No such file or directory
```

As you can see, the command failed because bash tried to find the executable in the old location which is recorded in its hash table.

To fix this problem, we need to clear or reset the hash table for the _current shell_. And there is a couple of ways you can do it.

First, you can use one of the `hash` commands:

```bash
$ hash -d hello # remove a single entry from a hash table
$ hash hello    # tell shell to update path for the command in a hash table
$ hash -r       # clear a hash table completely
```

Secondly, you can update the `PATH` variable. Hash table gets cleared when the `PATH` gets updated:

```bash
$ hash
hits	command
   2	/usr/bin/sudo
   0	/usr/local/bin/hello
$ export PATH=$PATH
$ hash
hash: hash table empty
```

#### Remember executables outside of PATH

You can store executables stored outside of directories listed in `PATH` in a hash table. This might be useful if you just have a single executable in a directory outside of `PATH` that you want to run by typing the command name instead of the full path of the executable file.

The `hash -p` command allows to add a mapping of a command to its executable file path and has the following syntax:

```bash
$ hash -p <path-to-executable> <command-name>
```

Consider the following example of adding a command whose executable is stored outside of `PATH` to a hash table:

```bash
$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin
$ cat /tmp/hello
#!/bin/bash
echo "hello world"
$ hash -r                   # reset hash table
$ hash
hash: hash table empty
$ hello
-bash: hello: command not found  # command fails because /tmp directory is not in the PATH
$ hash -p /tmp/hello hello  # add command to the hash table for the current shell
$ hash                      # verify that the command has been added to the hash table
hits	command
   0	/tmp/hello
$ hello                    # command works now, because its executable path is stored in the hash table
hello world
```

Note: when you manually add a command to a hash table this way, you can run the command by name (instead of specifying a full path to executable) only in the _current shell_ and only as long as this entry is present in the hash table.

### Resources used to create this document:

* https://unix.stackexchange.com/questions/86012/what-is-the-purpose-of-the-hash-command
* http://www.informit.com/articles/article.aspx?p=169504&seqNum=8
