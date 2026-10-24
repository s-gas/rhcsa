# Regex

Regex stands for **regular expressions** and consist of a series of character that specifies a match pattern in a text. Many programming languages and also Linux commands can use regex.

There are 2 types of regular expressions, basic regex and extended regex. The difference between them is the way the handle some special characters.

## Regex and `grep`

One of the commands that supports regex is `grep`, which prints the lines that match the given pattern (basic regex):

```bash
[user@rocky ~]$ grep 'day' script.sh 
day=$(date +%A)
if [[ $day == "Saturday" ]]; then
elif [[ $day == "Sunday" ]]; then
	echo "It's a weekday"
```

The example prints all the lines of `script.sh` that contain `#!`.

To match only at the beginning of the line, you have to use the caret (`^`):

```bash
[user@rocky ~]$ grep '^day' script.sh 
day=$(date +%A)
```

To match only the end of the line, you have to use the dollar sign (`$`):

```bash
[user@rocky ~]$ grep '/usr/sbin/nologin$' /etc/passwd
bin:x:1:1:bin:/bin:/usr/sbin/nologin
daemon:x:2:2:daemon:/sbin:/usr/sbin/nologin
adm:x:3:4:adm:/var/adm:/usr/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:12:mail:/var/spool/mail:/usr/sbin/nologin
operator:x:11:0:operator:/root:/usr/sbin/nologin
games:x:12:100:games:/usr/games:/usr/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/usr/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/usr/sbin/nologin
tss:x:59:59:Account used for TPM access:/:/usr/sbin/nologin
dbus:x:81:81:System Message Bus:/:/usr/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
systemd-coredump:x:996:996:systemd Core Dumper:/:/usr/sbin/nologin
```

You can also use wildcards, one of these is the dot (`.`), which matches any single character:

```bash
[user@rocky ~]$ grep 'week...' script.sh 
	echo "It's the weekend"
	echo "It's the weekend"
	echo "It's a weekday"
```

You can also combine the dot (`.`) with the asterisk (`*`) to say that the character can be present any number of times (including 0 times):

```bash
[root@localhost ~]# grep -i 'week.*' file.txt 
echo "Weekday"
echo "Weekend"
echo "Twice a week"
```

A very useful usecase of `grep` with a regex is to exclude lines that are comments or are empty:

```bash
[user@rocky ~]$ grep -v -e '^#' -e '^$' script.sh 
day=$(date +%A)
if [[ $day == "Saturday" ]]; then
	echo "It's the weekend"
elif [[ $day == "Sunday" ]]; then
	echo "It's the weekend"
else
	echo "It's a weekday"
fi
```

In this example, the `-v` flag is used to "reverse" the output, and the `-e` flag to check multiple patterns.

Another useful regex is **the pirate** (`(/.*)?`), which is normally used to match a directory and its children. The parenthesis create a group and the question mark (`?`) makes the group optional:

```bash
[root@localhost ~]# semanage fcontext -l | grep '(/.*)?' | tail -5
/var/www/svn/hooks(/.*)?                           all files          system_u:object_r:httpd_sys_script_exec_t:s0 
/var/www/usage(/.*)?                               all files          system_u:object_r:webalizer_rw_content_t:s0 
/var/www/wiki[0-9]?(/.*)?                          all files          system_u:object_r:mediawiki_rw_content_t:s0 
/var/yp(/.*)?                                      all files          system_u:object_r:var_yp_t:s0 
/xen(/.*)?                                         all files          system_u:object_r:xen_image_t:s0 
```

The example shows how this regex is used by SELinux in its context label policy.
