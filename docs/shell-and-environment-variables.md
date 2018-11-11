## Shell and Environment variables

### Shell variables

The shell can store temporary variables, called **shell variables**, containing the values of text strings. Shell
variables are very useful for keeping track of values in scripts, and some shell variables control the way the
shell behaves. (For example, the bash shell reads the `PS1` variable before displaying the prompt.)

To assign a value to a shell variable, use the equal sign (`=`):

```bash
$ VAR1='hello world'
```

To access the value of a shell variable, prepend the name of the variable with a dollar sign (`$`), e.g.:

```bash
$ echo $VAR1
hello world
```

Another way to access a shell variables has the following syntax:

```bash
$ echo ${VAR1}
hello world
```

In particular, the latter method allows to interpolate the value of a variable if it is part of word:

```bash
$ VAR2=hello
$ echo "output: ${VAR2}_world"
output: hello_world
$ echo "output: $VAR2_world"
output:
```

### Environment variables

An **environment variable** is like a [shell variable](#shell-variables), but it has a bigger scope. To create an environment variable, you need to use and **export** command before the variable assignment:

```bash
$ export VAR3='hola mundo'
```

The environment variables are accessed the same way as shell variables, i.e.:

```bash
$ echo $VAR3
hola mundo
$ echo ${VAR3}
hola mundo
```

### Shell vs Environment variables

I honestly feel a bit confused by the difference between shell and environement variables to give any definition here.

I saw several times explanations that sounded like ["shell variables are specific to the shell where they were defined, whereas environment variables are not specific to the shell and can be inherited by any child shells or processes"](https://www.digitalocean.com/community/tutorials/how-to-read-and-set-environmental-and-shell-variables-on-a-linux-vps). But at the same, time I observed that shell variables are [visible in the subshells](https://unix.stackexchange.com/questions/138463/do-parentheses-really-put-the-command-in-a-subshell), which kind of contradicts that definition.

So instead of givining any definitions here, I'll describe here an example of when the difference between shell and environment variables seems obvious and significant.

_Example:_ let's define a shell variable first:

```bash
$ VAR4='hola mundo'
```

The variable is visible in the current shell:

```bash
$ echo $$ # check the PID of the current shell
14887
$ echo "output: $VAR4"
output: hola mundo
```

Now I will start a new shell from the current shell by running `/bin/bash` and then will check the variable again:

```bash
$ /bin/bash # start a new shell
$ echo $$ # check the PID of a new shell
14911
$ echo "output: $VAR4"
output:
```

The variable that I defined in the shell with PID=14887 is not visible in the new shell. Now I will exit from this shell and return to my original shell:

```bash
$ exit
exit
$ echo $$
14887
$ echo "output: $VAR4"
output: hola mundo
```

Let's also see if the shell variable will be visible inside the script that I launch from this shell:

```bash
$ cat <<'EOF' > test.sh
#!/bin/bash
echo "shell PID: $$"
echo "output from script: $VAR4"
EOF
$ chmod +x test.sh
$ ./test.sh
shell PID: 14930
output from script:
```

The variable is not visible inside the script either.

Now let's how an environment variable will behave in the same cases:

```bash
$ export VAR4='hola mundo'
$ echo $$ # check the PID of the current shell
14948
$ echo "output: $VAR4"
output: hola mundo
$ /bin/bash # start a new shell
$ echo $$ # check the PID of a new shell
14968
$ echo "output: $VAR4"
output: hola mundo
$ exit # return to the original shell
exit
$ echo $$
14948
$ ./test.sh
shell PID: 14981
output from script: hola mundo
```

As you can see the environment variable was visible in both of those cases that we've tested. 

_Thus, we can make conclusion that, unlike environment variables, shell variables are not visible in the programs and scripts that you launch._ However, shell variables are [visible in the subshells](https://unix.stackexchange.com/questions/138463/do-parentheses-really-put-the-command-in-a-subshell).

### Variables in scripts

Just a few notes about setting variables in scripts...

#### Running scripts in a separate shell/process

If you set a shell or environment variable in a script and run it from shell, the parent shell (from which the script was launched) doesn't inherit variables set in the script:

```bash
$ echo $$ # check current shell PID
15019
$ cat <<'EOF' > test1.sh
#!/bin/bash
VAR5=hello # shell var
export VAR6=hola # environment var
echo "shell PID: $$"
echo "output from script: $VAR5 - $VAR6"
EOF
$ chmod +x test1.sh
$ ./test1.sh
shell PID: 15041
output from script: hello - hola
$ echo "output: $VAR5 - $VAR6"
output:  -
```

As you can see variables are not visible in the shell from which we launched the script, because the script was run in a new shell specified in the shebang (`#!/bin/bash`).

#### Running scripts in the current shell

In bash there are 2 commands **source** and **.** (dot command) that allow you to execute commands from a script in the current shell (see `help source` or `help .`). As the result, after running scripts with either one of these commands, the current shell environment will be changed if the script has variables definitions.

Let's run the `test1.sh` script from the [previous example](#running-scripts-in-a-separate-shell/process):

```bash
$ echo $$ # check current shell PID
15019
$ echo "output: $VAR5 - $VAR6"
output:  -
$ . ./test1.sh
shell PID: 15019
output from script: hello - hola
$ echo "output: $VAR5 - $VAR6"
output: hello - hola
```

As you can see, the script commands were run in the same shell with PID=15019 and new environment variables were set in the current shell after running the script.

The `source` and dot commands are very useful and often used for updates of current shell environment from a script. For example, you may find yourself sourcing a `~/.bashrc` or `~/.zshrc` file quite often when you update them with new variables or aliases and want the changes to be visible in the current shell.

### Listing variables

To get a list of _environment variables_, run the **env** command:

```bash
VAR5=hello
$ export VAR6=hola
$ env | fgrep VAR # will show only exported variables
VAR6=hola
```

To get a list of all variables, i.e. shell and environment variables included, you can run **set** command:

```bash
$ set | fgrep VAR
VAR5=hello
VAR6=hola
```

### Unsetting variables

If you haven't [persisted the variables set in the current shell into files](#persisting-variables), then you can simply launch a new shell and those variables will be gone.

You can also clear shell/environment variables using the **unset** command:

```bash
$ VAR5=hello
$ export VAR6=hola
$ set | fgrep VAR
VAR5=hello
VAR6=hola
$ unset VAR5
$ set | fgrep VAR
VAR6=hola
$ unset VAR6
$ set | fgrep VAR
```

### Persisting variables

_Persisting variables means recording them to files read by a shell when it gets started._

#### Login shells

When a [login shell](shells.md) starts, it first executes `/etc/profile` script. So if you want to set some variables for all users, you can change this file.

Note: the `/etc/profile` script is often set to read scripts in the `/etc/profile.d` directory. It's a good idea to make your environment modifications by adding files to the `/etc/profile.d` directory instead of changing `/etc/profile` script directly:

```bash
$ echo "export LOGIN_VAR1=hola" > /etc/profile.d/test_vars.sh
$ /bin/bash -l         # start a login shell
$ echo $LOGIN_VAR1
hola
$ exit                 # log out of the started shell
logout
```

As you can see, upon start of a new login shell the new environment variable `LOGIN_VAR1` was set for me.

Changing `/etc/profile` file will affect all system users. If you want to customize the shell environment only for your user, you will need to add it to `~/.bash_profile`, `~/.bash_login`, or `~/.profile` file.

After running `/etc/profile` script, the new login shell will look for
`~/.bash_profile`, `~/.bash_login`, or `~/.profile` script (in this specific order) and executes the first one that it finds.

```bash
$ ls ~/.bash_profile             # check if I have ~/.bash_profile file
/home/vagrant/.bash_profile
$ echo "export LOGIN_VAR2=hola2" >> ~/.bash_profile
$ /bin/bash -l                   # start a login shell
$ echo $LOGIN_VAR2
hola2
$ exit                           # log out of the started shell
logout
$ /bin/bash                      # start a non-login shell
$ echo $LOGIN_VAR2

$ exit                           # log out of the started shell
exit
```

As you can see it worked, the `LOGIN_VAR2` variable was set for my user environment when I launched a new [login shell](shells.md), but it wasn't set when I launched a [non-login shell](shells.md). Read below on how to make it work for non-login shells as well.

#### Non-login interactive shells

The [non-login shell](shells.md) copies the parent shell environment and then reads the user's `~/.bashrc` file for additional startup configuration instructions.

Most Linux distros configure the login configuration files (e.g. `~/.bash_profile`, `~/.bash_login`) to _source_ the non-login configuration file. For example, my `~/.bash_profile` script has the following line:

```bash
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi
```

This means that by defining variables in the `~/.bashrc`, I can set them for both login and non-login shells:

```bash
$ echo "export LOGIN_VAR3=hola3" >> ~/.bashrc
$ /bin/bash -l                   # start a login shell
$ echo $LOGIN_VAR3
hola3
$ exit                           # log out of the started shell
logout
$ /bin/bash                      # start a non-login shell
$ echo $LOGIN_VAR3
hola3
$ exit                           # log out of the started shell
exit
```

### Passing variables to commands

In bash the following syntax can be used to add values to the environment used to execute a single
program, without affecting the parent shell (and subsequent commands):

```bash
$ NAME=value program
```

This adds a variable definition to the environment of just the child process executing the specified program. If desired, multiple assignments (delimited by white space) can precede the program name.

For example:

```bash
$ cat <<'EOF' > echo-program.sh
#!/bin/bash

echo "output: ${VAR10}"
EOF
$ chmod +x echo-program.sh
$ export VAR10='hello world'            # set environment var in the current shell
$ ./echo-program.sh                     # check output of the script
output: hello world
$ VAR10='hola mundo' ./echo-program.sh  # pass the variable values to the script
output: hola mundo
```

### Resources used to create this document
* Chapter 6.7 of [Linux Programming Interface](https://www.amazon.com/Linux-Programming-Interface-System-Handbook-ebook/dp/B004OEJMZM)
* http://www.linuxfromscratch.org/blfs/view/cvs/postlfs/profile.html
* https://www.digitalocean.com/community/tutorials/how-to-read-and-set-environmental-and-shell-variables-on-a-linux-vps
* https://unix.stackexchange.com/questions/138463/do-parentheses-really-put-the-command-in-a-subshell
