# Vars and loops

Before this lesson, we mainly used the shell did to call other programs with specific parameters, and used pipes, which are practically identical between shells, to glue different commands together. There are shells other than `bash`, but so far those would have done the exact same thing.

But with variables, this is no longer the case. So before we start, I need you to run this command:

```
$ echo $0
```

and verify that it actually returns `bash` (in some form or other, like `-/bin/bash`).

If it doesn't, you can always start a bash shell by running the command

```
$ bash
```

## Greet me

For starters, we want the computer to greet me. We can do something like this:

```
$ echo Hello Derrial
Hello Derrial
```

However, this is all hard coded, and I would have to change everything to greet someone else.

This is where variables come in.

Bash already supplies many variables, and we can get a list of them with the command `declare`:

```
$ declare
BASH=/bin/bash
BASH_ALIASES=()
BASH_ARGC=()
BASH_ARGV=()
BASH_CMDS=()
BASH_COMPLETION_COMPAT_DIR=/etc/bash_completion.d
BASH_LINENO=()
BASH_REMATCH=()
BASH_SOURCE=()
...
```

In there, we find the variable `USER`, and it contains our user name. The first attempt to use it would look something like this:

```
$ echo Hello USER
Hello USER
```

This is not what we want. The shell has not replaced the word `USER` with its value. We have to tell the shell that this is a variable, and that it should use its value instead. This is done by putting a `$` in front of it:

```
$ echo Hello $USER
Hello derrialb
```

But the username isn't really what we want. We want the real name, so we try to create our own variable.

```
$ $NAME=Derrial
=Holger: command not found
```

This didn't work either, because `bash` has replaced `$NAME` with its contents (and before it is set, the contents are empty), and then tried to interpret the resulting command `=Holger`, which of course failed.

When **assigning** a variable, we must **not** use the dollar sign:

```
$ NAME=Derrial
$ echo Hello $NAME
Hello Derrial
```

### Exercise:

What is the difference between these 3 commands: (Try them out)

```
$ echo Hello $NAME
$ echo "Hello $NAME"
$ echo 'Hello $NAME'
```

(Hint: put some more spaces between "Hello" and "$NAME")

## How to assign values to variables

When assigning values to variables, it is important to not use spaces outside of qutations:

```
$ NAME = Kylee
NAME: command not found
$ NAME= Kylee
Holger: command not found
$ NAME=Kylee Frye
Frye: command not found
```

We'll come to the reason behind these strange errors in a bit.

If the content of the variable should contain spaces, the whole content must be surrounded by quotation marks:

```
$ NAME="Kaylee Frye"
$ echo $NAME
Kylee Frye
```

The command `unset` deletes a variable:

```
$ NAME=Saffron
$ echo $NAME
Saffron
$ unset NAME
$ echo $NAME
```

To ask the user for something to be put into a variable, use the `read` command:

```
$ read -p "Please enter your name: " NAME
Please enter your name: Malcom Reynolds
$ echo Hello $NAME
Hello Malcom Reynolds
```

The option `-p` sets the prompt for the user, and whatever the user enters is stored in the variable `NAME`.

But we can also automatically get the name from the system: The command `finger` provides a bit of information about a user, including the name.

```
$ finger $USER
Login: iserra                    Name: Inara Serra
Directory: /home/iserra            Shell: /bin/bash
On since Mon Jun 18 10:16 (AEST) on tty8 from :0
    2 hours 38 minutes idle
On since Mon Jun 18 10:16 (AEST) on pts/5 from :0.0
No mail.
No Plan.
$ finger $USER | head -1
Login: iserra                    Name: Inara Serra
$ finger $USER | head -1 | tr -s '\t' ' '
Login: iserra Name: Inara Serra
$ finger $USER | head -1 | tr -s '\t' ' ' | cut -d ' ' -f 4-
Inara Serra
```

This is a callback to last week's pipes: The output of `finger` is piped into `head -1` to get only the first line, that is piped into `tr -s '\t' ' '` to convert the tabs into spaces and condense successive spaces into a single space. And finally `cut -d ' ' -f 4-` is used to split the line into space deliited fields and only get the fields starting from 4th.

To get this output into a variable, the whole line is surrouned by either `$( )` or ``:

```
$ NAME=$(finger $USER | head -1 | tr -s '\t' ' ' | cut -d ' ' -f 4-)
$ NAME=`finger $USER | head -1 | tr -s '\t' ' ' | cut -d ' ' -f 4-`
```

## Passing variables to child processes

Because it's easy to assess, we use a child `bash` environment as a child process, but any program started by the shell is a child process.

So we've set a variable, then start a child process:

```
$ NAME="Hoban \"Wash\" Washburne"
$ bash
(child) $ echo $NAME

(child) $ exit
$ echo $NAME
Hoban "Wash" Washburne
```

(If your variable needs to contain quotation marks, you can escape them as seen above.)

The variable `NAME` was not passed on to the child process. To do that, we need to `export` the variable

```
$ export NAME
$ bash
(child) $ echo $NAME
Hoban "Wash" Washburne
(child) $ exit
```

