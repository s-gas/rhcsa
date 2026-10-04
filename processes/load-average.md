# Load average

Load average represents the perceived system load for a period of time.

## Load average calculation

The load number is obtained by calculating the average of the number of process that are in one of the following states:
- running or runnable state (`R`)
- uninterruptible sleep (`D`): waiting for disk or network I/O

Other UNIX systems consider only running processes to calculate the load average. Linux also includes disk or network usage, because the high usage of these resources can significantly impact system performance as CPU load.
