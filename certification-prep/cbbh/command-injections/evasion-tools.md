# 😘 Evasion Tools

### Linux (Bashfuscator)

[Bashfuscator](https://github.com/Bashfuscator/Bashfuscator)

```shell-session
ElFelixi0@htb[/htb]$ git clone https://github.com/Bashfuscator/Bashfuscator
ElFelixi0@htb[/htb]$ cd Bashfuscator
ElFelixi0@htb[/htb]$ pip3 install setuptools==65
ElFelixi0@htb[/htb]$ python3 setup.py install --user
```

We can start off with the simple command

```shell-session
./bashfuscator -c 'cat /etc/passwd'
```

But it will be a random payload with length ranging from a few hundred characters to over a million characters

So let's try producing a shorter payload:

```shell-session
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
```

and to test out the payload we can use `bash -c ''`

```shell-session
ElFelixi0@htb[/htb]$ bash -c 'eval "$(W0=(w \  t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"'

root:x:0:0:root:/root:/bin/bash
...SNIP...
```

### Windows (DOSfuscation)

for Windows we can use a tool called [DOSfuscation](https://github.com/danielbohannon/Invoke-DOSfuscation). Unlike `Bashfuscator`, this is an interactive tool, as we run it once and interact with it to get the desired obfuscated command.

```powershell
PS C:\htb> git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
PS C:\htb> cd Invoke-DOSfuscation
PS C:\htb> Import-Module .\Invoke-DOSfuscation.psd1
PS C:\htb> Invoke-DOSfuscation
```

And then we get to use the tool like this:

```powershell
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
Invoke-DOSfuscation> encoding
Invoke-DOSfuscation\Encoding> 1

...SNIP...
Result:
typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\h%TMP:~-13,-12%b-stu%SystemRoot:~-4,-3%ent%TMP:~-19,-18%%ALLUSERSPROFILE:~-4,-3%esktop\flag.%TMP:~-13,-12%xt
```

and to test the output in our powershell cmd:

```cmd-session
C:\htb> typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\h%TMP:~-13,-12%b-stu%SystemRoot:~-4,-3%ent%TMP:~-19,-18%%ALLUSERSPROFILE:~-4,-3%esktop\flag.%TMP:~-13,-12%xt

test_flag
```
