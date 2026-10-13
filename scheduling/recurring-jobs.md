# Recurring Jobs

Recurring jobs are jobs that are scheduled to run repeatedly.

## Cron jobs

A way of scheduling recurring jobs is through the `crond` service, which is enabled and active by default:

```bash
[user@rocky ~]$ systemctl status crond
● crond.service - Command Scheduler
     Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-10-13 14:44:46 CEST; 19min ago
 Invocation: 2a425b1821264c569b1393d3773db187
   Main PID: 765 (crond)
      Tasks: 2 (limit: 10640)
     Memory: 1.1M (peak: 3.4M)
        CPU: 39ms
     CGroup: /system.slice/crond.service
             ├─ 765 /usr/sbin/crond -n
             └─2274 /usr/sbin/anacron -s
```

The `crond` service reads configuration files for each user and for the system. You can use `crontab` to manage scheduling jobs.

### User cron jobs

To list the jobs of the current user, use `crontab -l`:

```bash
[user@rocky ~]$ crontab -l
*/10 * * * * echo "hello" >> file.log
```

As the `at` command, also `crontab` doesn't print on `stdout`, but tries to email the user. If you want to print something somewhere else you need to redirect.

Every entry consist of 6 fields:
- minutes (`*/10`)
- hour (`*`)
- day (`*`)
- month (`*`)
- day of the week (`*`)
- command to execute (`echo "hello" >> file.log`)

This specific cron job will run every 10 minutes, because the star (`*`) means execute at every possible instance.

To modify a cron job, use `crontab -e`, which will open the `vim` editor with which you can edit the entries.

To delete all cron jobs for a specific user, run `crontab -r`:

```bash
[user@rocky ~]$ crontab -l
*/10 * * * * echo "hello" >> file.log
[user@rocky ~]$ crontab -r
Backup of user's previous crontab saved to /home/user/.cache/crontab/crontab.bak
[user@rocky ~]$ crontab -l
no crontab for user
```

### System cron jobs

For system administrators, it is better to schedule system-wide cron jobs, instead of user specific ones. This can be done by modifying `/etc/crontab` or adding a drop-in file into `/etc/cron.d/`. As every case where a drop-in directory exists, always prefer a drop-in file to the default file!

Compared to `crontab`, system-wide cron jobs have one extra field (7 in total), which is the user that runs the command:

```bash
[user@rocky /]$ cat /etc/crontab 
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

There are also `/etc/cron.daily/`, `/etc/cron.hourly/`, `/etc/cron.weekly/` and `/etc/cron.monthly/` which can contain scripts to be run at that interval. Remember that the scripts need to be executable!

## Anacron jobs

The `anacron` command uses the `run-parts` script to execute the daily, weekly and monthly scripts from `/etc/anacrontab`:

```bash
[user@rocky etc]$ cat /etc/anacrontab 
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1	5	cron.daily		nice run-parts /etc/cron.daily
7	25	cron.weekly		nice run-parts /etc/cron.weekly
@monthly 45	cron.monthly		nice run-parts /etc/cron.monthly
```

The difference between anacron and cron jobs is the fact that anacron guarantees that the scheduled jobs are run, even when the system is not available.

Also, as you can see from `/etc/anacrontab`, the syntax is different from cron jobs. Anacron has 4 fields:
- period in days: interval
- delay in minutes: time that `crond` needs to wait before running the job
- job-identifier: the name given to the log file stored in `/var/spool/anacron`
- command: command to run
