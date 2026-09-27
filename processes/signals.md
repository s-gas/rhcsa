# Signals

A signal is an asynchronous notification to a process.

## Process management signals

| Signal | Name       | Definition                                                                        |
|--------|------------|-----------------------------------------------------------------------------------|
| `1`    | `SIGHUP`   | `Hangup`: reports termination of the terminal. Also requests configuration reload |
| `2`    | `SIGNINT`  | `Keyboard interrupt`: causes program termination via the keyboard (`Ctrl-C`)      |
| `3`    | `SIGQUIT`  | `Keyboard quit`: causes program termination and adds process dump via (`Ctrl-\`)  |
| `9`    | `SIGKILL`  | `Kill`: causes abrupt program termination.(unblockable)                           |
| `15`   | `SIGTERM`  | `Terminate`: causes program termination. Default signal for program termination   |
| `18`   | `SIGCONT`  | `Continue`: it resumes a program previously stopped. (unblockable)                |
| `19`   | `SIGSTOP`  | `Stop`: it suspends a program. (unblockable)                                      |
| `20`   | `SIGTSTP`  | `Keyboard stop`: it suspends a program via the keyboard (`Ctrl-Z`)                |

> Signal numbers vary between Linux hardware platforms, but signal names and meanings are standard. It is advised to use signal names rather than numbers when signaling.