We can also set a variable _only_ for a child process:

```
$ unset NAME
$ NAME="River Tam" bash
(child) $ echo $NAME
River Tam
(child) $ exit
$ echo $NAME
```

(This is why the earlier assignments `NAME= Kaylee` and `NAME=Kaylee Frye` failed.)

## Modifying variables

Variables can be used as content of other variables:

```
$ NAME="Simon Tam"
$ GREETING="Hello $NAME"
$ echo $GREETING
Hello Simon Tam
```

Note that the variable `NAME` gets evaluated at the assignment to `GREETING`. If you change `NAME` afterwards, you have to change `GREETING` too:

```
$ NAME="Jayne Cobb"
$ echo $GREETING
Hello Simon Tam
```

We can use this to accumulate text:

```
$ NAMES="Malcom"
$ NAMES="$NAMES Zoe"
$ NAMES="$NAMES Wash"
$ NAMES="$NAMES Jayne"
$ NAMES="$NAMES Kaylee"
$ echo $NAMES
Malcom Zoe Wash Jayne Kaylee
```

What if we don't want spaces inbetween?

```
$ FOO="BAR"
$ FOO="$FOOBAR"
$ echo $FOO
```

This didn't work, as the second assignment to `FOO` assined the contents of the (empty) variable `FOOBAR`, and did not append "BAR" to the end of variable `FOO`.

In order to do that, we have to encapsulate the variable name in curly braces:

```
$ FOO="BAR"
$ FOO="${FOO}BAR"
$ echo $FOO
BARBAR
```

We can also only return parts of a variable. `${VARNAME:n:l}` omits the first `n` characters, then prints the next `l`:

```
$ echo $NAMES
Malcom Zoe Wash Jayne Kaylee
$ echo ${NAMES:7:3}
Zoe
```

To omit the _shortest_ match of a substring from a string variable, use a single `#`:

```
$ echo ${NAMES#* }
Zoe Wash Jayne Kaylee
```

This removed the shortest match of

<any number="" of="" random="" chars=""> followed by a space.
In other words, the first name.</any>

Note that this does not change the contents of `NAMES` at all, it just only prints part of it.

To omit the _longest_ match, use the double `##`:

```
$ echo ${NAMES##* }
Kaylee
```

The same can also be done from the rear with `%` (shortest match) or `%%` (longest match):

```
$ echo ${NAMES% *}
Malcom Zoe Wash Jayne
$ echo ${NAMES%% *}
Malcom
```

