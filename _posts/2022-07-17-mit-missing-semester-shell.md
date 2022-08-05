---
title:  Missing Semester - The Shell
category: "Linux"
tags: [Shell, Linux, MIT, Missing Semester]
---

MITのCS基礎コース「The Missing Semester of Your CS Education」についてのメモ。

Course located at: https://missing.csail.mit.edu/

<br/>

## What is the shell

- A textual interface to the system  
- A programming environment (like ruby, python) which can run programs, commands, shell scripts. Also has functions, conditionals, path variables, loops.

## Using the shell

- Environment variable(環境変数)

```zsh
 $ date
2022年 7月29日 金曜日 15時20分22秒 JST
$ echo Hello
Hello
```
When input a command, the shell will match one of its programming keywords.  
ex. Input `date`, the shell will execute the `date` program to print the date on the terminal.

If the shell doesn’t match any keywords, it consults an `environment variable`, called `$PATH`, that lists which directories the shell should search for programs when it is given a command.

```zsh
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
$ which echo
/bin/echo
$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Use `man` to check the manual of command

comman like `ls --help` doesn't work on MacOS, instead use `man ls` . (Press `q` to exit.)

## Connecting programs

## The input stream and output stream

In the shell, programs have two primary “streams” associated with them: 
their input stream and their output stream. When the program tries to read input, it reads from the input stream, and when it prints something, it prints to its output stream. 

Normally, a program’s input and output are both your terminal. That is, your keyboard as input and your screen as output. 

## Using `cat` with  `< `,  `> ` and  `>>` 
`cat`: a program that con`cat`enates(連結) files.

When given file names as arguments, it prints the contents of each of the files in sequence to its output stream. 

But when `cat` is not given any arguments, it prints contents from its input stream to its output stream.
```zsh
$ echo hello > hello.txt # "hello" as input stream, "hello.txt" as output stream
$ cat hello.txt
hello
$ cat < hello.txt # hello.txt as input stream
hello
$ cat < hello.txt > hello2.txt # prints contents from hello.txt to hello2.txt(output stream)
$ cat hello2.txt
hello
```
use `>>` to append to a file.

```zsh
$ cat < hello.txt >> hello2.txt
$ cat hello2.txt  #will get two lines of "hello"
hello
hello
```

## Using `|` operator to chain programs

Like using the output of one program as the input of another program.

```zsh
$ ls -l / | tail -n1 # listing files under / and display the last line
lrwxr-xr-x@  1 root  wheel    11  1  1  2020 var -> private/var
```

`ls -l`: use a long listing format
`/`: use `/` the home directory as argument of `ls -l`

`tail -n [number]`:  display the last part of a file, with `n`lines.


```zsh
$ curl --head --silent google.com | grep --ignore-case content-length | cut -d' ' -f 2
219
```
`curl`: transfer data to or from a server  
`--head --silent`: Fetch the headers only, in silent mode (without showing progress meter or error messages.)

`grep`: a utiltiy that searches any given input files, selecting lines that
     match one or more patterns.  
Used like ` grep [search options] [file]`
` grep --ignore-case content-length`: search the line matches "content-length" ignoring case

`cut`:  cut out selected portions of each line of a file  
`-d (--delimiter) `: Specify a delimiter(区切り文字) that will be used instead of the default “TAB” delimiter.  
`-f (--fields=LIST) ` Select by specifying a field, a set of fields, or a range of fields. The most commonly used option.

ex. for file `test.txt`
```test.txt
245:789 4567    M:4540  Admin   01:10:1980
535:763 4987    M:3476  Sales   11:04:1978
```
to display the 1st and 3rd fields using “:” as a delimiter:

```zsh
$cut test.txt -d ':' -f 1,3
# output will be
245:4540	Admin	01
535:3476	Sales	11
```

## Permission bits of the file

There are 3 sets of permissions. One set for the owner of the file, another set for the members of the file’s group, and a final set for everyone else.

```zsh
$ ls -l
drwxrwxr-x  29 root  admin   928  7 22 17:18 Applications
-rw-r--r--  1 mia  staff  61  7 29 11:07 semester
```
first character:   
`d`: means a directory  
`-`: means a file

next 9 character indicates 3 sets:  
first 3 characters: permissions for the owner  
midlle 3 characters: permissions for the group member  
last 3 characters: permissions for everyone else

 Permission bits (characters indicate permissions):  
`r`: read, it ban be opened, and its content be viewed  
`w`: write, it can be edited, modified, and deleted  
`x`: execute, can be executed if it is a script or a program.


## Using Shebang `#!` and `chmod`

