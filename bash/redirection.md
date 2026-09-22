# Redirection

## File descriptors

A **file descriptor** is a non-negative integer used to identify I/O channels.

Every process starts with 3 file descriptors:
- `0`: `stdin` (standard input)
- `1`: `stdout` (standard output)
- `2`: `stderr` (standard error)

`stdin` reads input from the keyboard, `stdout` and `stderr` write in the terminal.

If a process opens other files, they will be identified with a file descriptor higher than 2.

**Redirection** changes how a process gets its input or output.

## Output redirection

- Redirect `stdout` to overwrite a file:

```bash
> file
```

- Redirect `stdout` to append to a file:

```bash
>> file
```

- Redirect `stderr` to overwrite a file:

```bash
2> file
```

- Redirect `stdout` and `stderr` to overwrite a file:

```bash
&> file
```

- Redirect `stdout` and `stderr` to append to a file:

```bash
&>> file
```

A common usecase of redirection of `stderr` is to discard error messages:

```bash
find . -name 'file' 2> /dev/null
```

## Pipeline

A pipeline is a sequence of one or more commands that are separated by the vertical bar character (`|`). A pipeline connects `stdout` of the first command to `stdin` of the next command:

```bash
ls | wc -l
```

## Pipeline with redirection

```bash
echo "hello" > file.txt | wc -c
```

This example would print `0`. This is because the shell sets up the pipeline first and then redirects the output, which means that the output of `echo` goes into `file.txt` and not to `wc`.

To solve this problem you can use `tee`, which copies its `stdin` to its `stdout` and also redirects its `stdout` to the given argument:

```bash
echo "hello" | tee file.txt | wc -c
```

This would print `hello` into `file.txt`, but also print in `stdout` the wordcount, which is `6`.

## Redirect `stderr` through a pipeline

If you want to pass `stderr` as input for the next command, `&>` and `&>>` cannot be used. Use `2>&1` and `2>>&1` instead:

```bash
find . -name 'file' 2>&1 | less
```