(Of course the pattern has to change too, now it's a space followed by any number of characters.)

### Exercise:

What does the term `${#VARNAME}` do?

```
$ FOO=BAR
$ echo ${#FOO}
3
```

(Hint: change the variable `FOO` to different strings, see what changes with `${#FOO}`.)

## Integer variables

So far, we've only looked at string variables. What about integers?

```
$ i=0
$ echo $i
0
$ i=$i+1
$ echo $i
0+1
```

By default, `bash` interprets any variable as a string, which means it can't do math on it.

One way to get around it is to use the program `bc`, which is a quick and easy calculator

```
$ bc
bc 1.06.95
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type 'warranty'.
1+2
3
```

As we discussed in an ealier session, we can pipe text into the standard input of a program we call:

```
$ bc <<< "3-1"
2
```

We can assign this to a variable:

```
$ i=4
$ i=$(bc <<< "$i-1")
$ echo $i
3
```

But there is an easier way: You can declare a variable as integer:

```
$ declare -i n=6
$ echo $n
6
$ n=$n+2
$ echo $n
8
```

In fact, if `n` is declared as an integer, bash will try everything to interpret the assignment as an integer, to the point even to recognising variables without `$`:

```
$ declare -i myvar=4
$ myvar=myvar+2
$ echo $myvar
6
```

And finally, there's a C-like syntax if you surround the expression with double parentheses:

```
$ k=4
$ (( k++ ))
$ echo $k
5
```

## For Loops

Say we have lots of files in our directory, all ending in `.dat`, which we want to work with.

But just in case, we want to make a backup of each file first:

```
$ cp *.dat *.bak
cp: target *.bak is not a directory
```

Here, we need a for loop:

```
$ for FILE in *.dat;
> do
>     echo cp $FILE ${FILE%dat}bak
> done
cp atlantic.dat atlantic.bak
cp indian.dat indian.bak
cp north_polar.dat north_polar.bak
cp pacific.dat pacific.bak
cp south_polar.dat south_polar.bak
```

When pressing enter after the `for` line, nothing happens, instead, bash presents us with a different prompt: `>` `bash` has recognised that the command isn't finished yet. It still needs a code block, encapsulated by `do` and `done`, to tell it what it should do for each element of the `for` loop.

You notice that for the first time, instead of executing the `cp` command, I've used `echo` to instead print the command to the screen. That way I can make sure that the text substitution works correctly.

for each element in the list (and don't forget, `bash` expands `*.dat` into a list of all files that match this pattern), the variable `FILE` gets set to this element.

If we're happy with this output, then we press the **up** key, and get a different line:

```
$ for FILE in *.dat; do     echo cp $FILE ${FILE%dat}bak; done
```

This is not actually a different command, `bash` has just pushed all commands into a single line, delimited by semicolons. From `bash`'s perspective, this is the same as the multiline.

Remove the `echo` from that line and run it, and the files will be copied, one-by-one.

This list of values doesn't have to be a list of file names. It can be anything:

```
$ for v in You cant take the sky from me; do echo "in this iteration, v is $v"; done
in this iteration, v is You
in this iteration, v is cant
in this iteration, v is take
in this iteration, v is the
in this iteration, v is sky
in this iteration, v is from
in this iteration, v is me
```

### Exercise:

Write a loop to greet 4 people (select your own names)

## Numeric Loops

Often you will need the iterator to loop over numbers. The easiest way to do that is to use `{n..m}`, which `bash` expands to a list of numbers from `n` to `m`:

```
$ echo {3..5}
3
4
5
$ for f in {3..5}; do echo "This is iteration for $f"; done
This is iteration for 3
This is iteration for 4
This is iteration for 5
```

But if you want more control, you can use the `seq` command:

```
$ for v in $(seq -w 0 0.1 1); do echo "v is $v"; done
v is 0.0
v is 0.1
v is 0.2
v is 0.3
v is 0.4
v is 0.5
v is 0.6
v is 0.7
v is 0.8
v is 0.9
v is 1.0
```

See the `seq` man page for info.

## Conditional Loops

There are two more loops apart from the `for` loop: `while` and `until`.

In both cases, at the beginning of each iteration of the loop, a condition is checked.

- A `while` loop will continue executing the body of the loop _as long as_ this contition evaluates to true.
- A `until` loop will continue executing the body of the loop _until_ this condition evaluates to true.

For example:

```
$ while True
> do
>    echo "Hello there"
>    sleep 1
> done
Hello there
Hello there
Hello there
```

This loop will continue to print "Hello there" (and then wait 1 second) for all eternity (or until it is killed with CTRL-C).

## Exit Codes

What does it mean "a condition evaluates to true"?

Every command you run will return a so-called exit code. This is an integer number, and by convention, a result of 0 means that the command executed correctly. Any other number represents some sort of "unexpected behaviour"

You can get the exit code of the last command with `$?`

```
$ ls
atlantic.dat  indian.dat  north_polar.dat  pacific.dat  south_polar.dat
$ echo $?
0
$ ls foo
ls: cannot access 'foo': No such file or directory
$ echo $?
2
$ echo $?
0
```

Note that when we asked `ls` to list a file that didn't exist, the exit code was 2\. But immediately afterwards it was back to 0, because it was no longer the exit code of `ls` but of the last command: `echo`.

So if you need the exit code later, you need to store it in your own variable:

```
$ ls $FILE
$ EC=$?
$ echo "something else"
$ echo "Exit code of ls was $EC"
```

`while`, `until`, and `if` (next week) run a command, and interpret an exit code of 0 as True, and any other exit code as False. (And yes, `True` is a program that always returns exit code 0 and `False` is a program that always returns 1.)

## Loop over file contents

A very common `while` loop is over `read`ing a file.

```
$ cat names.txt
Malcom
Jayne
Wash
Simon

$ while read NAME; do echo "Hello $NAME"; done < names.txt
Hello Malcom
Hello Jayne
Hello Wash
Hello Simon
```

If we have a file called `names.txt` with 4 names (one per line), we can pipe this file into the `while` loop. Every iteration, it will call `read NAME`, the input comes from the file, line by line.

For the first iteration, it (sucessfully) reads the first name, "Malcom", into the variable `NAME` and, because the exit code is 0, it executes the loop. Next it reads the other names, each causing its own interation of the loop. When it reaches the end of the file, the input stream sends an end-of-file signal, the `read` command exits with a non-zero exit code, and the loop finishes.

## Answers:

### Exercise 1:

What is the difference between these 3 commands: (Try them out)

```
$ echo Hello $NAME
$ echo "Hello $NAME"
$ echo 'Hello $NAME'
```

(Hint: put some more spaces between "Hello" and "$NAME")

The difference between the single quotes and the other two is obvious: The single quotes tell the shell to not interpret the contents of this string, so the reference to the variable remains unresolved, an it prints `$NAME` instead of the name.

The difference between the first and the other two become clearer when more spaces are added between the two words: Without quotation marks, `bash` interprets the two words as two different arguments, and passes each one on to `echo` separately. `echo` then prints them with only a single space between them.

With quotation marks, `bash` recognises that the whole phrase, including _all_ spaces inbetween, is a single argument.

### Exercise 2:

What does the term `${#VARNAME}` do?

```
$ FOO=BAR
$ echo ${#FOO}
3
```

(Hint: change the variable `FOO` to different strings, see what changes with `${#FOO}`.)

This construct returns the number of characters in the string referenced by the variable `FOO`

### Exercise 3:

Write a loop to greet 4 people (select your own names)

Here is one version:

```
$ for NAME in Zoe Kaylee Inara River
> do
>     echo Hello $NAME
> done
Hello Zoe
Hello Kaylee
Hello Inara
Hello River
```
