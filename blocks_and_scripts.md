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
>     test -f ${FOOD}.cfg && echo "Yummy! ${FOOD}" || echo "Sorry, no ${FOOD} today"
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
$ [ -f ${FOOD}.cfg ] && echo "Yummy! ${FOOD}!" || echo "Sorry, no ${FOOD} today"
Yummy! pizza!
$ FOOD=meatpie
$ [ -f ${FOOD}.cfg ] && echo "Yummy! ${FOOD}!" || echo "Sorry, no ${FOOD} today"
Sorry, no meatpie today.
```

Quick confession: I'm not following best practices here.
Say you have this:

```
$ FILE=pizza.cfg
$ [ -f $FILE ] && echo "Yummy!"
Yummy!
$ unset FILE
$ [ -f $FILE ] && echo "Yummy!"
Yummy!
```

This is not expected. `FILE` isn't set, so why does it suggest a file with no name exists?

Because `FILE` isn't set, this reverts to `[ -f ]` which in this case returns exit code 0.

To protect against that, it's good practise to put variables in double-quotes:

```
$ [ -f "$FILE" ] && echo "Yummy!"
```

But why the whole thing with square brackets? Because if. If what? No, because: `if`, the command `if`:

```
$ if [ -f "$FILE" ]
> then
>     cat $FILE
>     wc -l $FILE
> else
>     echo "$FILE does not exist"
> fi
 does not exist
$ FILE=pizza.cfg
$ if [ -f "$FILE" ]; then cat $FILE; wc -l $FILE; else echo "$FILE does not exist"; fi
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
