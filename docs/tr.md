## tr

The **tr** (short for "translate") command is used to translate specified characters into other characters or to delete them.

The general syntax of `tr`:

```
tr [options] set1 [set2]
```

The items in the square brackets are optional. `tr` requires at least one argument and accepts a maximum of two. The first, designated `set1`, lists the characters to be replaced or removed from the text. The second, `set2`, lists the characters that are to be substituted for the characters listed in the first argument.

In contrast to many command line programs, `tr` does not accept file names as arguments. Instead, it only accepts inputs via standard input or from the output of other programs via redirection.

For example, the following command will replace all the `l` characters with `*` character in a file and write it to the stdout:

```bash
$ echo 'hello world' > file1.txt
$ tr 'l' '*' < file1.txt
he**o wor*d
```

**Note:** remember to not redirect output to write to the same file from which the text is being read, as this will erase all of the text in that file, including that outputted by `tr`. Thus, for example, the following should not be used:

```bash
$ cat file1.txt
hello world
$ tr 'l' '*' < file1.txt > file1.txt
$ cat file1.txt
$
```

### Command examples

#### Replace multiple characters

Suppose you want to replace "l" with "*", "o" with "+" and "h" with "-". Then in the sets you specify, those characters should be at the same positions:

```bash
$ tr 'loh' '*+-' < file1.txt
-e**+ w+r*d
```

See what happens when the number of characters in the two sets don't match:

```bash
$ tr 'loh' '*+' < file1.txt
+e**+ w+r*d
```

the "h" was replaced with the last character in the second second set which was "+".

#### Replace ranges of characters

`tr` can also replace characters in a specified range by their counterparts in another specified range, which can be more convenient when numerous characters are involved. A range is indicated by inserting a hyphen between the first and last characters:

```bash
$ tr 'a-er-z' 'A-ER-Z' < file1.txt
hEllo WoRlD
```

#### Remove character repetitions

When the **-s** (**--squeeze-repeats**) option is specified, after any deletions or translations have taken place, repeated sequences of the same character will be replaced by one occurrence of the same character. Compare the two commands with and without this option:

```bash
$ echo 'hello world' > file1.txt
$ tr 'l' '*' < file1.txt
he**o wor*d
$ tr -s 'l' '*' < file1.txt
he*o wor*d
```

As you can see, repeated sequences of the same character (in this case "*" character) were removed after translation.

In the example below, we replace numerous spaces with one space:

```bash
$ cat file1.txt
Hello       world    !
$ tr -s ' ' < file1.txt
Hello world !
```

#### Translate multiple lines into one sentence

To accomplish this, we need to replace the new line character (`\n`) with a space:

```bash
$ cat file1.txt
hello
world
!
$ tr -s '\n' ' ' < file1.txt
hello world !
```

#### Delete characters

To delete characters, use the **-d** option:

```bash
$ cat file1.txt
Hello world
$ tr -d 'l' < file1.txt
Heo word
$ tr -d 'lo' < file1.txt
He wrd
$ tr -d 'lod' < file1.txt
He wr
```

For example, to remove all digits from text I could use the following command:

```bash
$ cat file1.txt
H9ll8 w7rl1
$ tr -d '0-9' < file1.txt
Hll wrl
```

Also, when the  **-c**  option is used along with the `-d`, all values except those specified by [set1](#tr) will be deleted. So the following command will remove everything but numbers from the text (compare to the previous example):

```bash
$ cat file1.txt
H9ll8 w7rl1
$ tr -cd '0-9' < file1.txt
9871
```

#### Replace non-matching characters

The **-c** (**--complement**) option makes the `tr` command to work in reverse. If without this option the `tr` will translate characters you specify, with this option it will translate all non-matching characters.

For example, compare the output from the two commands:

```bash
$ echo 'hello world' > file1.txt
$ tr 'l' '*' < file1.txt
he**o wor*d
$ tr -c 'l' '*' < file1.txt
**ll*****l**
```

#### Interpreted sequences of characters

`tr` can also be used with **control characters**. Also referred to as non-printing characters, these are bit sequences that represent basic formatting instructions, etc., and each begins with a backslash. They are `\b` for backspace, `\f` for form feed, `\n` for new line, `\r` for return, etc.

For example, the following command will print each world of text on a new line:

```bash
$ cat file1.txt
hello world & hola mundo
$ tr -s ' ' '\n' < file1.txt # replace spaces with a new line character
hello
world
&
hola
mundo
```

`tr` also has a number of **predefined arguments** that represent ranges of certain types of characters. Each consists of a word or abbreviation surrounded by colons and then enclosed in a set of square brackets. For example, all of the letters of the alphabet are represented by `[:alpha:]`, all numerals are represented by `[:digit:]`, all alphanumeric characters are represented by `[:alnum:]`, all lower case letters are represented by `[:lower:]`, all upper case letters are represented by `[:upper:]`, etc.

For example:

```bash
$ cat file1.txt
H9ll8 w7rl1
$ tr -cd [:digit:] < file1.txt # same as tr -cd '0-9' < file1.txt
9871
$ cat file1.txt
hello world
$ tr [:lower:] [:upper:] < file1.txt # same as tr 'a-z' 'A-Z' < file1.txt
HELLO WORLD
```

**Note:** to see a full list of interpreted sequences of characters supported by `tr`, run `tr --help` command.

### Resources used to create this document:

* https://www.thegeekstuff.com/2012/12/linux-tr-command/
* http://www.linfo.org/tr.html
* https://www.tecmint.com/tr-command-examples-in-linux/
* https://www.linuxnix.com/16-tr-command-examples-in-linux-unix/
* https://linoxide.com/how-tos/linux-tr-command/
* https://www.poftut.com/linux-tr-command-tutorial-examples/
* `man tr`, `tr --help`
