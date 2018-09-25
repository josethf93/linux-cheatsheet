## Shell builtins

**Shell builtin commands** (or simply **shell builtins**) are commands contained within the shell itself.
When a shell builtin command is called, the shell executes the command directly in the shell itself, instead of looking for an external executable program which the shell would load and execute.

Shell builtins work significantly faster than external programs, because there is no program loading overhead. However, their code is inherently present in the shell, and thus modifying or updating them requires modifications to the shell.

You can determine if the command is a shell builtin or not by using the [type](type.md) command:

```bash
$ type -a eval
eval is a shell builtin
$ type -a history
history is a shell builtin
$ type -a logout
logout is a shell builtin
```

Note that some commands may be present as shell builtins and as external commands:

```bash
$ type -a cd
cd is a shell builtin
cd is /bin/cd
cd is /usr/bin/cd
```

### Resources used to create this document:

* https://www.gnu.org/software/bash/manual/bashref.html#Shell-Builtin-Commands
* https://en.wikipedia.org/wiki/Shell_builtin
* https://unix.stackexchange.com/questions/442945/understanding-shell-builtin-commands
* https://stackoverflow.com/questions/3192373/what-are-shell-built-in-commands-in-linux
