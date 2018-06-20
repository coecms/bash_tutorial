# Blocks and Scripts

At the end of last week, we looked at exit codes, the integer that a program returns to the calling shell.

An exit code of 0 generally means a success, anything else an error (or at least an abnormal result).

That is useful if we need to run a second command conditional on the successful completion of another.

For example you might only want to delete a file if it has been successfully transferred to a remote computer.

These commands can be connected by a double ampersand: `&&`:

```
$ cd ~/Desktop/data-shell
$ ls pizza.cfg && echo "Yummy! pizza"
pizza.cfg
Yummy! pizza
$ ls meatpie.cfg && echo "Yummy! meatpie"
ls: meatpie.cfg: No such file or directory
```

Or you might want to run a second command only if the first one fails:

```
$ ls pizza.cfg || echo "Sorry, no pizza"
pizza.cfg
$ ls meatpie.cfg || echo "Sorry, no meatpie"
ls: meatpie.cfg: No such file or directory
Sorry, no meatpie
```

There are better ways to just test whether a file exists.
The command `test` will give us a lot of options:

```
$ man test
```

The option `-e` looks whether a file exists.

```
$ test -e pizza.cfg
$ echo $?
0
$ test -e meatpie.cfg
$ echo $?
1
```

The command `test` does not write anything to the output, instead it only sets the exit code to 0 or 1 respectively.

Let's combine this with last week's variables and loops:

```
$ for FOOD in pizza meatpie
> do
>     test -e ${FOOD}.cfg && echo "Yummy! ${FOOD}" || echo "Sorry, no ${FOOD} today"
> done
Yummy! pizza
Sorry, no meatpie today
```

Going back to the man page again:

```
$ man test
```

See that there's another **SYNTAX**: `[ EXPRESSION ]`.
We can use this syntax as well:

```
$ FOOD=pizza
$ [ -e ${FOOD}.cfg ] && echo "Yummy! ${FOOD}!" || echo "Sorry, no ${FOOD} today"
Yummy! pizza!
$ FOOD=meatpie
$ [ -e ${FOOD}.cfg ] && echo "Yummy! ${FOOD}!" || echo "Sorry, no ${FOOD} today"
Sorry, no meatpie today.
```

Quick confession: I'm not following best practices here.
Say you have this:

```
$ FILE=pizza.cfg
$ [ -e $FILE ] && echo "Yummy!"
Yummy!
$ unset FILE
$ [ -e $FILE ] && echo "Yummy!"
Yummy!
```

This is not expected. `FILE` isn't set, so why does it suggest a file with no name exists?

Because `FILE` isn't set, this reverts to `[ -e ]` which tests whether `-e` is a string with at least one character, which it is.

To protect against that, it's good practise to put variables in double-quotes:

```
$ [ -e "$FILE" ] && echo "Yummy!"
```

But why the whole thing with square brackets? Because if. If what? No, because: `if`, the command `if`:

```
$ if [ -e "$FILE" ]
> then
>     cat $FILE
>     wc -l $FILE
> else
>     echo "$FILE does not exist"
> fi
 does not exist
$ FILE=pizza.cfg
$ if [ -e "$FILE" ]; then cat $FILE; wc -l $FILE; else echo "$FILE does not exist"; fi
```

The `else` part is optional.

## Scripts

This last command was already a lot of typing.
If I need to perform several operations in a specific order, and have to retype everything again, well ...
Did I say that programmers are lazy?

This is where scripts come in.

Let's go into the `molecules` subdirectory:

```
$ cd ~/Desktop/data-shell/molecules
$ ls
```

We already have `head` and `tail`, but what if we want to grab something from the middle?
Let's create a script called `middle.sh`.
We use the extension `.sh` to hint that this is a shell script:

```
$ nano middle.sh

head -n 15 octane.pdb | tail -n 5
```

The first command returns only the first 15 lines of the file, which is then piped into `tail` which selects the last 5 of these.
This leaves lines 11-15 of the file.

