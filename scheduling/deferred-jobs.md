# Deferred User Jobs

Deferred user jobs are processes that are scheduled to run once in the future.

This can be achieved with the `at` command:

```bash
[user@rocky ~]$ at now +2min < script.sh 
warning: commands will be executed using /bin/sh
job 2 at Mon Oct 12 20:46:00 2026
```

The `at` command reads by default from `stdin`. The example uses input redirection, so that the command reads from the given script.

Also keep in mind that `at` does not print in `stdout`, it sends the output via email.

In order to run, the `atd` service needs to be currently active:

```bash
[user@rocky ~]$ systemctl status atd
● atd.service - Deferred execution scheduler
     Loaded: loaded (/usr/lib/systemd/system/atd.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-10-13 14:44:46 CEST; 15min ago
 Invocation: ad3464841ca14c1bb285fc587db0bf86
       Docs: man:atd(8)
   Main PID: 763 (atd)
      Tasks: 1 (limit: 10640)
     Memory: 332K (peak: 1.4M)
        CPU: 50ms
     CGroup: /system.slice/atd.service
             └─763 /usr/sbin/atd -f
```

## Inspect deferred jobs

To inspect the deferred jobs, use `atq`:

```bash
[user@rocky ~]$ atq
4	Tue Oct 13 14:47:00 2026 a user
```

`4` is the job number, `Tue Oct 13 14:47:00 2026` is the execution time, `a` is the job queue and `user` is the user owner.

## Remove deferred jobs

To remove a deferred job, use `atrm` with the job number:

```bash
[user@rocky ~]$ at now +2min < script.sh 
warning: commands will be executed using /bin/sh
job 8 at Tue Oct 13 14:58:00 2026
[user@rocky ~]$ atrm 8
[user@rocky ~]$ atq
```

## Usecase

A commond usecase for a deferred job is when some critical changes need to be made. A deferred job can be scheduled to restore the previous configuration.
