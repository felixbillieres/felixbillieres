# 🌠 Attacking Authentication

### &#x20;<a href="#lecture_heading" id="lecture_heading"></a>

Attacking authentication in penetration testing involves attempting to identify and exploit vulnerabilities related to the authentication mechanisms used by a system. Authentication is a crucial aspect of security, as it ensures that only authorized users can access resources or perform specific actions. Penetration testers aim to uncover weaknesses in authentication processes to help organizations strengthen their security posture.

<figure><img src="../../../../.gitbook/assets/image (214).png" alt=""><figcaption></figcaption></figure>

### Brute Force <a href="#lecture_heading" id="lecture_heading"></a>

<figure><img src="../../../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

we're given a username called jeremy

let's start burp suite

first we enter dumb credentials like "jeremy - password" and capture the request&#x20;

now send it to intruder by typing ctrl+u

<figure><img src="../../../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

now since you know jeremy is a valid username, double-click on the password and click this adds button to make it a variable for the brute force:

<figure><img src="../../../../.gitbook/assets/image (207).png" alt=""><figcaption></figcaption></figure>

now go in the payload tab and load a wordlist, make sure it's not too long or be aware of the time you're going to spend on the brute force:

<figure><img src="../../../../.gitbook/assets/image (208).png" alt=""><figcaption></figcaption></figure>

Now we just need to load our list and start the attack, a good thing for visibility is to filter by length, check for "big" length change. here we have some variations like 2133 or 2134 but we see the "letmein" made a 2127 length response and when we test it out it's the good password, success!

<figure><img src="../../../../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>

now you take the request in the proxy and paste it in a auth.txt file and replace the password variable with the word FUZZ:

<figure><img src="../../../../.gitbook/assets/image (210).png" alt=""><figcaption></figcaption></figure>

and run the command:

```
ffuf -request test.txt -request-proto http -w /usr/share/wordlists/SecLists/Passwords/xato-net-10-million-passwords-1000.txt
```

<figure><img src="../../../../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>

we need to do some filtering with the size ->

<figure><img src="../../../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

to filter with size add a&#x20;

```
-fs 1814
```

at the end of the command

* `ffuf`: This is the executable for the `ffuf` tool.
* `-request test.txt`: Specifies the file containing the raw HTTP request. This file typically includes the target URL, HTTP method, headers, and any other necessary information for the request. In your case, it's named "test.txt."
* `-request-proto http`: Specifies the protocol to be used in the requests. In this case, it's set to HTTP.
* `-w /usr/share/wordlists/SecLists/Passwords/xato-net-10-million-passwords-1000.txt`: Specifies the wordlist (password list) to be used for brute-forcing. The provided path is the location of the wordlist file on your system.
* `-fs 1814`: Sets the filter status code. In this example, `1814` is the status code used to filter responses. `ffuf` will only display responses with this specific status code.

### MFA <a href="#lecture_heading" id="lecture_heading"></a>

<figure><img src="../../../../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure>

first we type in the credentials and login

<figure><img src="../../../../.gitbook/assets/image (215).png" alt=""><figcaption></figcaption></figure>

we then request MFA number and login with it but as a pentester, you're already thinking that an int list of 1 to 999999 for the MFA generated code isn't too big for a bruteforce, let's try few stuff:

first we can imagine an easy thing, ok a mfa code is generatoed but in some cases the MFA int is valid for other users:

<figure><img src="../../../../.gitbook/assets/image (216).png" alt=""><figcaption></figcaption></figure>

you capture the mfa code with user jessamy and change the username with jeremey and now if you forward it in the proxy you'll be auth'd as jeremy. Prretty rare but good to test out&#x20;

and obviously you could try to bruteforce it but doing a payload that does brute force like this

&#x20;

<figure><img src="../../../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

but from 1 to 9999999 and you'll get it :tada:

### Challenge <a href="#lecture_heading" id="lecture_heading"></a>

<figure><img src="../../../../.gitbook/assets/image (368).png" alt=""><figcaption></figcaption></figure>

We have a tea page with a login page that locks after a few attempts, it's going to be complicated for a brute force. I test out jeremy and a random password and nothing. Then i test out admin and a password and an error pops. We can then imagine that the user jeremy does not exist and the admin user does.

<figure><img src="../../../../.gitbook/assets/image (381).png" alt=""><figcaption></figcaption></figure>

To start, let's capture a request and put it in a file .txt, then change the variables with FUZZUSER and FUZZPASS to be able to use ffuf on this file.

In a file add a list of 4 password&#x20;

```
1234
password
letmein
teashop
```

Then we're going to test out those passwords on a lot of username to always skip the blocking account parameter

```
┌──(kali㉿kali)-[~]
└─$ ffuf -request request.txt -request-proto http -mode clusterbomb -w pass.txt:FUZZPASS -w /usr/share/wordlists/SecLists/Usernames/top-usernames-shortlist.txt:FUZZUSER -fs 3376
```

after checking the content length of a bad request&#x20;

<figure><img src="../../../../.gitbook/assets/image (198).png" alt=""><figcaption></figcaption></figure>

we then filter out with the `-fs 3376` at the end of the request

<figure><img src="../../../../.gitbook/assets/image (199).png" alt=""><figcaption></figcaption></figure>

so technically, the request is going to try the 4 passwords on plenty of different usernames who dont exist, that wont error out but the content length will not be 3376 like a bad password request. So what we need to do is check the one content length that is different from those

here we found one, let's test it out:

`admin`

`letmein`

<figure><img src="../../../../.gitbook/assets/image (200).png" alt=""><figcaption><p><span data-gb-custom-inline data-tag="emoji" data-code="1f389">🎉</span></p></figcaption></figure>