**`#!interpreter [arguments]`**  
the exclamation mark (!) is used with the number sign/hash (#) symbol to **specify the interpreter path**. This usage is called “`shebang`” , or `hashbang(`#!`)

Often used for writing shell scripts.

When we try to run an executable file, the `execve` program is called to replace the current process (bash/zsh shell if we are using terminal) with a new one and decide how exactly that should be done.

If we try to run a text file, `execve` expects the first two characters of the file to be “`#!`”  followed by a path to the interpreter that will be used to interpret the rest of the script.

ex.   
```zsh
$ echo '#!/bin/sh' >> greetings
$ echo 'echo "Hello, ${USER}"' >> greetings
$ cat greetings
#!/bin/sh
echo "Hello, ${USER}"

$ sh greetings
Hello, mia
```
`sh`: A shell command language interpreter that executes commands read from a command line string, the standard input, or a specified file.  


**Using `chmod` to modify file permissions**

To use `chmod` to set permissions, we need to tell it:

- *Who:* Who we are setting permissions for.
    - `u, g, o, a`( for ' user / group/ others / or all above without specifying)
- *What*: What change are we making? Are we adding or removing the permission?
    - `-`: Remove permissions
    - `+` : Add permissions
    - `=` : Set a permission and remove others
- *Which*: Which of the permissions are we setting?  
`r`, `w`, `x`

```zsh
$ ls -l
-rw-r--r--  1 mia  staff  32  7 29 17:38 greetings

$  chmod +x greetings  # add execute permission to all
$ ls -l
-rwxr-xr-x  1 mia staff  32  7 29 17:38 greetings
```

Then, `greetings` is an executable file
```zsh
$  ./greetings
Hello, mia
```

## Exercises

*Create a new directory called `missing` under `/tmp`.*
```zsh
mkdir /tmp/missing
```
*Use `touch` to create a new file called `semester` in` missing`.*
```
touch /tmp/missing/semester
```
*Write the following into that file, one line at a time:*

`#!/bin/sh`

`curl --head --silent https://missing.csail.mit.edu`

```zsh
echo '#!/bin/sh' >> semester
echo 'curl --head --silent https://missing.csail.mit.edu' >> semester
```

*Try to execute the file, i.e. type the path to the script (`./semester`) into your shell and press enter. Understand why it doesn’t work by consulting the output of `ls `(hint: look at the permission bits of the file).*
```
No execution bit
```

*Run the command by explicitly starting the `sh` interpreter, and giving it the file `semester` as the first argument, i.e. `sh semester`. Why does this work, while `./semester `didn’t?*

>`./semester `asks the `kernel` to run `semester` as a program, and the `kernal` (program loader) will check permissions first, and then use /bin/zsh (or sh or bash etc) to actually execute the script.  
>`sh semester `asks the `kernel` (program loader) to run `/bin/sh`, not the program so the execute permissions of the file do not matter.

<br/>

*Use `chmod` to make it possible to run the command` ./semester` rather than having to type `sh` semester. How does your shell know that the file is supposed to be interpreted using `sh`?*

```zsh
$ chmod +x semester
$ ./semester
HTTP/2 200 
server: GitHub.com
content-type: text/html; charset=utf-8
x-origin-cache: HIT
last-modified: Sat, 14 May 2022 10:50:11 GMT
access-control-allow-origin: *
etag: "627f8963-1f37"
expires: Sat, 23 Jul 2022 03:08:29 GMT
cache-control: max-age=600
x-proxy-cache: MISS
x-github-request-id: ED62:476C:5D2A8:BC4B1:62DB63D5
accept-ranges: bytes
date: Fri, 29 Jul 2022 09:02:02 GMT
via: 1.1 varnish
age: 0
x-served-by: cache-tyo11976-TYO
x-cache: HIT
x-cache-hits: 1
x-timer: S1659085322.824124,VS0,VE435
vary: Accept-Encoding
x-fastly-request-id: 94ac286b0de6f5f883905082fa06aef9c1ce2227
content-length: 7991
```

>The shebang is parsed as an interpreter directive by the program loader mechanism. The loader executes the specified interpreter program, passing to it as an argument the path that was initially used when attempting to run the script, so that the program may use the file as input data.

*Use` | `and` >` to write the “last modified” date output by `semester `into a file called `last-modified.txt` in your home directory.*

```zsh
# get the 5th line
$ ./semester | head -n 5 | tail -n 1 
or 
$ ./semester | sed -n 5p
last-modified: Sat, 14 May 2022 10:50:11 GMT

or use grep
$ ./semester | grep last-modified | cut -d ':' -f 2-4 | > ~/last-modified.txt

$ cat ~/last-modified.txt 
Sat, 14 May 2022 10:50:11 GMT
```

---
参照：

[Cut Command in Linux](https://linuxize.com/post/linux-cut-command/)

[Using Shebang #! in Linux Scripts](https://www.baeldung.com/linux/shebang)

[curl command in Linux with Examples](https://www.geeksforgeeks.org/curl-command-in-linux-with-examples/)

[How to Use the chmod Command on Linux](https://www.howtogeek.com/437958/how-to-use-the-chmod-command-on-linux/)

[./missing-semester - Course Overview + The Shell - Exercises](https://zacheller.dev/missing-semester0)