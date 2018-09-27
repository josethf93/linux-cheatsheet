## tee

The **tee** command line utility is used to store and at the same time view the output of any other command. It does it by copying standard input (stdin) to standard output ([stdout](http://www.linfo.org/standard_output.html)) while also saving a copy of that input in zero or more files.

Consider the following example. By default, the `echo` command writes its output to the stdout which in turn has a [default destination](http://www.linfo.org/standard_output.html) of your display screen. That's why you immediately see the command's output:

```bash
$ echo "hello world"
hello world
```

One of the features of standard output is that its default destination can easily be [redirected](io-redirection.md) (i.e., diverted) to another destination. For example, the following command will redirect the stdout of the `echo` command to a file named `output.txt`:

```bash
$ echo "hello world" > output.txt
```

As a result, we don't see the output of the command on the screen, because it gets written to a file `output.txt` instead. Let's see the contents of the file:

```bash
$ cat output.txt
hello world
```

But what if you wanted to save the command's output to a file while also being able to see it on the screen at the same time? This is what the `tee` command allows you to do:

```bash
$ echo "hola mundo" | tee output.txt
hola mundo
$ cat output.txt
hola mundo
```

In this case we use a [pipe](pipe.md) to redirect output of `echo` command to the input of `tee` command. The `tee` command takes the file path as an argument and saves the input in that file while displaying that input on the screen.

Note that you can provide more than one file path to the `tee` command:

```bash
$ echo "vsem privet" | tee output1.txt output2.txt
vsem privet
$ cat output1.txt
vsem privet
$ cat output2.txt
vsem privet
```

### Other use cases

#### Write the state of a pipe

As data flows through UNIX [pipes](pipe.md), it can be useful to take a snapshot of the state of the data. This can be useful for debugging purposes or to take a backup of some configuration.

The following command will take a backup of the crontab entries, and pass the crontab entries as an input to sed command which will do the substituion. After the substitution, it will be added as a new cron job:

```bash
$ crontab -l | tee crontab-backup.txt | sed 's/old/new/' | crontab –
```

#### Write to a privileged file

`tee` may also be used as part of a pipe to elevate to `sudo` permissions to be able to write to a privileged file.

Suppose you have a file owned by root. Trying to write to this file as a normal user will result in a permissions error:

```bash
$ whoami
vagrant
$ echo "hello world" > /root.out.txt
-bash: /root.out.txt: Permission denied
$ sudo echo "hello world" > /root.out.txt
-bash: /root.out.txt: Permission denied
```

We can use `tee` with `sudo` to achieve what we want:

```bash
$ echo "hello world" | sudo tee /root/out.txt
hello world
$ sudo cat /root/out.txt
hello world
```

### Useful options

#### Append to a file instead of rewriting it

To append to a file instead of rewriting it, use the **-a** option:

```bash
$ echo "hello world" | tee output.txt
hello world
$ echo "hello world" | tee -a output.txt
hello world
$ cat output.txt
hello world
hello world
```

### Resources used to create this document:

* https://shapeshed.com/unix-tee/
* https://linux.101hacks.com/unix/tee-command-examples/
* https://unix.stackexchange.com/questions/362090/how-does-the-tee-command-works
