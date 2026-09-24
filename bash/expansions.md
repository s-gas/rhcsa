# Bash expansions

## Brace expansion

It generates strings of characters.

Brace expansion with comma-separated list of strings:

```bash
[user@localhost ~]$ echo letter{A,B,C,D,E}.txt
letterA.txt letterB.txt letterC.txt letterD.txt letterE.txt
```

Brace expansion with sequence:

```bash
[user@localhost ~]$ echo file{1..5}.txt
file1.txt file2.txt file3.txt file4.txt file5.txt
```

A good usecase could be to create a backup file from an existing one:

```bash
[user@localhost ~]$ cp file{,.backup}
```

## Tilde expansion

The tilde character (`~`) expands to the current's user home directory.
If followed by a string without a slash (`/`), the string is interpreted as the username:

```bash
[user@localhost ~]$ cd ~user
```

## Variable expansion

It converts a variable name with its value:

```bash
[user@localhost ~]$ USER=john
[user@localhost ~]$ echo $USER
john
```

Use curly braces to separate the variable name from other characters:

```bash
[user@localhost ~]$ echo ${USER}.hello
john.hello
```

## Command substition

It replaces a command with its output:

```bash
[user@localhost ~]$ echo "Date: $(date +%F)"
Date: 2026-09-22
```

## Pathname Expansion (globbing)

It expands a pattern into a list of files.

### Patterns

| Pattern | Matches |
|---------|---------|
| `*`         | Any string of zero or more characters |
| `?`         | Any single character |
| `[abc..]`   | Any character in the enclosed class |
| `[!abc..]`  | Any character NOT in the class |
