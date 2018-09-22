## which

**which** command is a simple command used to locate commands executables on the system.

`which` takes names of the commands as arguments and returns the pathnames of the files (or links) which would be executed in the current environment if those commands were run. It does this by searching the `PATH` for executable files matching the names of the passed arguments (it does not follow symbolic links).

For example, let's find the executable of the [date](date.md) command:

```bash
$ which date
/bin/date
```

The output shows that if we were to run `date` command, the `/bin/date` executable file would be run:

```bash
$ date
Sat Sep 22 04:57:14 UTC 2018
$ /bin/date
Sat Sep 22 04:57:18 UTC 2018
```

### Resources used to create this document:

* man which
