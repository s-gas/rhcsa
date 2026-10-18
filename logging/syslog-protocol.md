# Syslog Protocol

Syslog files are logging files that follow the syslog protocol, which is supported both by `systemd-journald` and `rsyslog`.

They are categorized by:
- facility: the sub-system that generated the log
- priority: severity of the log

## Facilities

| Code | Facility | Description     |
|------|----------|-----------------|
| `0`  | `kern`   | kernel messages |
| `1`  | `user`   | user messages   |
| `2`  | `mail`   | mail messages   |
| `3`  | `daemon` | daemon messages |
| ...  | ...      | ...             |
| `16`-`23` | `local0`- `local7` | custom messages |

## Priorities

| Code | Priority   | Description  |
|------|------------|--------------|
| `0`  | `emerg`    | system is unusable |
| `1`  | `alert`    | action must be taken ASAP |
| `2`  | `crit`     | critical condition |
| `3`  | `err`      | non-critical error condition |
| `4`  | `warning`  | warning condition |
| `5`  | `notice`   | significant event |
| `6`  | `info`     | informational event |
| `7`  | `debug`    | debugging-level message |
