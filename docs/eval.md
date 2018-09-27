## eval

**eval** is a [shell builtin command](shell-builtins.md) that is used to execute its arguments as a shell command.

`eval` comes in handy when you have a command stored in a variable and you want to execute that command stored in the string:

```bash
$ COMMAND="ls -l"
$ eval $COMMAND
total 0
-rw-rw-r--. 1 vagrant vagrant 0 Sep 27 04:25 file.txt
-rw-rw-r--. 1 vagrant vagrant 0 Sep 27 04:25 script.sh
$ COMMAND2="rm file.txt; touch newfile.txt"
$ eval $COMMAND2
$ COMMAND="ls -l"
$ eval $COMMAND
total 0
-rw-rw-r--. 1 vagrant vagrant 0 Sep 27 04:26 newfile.txt
-rw-rw-r--. 1 vagrant vagrant 0 Sep 27 04:25 script.sh
```

### Resources used to create this document:

* https://unix.stackexchange.com/questions/23111/what-is-the-eval-command-in-bash
* https://www.folkstalk.com/2012/10/executes-eval-arguments-as-shell.html
* https://www.tutorialspoint.com/unix_commands/eval.htm
* help eval
