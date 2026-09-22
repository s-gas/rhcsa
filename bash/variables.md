# Shell variables

Shell variables are unique to a shell session. This means that these variables will not persist into a new shell session.

## Assign a value

Variable names can contain uppercase, lowercase, digits and underscore (`_`) and cannot start with a digit.

```bash
[user@localhost ~]$ NAME=john
[user@localhost ~]$ age=55
```

## View shell variables

To view the variables that are currently set, you can use `set`:

```bash
[user@localhost ~]$ set | less
```

## Remove a variable

To remove a variable:

```bash
[user@localhost ~]$ unset NAME
```

## Environment variables

A shell variable can be used only by the shell, an **environment variable**, on the other hand, can be used by any child of the shell.

To create an environment variable:

```bash
[user@localhost ~]$ export FAVORITE_COLOR=green
```

To convert a shell variable to an environment variable:

```bash
[user@localhost ~]$ NAME=john
[user@localhost ~]$ export NAME
```

To view the environment variables, you can use `env`:

```bash
[user@localhost ~]$ env | less
```

To unexport a variable without unsetting it:

```bash
[user@localhost ~]$ export -n NAME
```

## Uppercase or lowercase

By convention, shell variables and environment variables that are set by the shell are all uppercase. To avoid name collision, create variables with lowercase characters.

## Configuration scripts

### All users

To set variables available to all users, as user `root` add a `.sh` file to `/etc/profile.d`. The files inside this drop-in directory get sourced by `/etc/profile`, which runs for **login shells**.

### Specific user

To set variable available to a specific user, modify `~/.bashrc`. This script runs only for **non-login shells**, but is also sourced by `~/.bash_profile`, which runs for login shells:

```bash
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi
```

The dot (`.`) is an alias for `source`.

## Aliases

An alias is just another name for a command:

```bash
[user@localhost ~]$ alias ll="ls -l"
```

To remove an alias:

```bash
unalias ll
```
