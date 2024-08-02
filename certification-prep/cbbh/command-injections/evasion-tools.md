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