We can now execute the file:

```
$ bash middle.sh
```

But if we want the contents of a different file, we'd have to change the script.

So what we do instead is to fall back on the arguments:

```
$ nano middle.sh

head -n 15 "$1" | tail -n 5
```
`$1` refers to the first argument supplied to the script.
We put this in quotation marks for a similar reason as before.

Now we can give the script the file name from which we want the middle:

```
$ bash middle.sh octane.pdb
$ bash middle.sh pentane.pdb
```

### Exercise:

Use two more arguments to make the caller enter the **last** line to be output as argument 2, and the **number** of lines to be output as argument 3:

```
$ bash middle.sh octane 10 3
ATOM      6  C           1       1.892  -0.400   0.001  1.00  0.00
ATOM      7  C           1       3.113   0.429   0.414  1.00  0.00
ATOM      8  C           1       4.397  -0.374   0.199  1.00  0.00
```

So what if we have an unknown number of arguments?

Say our script wants to list the length (in lines) of all files in the arguments sorted?

Without a script, we'd do something like this:

```
$ wc -l *.pdb | sort -n
```

All arguments are stored in the variable `$@`:

```
$ nano sorted.sh

wc -l "$@" | sort -n

$ bash sorted.sh *.pdb ../creatures/*.dat
```

## Comments

When scripts become large, it can be hard to keep the overview.
For this reason, practically **all** languages incorporate comments.
Text that is meant only for the programmers, never to be executed itself.

`bash`, and most scripted languages, use the octothorpe, better known as the pound sign, or more recently the hashtag:
`#`

Unless the `#` is inside a string, `bash` will ignore it everything in the line behind it.

```
$ ls methane.pdb ethane.pdb # octane.pdb
methane.pdb ethane.pdb
```

We can use this to write some information about our script into the text itself:

```
$ nano sorted.sh

# Sort filenames by their length.
# Usage: bash sorted.sh <FILES>
wc -l "$@" | sort -n
```

## Making scripts executable

We still need to call `bash <script name>` -- how do we get rid of the need to type out `bash`?

2 Things:

First, we need to write into the script which interpreter should be used to interpret this script.
For this, we use what's called a *sha-bang*:

```
$ nano sorted.sh

#!/bin/bash
# Sort filenames by their length
# Usage: sorted.sh <FILES>
wc -l "$@" | sort -n
```

The sha-bang has to have exactly this form: It must be at the very beginning of the first line.
It must be a pound sign, followed by an exclamation mark, followed by the full path the the interpreter binary.

If you don't know where the binary is, use the `which` command:

```
$ which bash
/bin/bash
```

`bash` is practically always in this location

The second thing is to tell the operating system that this is an executable file.
We do this with the command:

```
$ chmod +x sorted.sh
```

Now we can run

```
$ ./sorted.sh *.pdb
```

Note the `./` before the script.
By default, the shell will not look in the local directory for the program to execute.
Therefore we have to give its path, absolute or relative. (Relative is easier in this case.)

### Exercise:

What do these special variables contain in a script:

| var |  content |
|-----|----|
| `$0` | ?  |
| `$#` | ?  |

## Solutions


### Exercise:

Use two more arguments to make the caller enter the **last** line to be output as argument 2, and the **number** of lines to be output as argument 3:

```
$ bash middle.sh octane 10 3
ATOM      6  C           1       1.892  -0.400   0.001  1.00  0.00
ATOM      7  C           1       3.113   0.429   0.414  1.00  0.00
ATOM      8  C           1       4.397  -0.374   0.199  1.00  0.00
```

```
head -n "$2" "$1" | tail -n "$3"
```

### Exercise:

What do these special variables contain in a script:

| var | content |
|---|---|
| `$0` | The name of the script  |
| `$#` | The number of arguments  |


## Thanks

I have to thank the people behind Software Carpentry for their work.
It would have been impossible to throw this course together so quickly without them.

Please have a look at https://software-carpentry.org/ for more information.
