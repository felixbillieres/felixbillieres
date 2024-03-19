# 🔑 Passwords & File Permissions

<figure><img src="../../../../.gitbook/assets/image (303).png" alt=""><figcaption></figcaption></figure>

## Stored Passwords <a href="#lecture_heading" id="lecture_heading"></a>

### Automatic research:

Reading linpeas' ([https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)) code is a good start, it looks for several files that may contain passwords. Another interesting tool you can use is Lazagne ([https://github.com/AlessandroZ/LaZagne](https://github.com/AlessandroZ/LaZagne)): an open source application that allows you to retrieve many passwords stored on your local computer in Windows, Linux and Mac.

### Manual research:

```
grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null
find . -type f -exec grep -i -I "PASSWORD" {} /dev/null \;
```

Will highlight all the occurrences of the PASSWORD word, good to modify with "pass" "passw" etc...

## Weak File Permissions <a href="#lecture_heading" id="lecture_heading"></a>

start by looking at your permissions for 2 very important files:

```
ls -al /etc/passwd
```

```
ls -al /etc/shadow
```

<figure><img src="../../../../.gitbook/assets/image (300).png" alt=""><figcaption></figcaption></figure>

Normally, the `/etc/passwd` file has permissions like this: `-rw-r--r--`. But, the shadow file is extra cautious and doesn't allow anyone to directly read it.

In some situations, you might need information from both the shadow and passwd files. To do this, you can create separate files containing the details from each file. Tools like `unshadow` can then be used to combine this information for further study or use.

The `unshadow` tool in Linux is used to combine the content of the `/etc/passwd` file (which contains user account information) and the `/etc/shadow` file (which contains encrypted password information) into a format suitable for further password cracking or analysis. The `unshadow` tool helps to simplify the process of extracting password-related information for use with password cracking tools like John the Ripper.

Here's a brief overview of how it works:

1. **/etc/passwd:** This file contains basic user account information, such as usernames, user IDs (UIDs), group IDs (GIDs), home directories, and shell types.
2. **/etc/shadow:** This file contains the encrypted password information (hashes) and other security-related data for user accounts.

<figure><img src="../../../../.gitbook/assets/image (301).png" alt=""><figcaption></figcaption></figure>

* **Prepare the Unshadow File:**
  * First, we extract the necessary information from the `/etc/passwd` and `/etc/shadow` files.
  * This combined information is stored in a file. Let's call it `password-file`.

```
root:$6$Tb/euwmK$OXA.dwMeOAcopwBl68boTG5zi65wIHsc84OWAIye5VITLLtVlaXvRDJXET..it8r.jbrlpfZeMdwD:$

TCM:$6$hDHLpYuo$El6r99ivR20zrEPUnujk/DgKieYIuqvf9V7M.6t6IZzxpwxGIvhqTwciEw16y/B.7ZrxVk1LOHmVb/:$
```

so we have a proper output of our two users with the hash of their passwords

you can look for the hash type yourself:

we see our hashes start with $6$

if we go to a tool such as [https://hashcat.net/wiki/doku.php?id=example\_hashes](https://hashcat.net/wiki/doku.php?id=example\_hashes) we easily find the ype and the mode number for the hashcat command

<figure><img src="../../../../.gitbook/assets/image (302).png" alt=""><figcaption></figcaption></figure>

To crack an unshadow file with SHA-512 hashed passwords using Hashcat, you would need to use the appropriate hash mode for SHA-512. The hash mode for SHA-512 is 1800 in Hashcat.

Assuming your unshadow file is named `password-file` and contains entries in the format `username:hashed_password:other_info_from_passwd_file:other_info_from_shadow_file`, you can use the following Hashcat command:

```bash
hashcat -m 1800 -a 0 -o cracked-passwords.txt password-file wordlist.txt
```

Explanation of the options:

* `-m 1800`: Specifies the hash mode for SHA-512.
* `-a 0`: Specifies the attack mode (straight/dictionary attack).
* `-o cracked-passwords.txt`: Specifies the output file for cracked passwords.
* `password-file`: The input unshadow file containing hashed passwords.
* `wordlist.txt`: The path to a wordlist containing potential passwords. You need to replace this with the path to your actual wordlist.

Make sure to replace `wordlist.txt` with the actual path to a suitable wordlist for your password cracking attempt. Hashcat will attempt to crack the hashed passwords using the words from the provided wordlist.
