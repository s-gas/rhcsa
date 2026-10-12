# Bash scripts

A Bash shell script is an executable file that contains a list of commands.

The first line of the script is the **shebang** and it specifies the command interpreter:

```bash
[user@rocky ~]$ grep -v '^$' script.sh 
#!/bin/bash
echo "shebang!"
```

## Execute a script

A Shell script needs to have execute permissions to run. If the script is stored in a directory that is part of the `PATH` environment variable, it can be run directly with its name, otherwise the full path its needed:

```bash
[user@rocky ~]$ chmod u+x script.sh 
[user@rocky ~]$ which script.sh 
/usr/bin/which: no script.sh in (/home/user/.local/bin:/home/user/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin)
[user@rocky ~]$ ./script.sh 
shebang!
```

## Escape special characters

Some characters have special meaning. In order to treat them as regular characters you need to escape them either with `\`, single quotes or double quotes:

```bash
[user@rocky ~]$ cat script.sh 
#!/bin/bash

echo # this is treated as a comment
echo \# this is not treated as a comment
echo '# this is also not a comment'
echo "# this as well"
[user@rocky ~]$ ./script.sh 

# this is not treated as a comment
# this is also not a comment
# this as well
```

Remember, single quotes suppress all expansions, whilst single quotes allow variable and command expansion!

## For..in loop

Syntax:

```bash
for VARIABLE in LIST; do
    [commands]
done
```

Examples:

```bash
[user@rocky ~]$ bat script.sh 
───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: script.sh
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ #!/bin/bash
   2   │ 
   3   │ for name in foo bar baz; do
   4   │     echo $name
   5   │ done
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
[user@rocky ~]$ ./script.sh 
foo
bar
baz
```

```bash
[user@rocky ~]$ bat script.sh 
───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: script.sh
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ #!/bin/bash
   2   │ 
   3   │ for num in {0..9}; do
   4   │     echo $num
   5   │ done
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
[user@rocky ~]$ ./script.sh 
0
1
2
3
4
5
6
7
8
9
```

## C-Style for loop

You can also loop with the classic for loop:

```bash
[user@rocky ~]$ bat script.sh 
───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: script.sh
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ #!/bin/bash
   2   │ 
   3   │ for ((i=0; i < 10; i++)); do
   4   │     echo $i
   5   │ done
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
[user@rocky ~]$ ./script.sh 
0
1
2
3
4
5
6
7
8
9
```

> The double parenthesis are needed for performing arithmetics in bash.

## Exit codes

A script, like any other process, when it finishes its execution, it exits and return a status to the parent process that called it. A script can also be exited before the end of the script by using the `exit` keyword:

```bash
[user@rocky ~]$ cat script.sh 
#!/bin/bash

echo "This prints"

exit 2

echo "This doesn't print"
[user@rocky ~]$ ./script.sh 
This prints
[user@rocky ~]$ echo $?
2
```

## Compare values with test

The `test` command can be used to test logic for strings:

```bash
[user@rocky ~]$ test "hello" == "hello"; echo $?
0
[user@rocky ~]$ test "hello" == "hola"; echo $?
1
```

You can also use `test` to check if the condition given is true or false:

```bash
[user@rocky ~]$ test 1 -gt 0; echo $?
0
[user@rocky ~]$ test 1 -lt 0; echo $?
1
[user@rocky ~]$ test 0 -eq 0; echo $?
0
```

The `test` command can be used only with its command syntax:

```bash
[user@rocky ~]$ [[ "hello" == "hello" ]]; echo $?
0
```

> You could also use single parenthesis, which are POSIX compliant, but if you using Bash double parenthesis are generally better.

The space between the brackets are **mandatory** because the test parser tracks the spaces to split the elements!

## If/Then Conditional

Syntax:

```bash
if <CONDITION>; then
    <STATEMENT>
fi
```

Example:

```bash
[user@rocky ~]$ bat script.sh 
───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: script.sh
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ #!/bin/bash
   2   │ 
   3   │ day=$(date +%A)
   4   │ 
   5   │ for i in Saturday Sunday; do
   6   │     if [[ $i == $day ]]; then
   7   │         echo "It's the weekend!"
   8   │         exit 0
   9   │     fi
  10   │ done
  11   │ 
  12   │ echo "It's not the weekend!"
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
[user@rocky ~]$ ./script.sh 
It's not the weekend!
```

## If/Then/Elif/Then/Else Conditional

Syntax:

```bash
if <CONDITION>; then
    <STATEMENT>
elif <CONDITION>; then
    <STATEMENT>
else
    <STATEMENT>
fi
```

```bash
[user@rocky ~]$ bat script.sh 
───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: script.sh
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ #!/bin/bash
   2   │ 
   3   │ day=$(date +%A)
   4   │ 
   5   │ if [[ $day == "Saturday" ]]; then
   6   │     echo "It's the weekend"
   7   │ elif [[ $day == "Sunday" ]]; then
   8   │     echo "It's the weekend"
   9   │ else
  10   │     echo "It's a weekday"
  11   │ fi
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
[user@rocky ~]$ ./script.sh 
It's a weekday
```
