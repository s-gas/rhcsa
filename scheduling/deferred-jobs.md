# Deferred User Jobs

Deferred user jobs are processes that are scheduled to run once in the future.

This can be achieved with the `at` command:

```bash
[user@rocky ~]$ at now +2min < script.sh 
warning: commands will be executed using /bin/sh
job 2 at Mon Oct 12 20:46:00 2026
```

## Inspect deferred jobs
