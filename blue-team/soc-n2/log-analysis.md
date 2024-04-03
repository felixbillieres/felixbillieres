# 🌟 Log Analysis

### Types of Logs

* Application Logs: Messages from specific applications, providing insights into their status, errors, warnings, and other operational details.
* Audit Logs: Events, actions, and changes occurring within a system or application, providing a history of user activities and system behavior.
* Security Logs: Security-related events like logins, permission alterations, firewall activities, and other actions impacting system security.
* Server Logs: System logs, event logs, error logs, and access logs, each offering distinct information about server operations.
* System Logs: Kernel activities, system errors, boot sequences, and hardware status, aiding in diagnosing system issues.
* Network Logs: Communication and activity within a network, capturing information about events, connections, and data transfers.
* Database Logs: Activities within a database system, such as queries performed, actions, and updates.
* Web Server Logs: Requests processed by web servers, including URLs, source IP addresses, request types, response codes, and more.

<figure><img src="../../../.gitbook/assets/image (285).png" alt=""><figcaption></figcaption></figure>

### Common Log File Locations

```

    Web Servers:
        Nginx:
            Access Logs: /var/log/nginx/access.log
            Error Logs: /var/log/nginx/error.log
        Apache:
            Access Logs: /var/log/apache2/access.log
            Error Logs: /var/log/apache2/error.log

    Databases:
        MySQL:
            Error Logs: /var/log/mysql/error.log
        PostgreSQL:
            Error and Activity Logs: /var/log/postgresql/postgresql-{version}-main.log

    Web Applications:
        PHP:
            Error Logs: /var/log/php/error.log

    Operating Systems:
        Linux:
            General System Logs: /var/log/syslog
            Authentication Logs: /var/log/auth.log

    Firewalls and IDS/IPS:
        iptables:
            Firewall Logs: /var/log/iptables.log
        Snort:
            Snort Logs: /var/log/snort/

```

### Common Patterns - Abnormal User Behavior

* **Multiple failed login attempts**
  * Unusually high numbers of failed logins within a short time may indicate a brute-force attack.
* **Unusual login times**
  * Login events outside the user's typical access hours or patterns might signal unauthorized access or compromised accounts.
* **Geographic anomalies**
  * Login events from IP addresses in countries the user does not usually access can indicate potential account compromise or suspicious activity.
  * In addition, simultaneous logins from different geographic locations (or indications of impossible travel) may suggest account sharing or unauthorized access.
* **Frequent password changes**
  * Log events indicating that a user's password has been changed frequently in a short period may suggest an attempt to hide unauthorized access or take over an account.
* **Unusual user-agent strings**
  * In the context of HTTP traffic logs, requests from users with uncommon user-agent strings that deviate from their typical browser may indicate automated attacks or malicious activities.
  * For example, by default, the [Nmap scanner](https://tryhackme.com/room/furthernmap) will log a user agent containing "Nmap Scripting Engine." The [Hydra brute-forcing tool](https://tryhackme.com/room/hydra), by default, will include "(Hydra)" in its user-agent. These indicators can be useful in log files to detect potential malicious activity.

### Common Attack Signatures

**SQL Injection**

Suspicious SQL queries might contain unexpected characters, such as single quotes (`'`), comments (`--`, `#`), union statements (`UNION`), or time-based attacks (`WAITFOR DELAY`, `SLEEP()`). A useful SQLi payload list to reference can be found [here](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)

```log
10.10.61.21 - - [2023-08-02 15:27:42] "GET /products.php?q=books' UNION SELECT null, null, username, password, null FROM users-- HTTP/1.1" 200 3122
```

In the above example, an SQL injection attempt can be identified by the `' UNION SELECT` section of the `q=` query parameter. The attacker appears to have escaped the SQL query with the single quote and injected a union select statement to retrieve information from the `users` table in the database. Often, this payload may be URL-encoded, requiring an additional processing step to identify it efficiently.

**Cross-Site Scripting (XSS)**

To identify common XSS attack patterns, it is often helpful to look for log entries with unexpected or unusual input that includes script tags (`<script>`) and event handlers (`onmouseover`, `onclick`, `onerror`). A useful XSS payload list to reference can be found [here](https://github.com/payloadbox/xss-payload-list).   &#x20;

In the example below, a cross-site scripting attempt can be identified by the `<script>alert(1);</script>` payload inserted into the `search` parameter, which is a common testing method for XSS vulnerabilities.

```log
10.10.19.31 - - [2023-08-04 16:12:11] "GET /products.php?search=<script>alert(1);</script> HTTP/1.1" 200 5153
```

**Path Traversal**

To identify common traversal attack patterns, look for traversal sequence characters (`../` and `../../`) and indications of access to sensitive files (`/etc/passwd`, `/etc/shadow`). A useful directory traversal payload list to reference can be found [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Directory%20Traversal/README.md).

It is important to note, like with the above examples, that directory traversals are often URL encoded (or double URL encoded) to avoid detection by firewalls or monitoring tools. Because of this, `%2E` and `%2F` are useful URL-encoded characters to know as they refer to the `.` and `/` respectively.

```log
10.10.113.45 - - [2023-08-05 18:17:25] "GET /../../../../../etc/passwd HTTP/1.1" 200 505
```

### Command Line Analysis&#x20;

The `cat` command (short for "concatenate") is typically not the best approach for dealing with long log files.

The `less` command is an improvement over `cat` when dealing with larger files. It allows you to view the file's data page by page, providing a more convenient way to read through lengthy logs.

You can exit the command's output via the `q` key.

The `tail` command is specifically designed for viewing the end of files and is very useful for seeing a summary of recently generated events in the case of log files. The most common use of `tail` is coupled with the `-f` option, which allows you to "follow" the log file in real-time, as it continuously updates the terminal with new log entries as they are generated and written. This is extremely useful when monitoring logs for live events or real-time system behavior.

By default, `tail` will only display the last ten lines of the file. However, we can change this with the `-n` option and specify the number of lines we want to view.

example:

```log
           
user@tryhackme$ tail -f -n 5 apache.log
176.145.201.99 - - [31/Jul/2023:12:34:24 +0000] "GET /login.php HTTP/1.1" 200 1234 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.90 Safari/537.36"
104.76.29.88 - - [31/Jul/2023:12:34:23 +0000] "GET /index.php HTTP/1.1" 200 5678 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.81 Safari/537.36"
128.45.76.66 - - [31/Jul/2023:12:34:22 +0000] "GET /contact.php HTTP/1.1" 404 5678 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.54 Safari/537.36"
76.89.54.221 - - [31/Jul/2023:12:34:21 +0000] "GET /about.php HTTP/1.1" 200 1234 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.85 Safari/537.36"
145.76.33.201 - - [31/Jul/2023:12:34:20 +0000] "GET /login.php HTTP/1.1" 200 5678 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.90 Safari/537.36"
```

**Note**: The opposite of the `tail` command is `head`, which allows you to view the _first_ ten lines of a file by default and takes in the same arguments.

The output of `wc` provides information about the number of lines, words, and characters in a log file.

```log
user@tryhackme$ wc apache.log     
   70  1562 14305 apache.log
```

After running `wc` on `apache.log`, we can determine that the file contains 70 lines, 1562 individual words (separated by whitespace), and 14305 individual characters.

The `cut` command extracts specific columns (fields) from files based on specified delimiters.

If we want to extract all of the IP addresses in the file, we can use the `cut` command to specify a delimiter of a `space` character and only select the first field returned.

```log
user@tryhackme$ cut -d ' ' -f 1 apache.log
203.0.113.42
120.54.86.23
185.76.230.45
201.39.104.77
112.76.89.56
211.87.186.35
156.98.34.12
202.176.73.99
122.65.187.55
77.188.103.244
189.76.230.44
153.47.106.221
200.89.134.22
```

The above command will return a list of every IP address in the log file. Expanding on this, we can change the field number to `-f 7` to extract the URLs and `-f 9` to extract the HTTP status codes.

```log
root@ip-10-10-65-240:~/Rooms/introloganalysis/task6# cut -d ' ' -f 1,7,9 apache.log 
203.0.113.42 /index.php 200
120.54.86.23 /contact.php 404
185.76.230.45 /about.php 200
201.39.104.77 /login.php 200
112.76.89.56 /index.php 200
211.87.186.35 /contact.php 200
156.98.34.12 /about.php 404
202.176.73.99 /login.php 200
122.65.187.55 /index.php 200
77.188.103.244 /contact.php 200
```

You can easily play with all that. Let's say I want to get the last 5 logs and get their IP addresses:

```log
root@ip-10-10-65-240:~/Rooms/introloganalysis/task6# tail -f -n 5 apache.log; cut -d ' ' -f 1 apache.log  
176.145.201.99 - - [31/Jul/2023:12:34:24 +0000] "GET /login.php HTTP/1.1" 200 1234 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.90 Safari/537.36"
104.76.29.88 - - [31/Jul/2023:12:34:23 +0000] "GET /index.php HTTP/1.1" 200 5678 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.81 Safari/537.36"
128.45.76.66 - - [31/Jul/2023:12:34:22 +0000] "GET /contact.php HTTP/1.1" 404 5678 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.54 Safari/537.36"
76.89.54.221 - - [31/Jul/2023:12:34:21 +0000] "GET /about.php HTTP/1.1" 200 1234 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.85 Safari/537.36"
145.76.33.201 - - [31/Jul/2023:12:34:20 +0000] "GET /login.php HTTP/1.1" 200 5678 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.90 Safari/537.36"
```

The `sort` command arranges the data in files in ascending or descending order based on specific criteria. This can be crucial for identifying patterns, trends, or outliers in our log data. It is also common to combine the _output_ of another command (cut, for example) and use it as the _input_ of the sort command using the pipe `|` redirection character.

For example, to sort the list of returned IP addresses from the above `cut` command, we can run:

```
user@tryhackme$ cut -d ' ' -f 1 apache.log | sort -n
76.89.54.221
76.89.54.221
76.89.54.221
76.89.54.221
76.89.54.221
...
```

`-n` option to sort _numerically_. If we want to reverse the order, we can add the `-r` option.

The `uniq` command identifies and removes adjacent duplicate lines from sorted input.

The `uniq` command is often combined with the `sort` command to **sort** the data before removing the duplicate entries.

For example, the output of the `sort` command we ran above contains a few duplicate IP addresses, which is easier to spot when the data is sorted numerically. To remove these repeatedly extracted IPs from the list, we can run:

```
user@tryhackme$ cut -d ' ' -f 1 apache.log | sort -n -r | uniq
221.90.64.76
211.87.186.35
203.78.122.88
203.64.78.90
203.0.113.42
202.176.73.99
201.39.104.77
200.89.134.22
192.168.45.99
```

the `-c` option to output unique lines and prepend the count of occurrences for each line. This can be very useful for quickly determining IP addresses with unusually high traffic.

```
user@tryhackme$ cut -d ' ' -f 1 apache.log | sort -n -r | uniq -c
      1 221.90.64.76
      1 211.87.186.35
      1 203.78.122.88
      6 203.64.78.90
      1 203.0.113.42
```

The `grep` allows you to search for specific patterns or regular expressions within files or streams of text.

```
user@tryhackme$ grep "admin" apache.log
145.76.33.201 - - [31/Jul/2023:12:34:54 +0000] "GET /admin.php HTTP/1.1" 200 4321 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.330 Safari/537.36"
```

Like the `uniq -c` command, we can append the `-c` option to `grep` to count the entries matching the search criteria.       &#x20;

```
user@tryhackme$ grep -c "admin" apache.log
1
```

For the `awk` command, a common use case, is conditional actions based on specific field values. For example, to print log entries where the HTTP response code is greater than or equal to `400` (which would indicate HTTP error statuses), we can use the following command:

```log
user@tryhackme$ awk '$9 >= 400' apache.log
120.54.86.23 - - [31/Jul/2023:12:34:57 +0000] "GET /contact.php HTTP/1.1" 404 5678 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36"
156.98.34.12 - - [31/Jul/2023:12:35:02 +0000] "GET /about.php HTTP/1.1" 404 5678 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.85 Safari/537.36"
189.76.230.44 - - [31/Jul/2023:12:35:06 +0000] "GET /about.php HTTP/1.1" 404 1234 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.170 Safari/537.36"
```

In this case, we're using the `$9` field (which in this log example refers to the HTTP status codes), requiring it to be greater than or equal to `400`.

This only scratches the surface of the power of these commands, and it is highly encouraged to read more about their options and use cases [here](https://www.theunixschool.com/p/awk-sed.html).

Questions;

_**In the apache.log file, how many total HTTP 200 responses were logged?**_

```
root@ip-10-10-65-240:~/Rooms/introloganalysis/task6# grep "HTTP/1.1\" 200" apache.log | wc -l
52

```

_**In the apache.log file, which IP address generated the most traffic?**_

```
root@ip-10-10-65-240:~/Rooms/introloganalysis/task6# awk '{print $1}' apache.log | sort | uniq -c | sort -nr 
      8 145.76.33.201
      6 76.89.54.221
      6 203.64.78.90
      6 176.145.201.99
      6 128.45.76.66
      6 110.122.65.76
      6 104.76.29.88
      2 178.53.64.100
      1 99.76.122.65
      1 77.188.103.244
      1 221.90.64.76
      ...
      ...
```

* `awk '{print $1}'`: This extracts the first field (IP address) from each line in the log file.
* `sort`: This sorts the IP addresses alphabetically.
* `uniq -c`: This counts the occurrences of each unique IP address.
* `sort -nr`: This sorts the counts in descending order.
* `head -1`: This retrieves the top IP address with the highest count.

_**What is the complete timestamp of the entry where 110.122.65.76 accessed /login.php?**_

```
root@ip-10-10-65-240:~/Rooms/introloganalysis/task6# grep "110.122.65.76" apache.log 
110.122.65.76 - - [31/Jul/2023:12:35:02 +0000] "GET /contact.php HTTP/1.1" 200 1234 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.210 Safari/537.36"
110.122.65.76 - - [31/Jul/2023:12:34:54 +0000] "GET /contact.php HTTP/1.1" 200 7890 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.330 Safari/537.36"
110.122.65.76 - - [31/Jul/2023:12:34:47 +0000] "GET /index.php HTTP/1.1" 200 9876 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.230 Safari/537.36"
110.122.65.76 - - [31/Jul/2023:12:34:40 +0000] "GET /login.php HTTP/1.1" 200 9876 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.110 Safari/537.36"

```

### _Regex_ for grep

Regular expressions, abbreviated as _regex_, are an invaluable way to define patterns for searching, matching, and manipulating text data. Regular expression patterns are constructed using a combination of special characters that represent matching rules and are supported in many programming languages, text editors, and software.

This log file contains log entries from a blog site. The site is structured so that each blog post has its unique ID, fetched from the database dynamically through the `post` URL parameter. If we are only interested in the specific blog posts with an ID between 10-19, we can run the following `grep` regular expression pattern on the log file:

```
root@ip-10-10-190-128:~/Rooms/introloganalysis/task7/regex# grep -E 'post=1[0-9]' apache-ex2.log 
    203.0.113.1 - - [02/Aug/2023:10:15:23 +0000] "GET /blog.php?post=12 HTTP/1.1" 200 - "Mozilla/5.0"
    100.22.189.54 - - [03/Aug/2023:12:48:43 +0000] "GET /blog.php?post=14 HTTP/1.1" 200 - "Mozilla/5.0"
    34.210.98.12 - - [03/Aug/2023:15:30:56 +0000] "GET /blog.php?post=11 HTTP/1.1" 200 - "Mozilla/5.0"
    102.210.76.44 - - [04/Aug/2023:19:26:29 +0000] "GET /blog.php?post=16 HTTP/1.1" 200 - "Mozilla/5.0"
    98.88.76.103 - - [05/Aug/2023:17:56:33 +0000] "GET /blog.php?post=13 HTTP/1.1" 200 - "Mozilla/5.0"
...
...
```

`-E` option signifies that we are searching on a pattern rather than just a string

### _Regex_ for Log Parsing

Let's say we have this raw log:

```
126.47.40.189 - - [28/Jul/2023:15:30:45 +0000] "GET /admin.php HTTP/1.1" 200 1275 "" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.9999.999 Safari/537.36"
```

From a security standpoint, several fields here would be beneficial to extract into an SIEM for visualization. Some of these include:

* The IP address
* The timestamp
* The HTTP method (POST, GET, or PUT, for example)
* The URL
* The user-agent

[RegExr](https://regexr.com) is an online tool to help teach, build, and test regular expression patterns.

if we want to extract just the remote IP address from this log, we can think about the structure of the IP address logically. The IP address is the log entry's first part, consisting of four octets separated by periods. We can use the following pattern:

`\b([0-9]{1,3}\.){3}[0-9]{1,3}\b`

Let's break down the components of this regular expression:

1. `\b`: This is a word boundary anchor. It asserts a position where a word (sequence of word characters) starts or ends. In this context, it ensures that the pattern is matched as a whole word and not as part of a larger word.
2.  `([0-9]{1,3}\.){3}`: This part is a group `( ... )` that matches three occurrences `{3}` of the following pattern:

    * `[0-9]{1,3}`: This matches one to three digits. The `{1,3}` indicates that the preceding digit pattern should occur between 1 and 3 times.
    * `\.`: This matches a literal dot (period). The backslash `\` is used to escape the dot because in regular expressions, a plain dot has a special meaning (matches any character), and we want to match a literal dot.

    Therefore, `([0-9]{1,3}\.){3}` matches three groups of one to three digits followed by a dot, representing the first three octets of an IPv4 address.
3. `[0-9]{1,3}`: This matches the last octet of the IPv4 address, which consists of one to three digits.
4. `\b`: Another word boundary anchor to ensure the whole pattern is matched as a complete word.

<figure><img src="../../../.gitbook/assets/image (331).png" alt=""><figcaption></figcaption></figure>

### Logstash and Grok

Grok is a powerful Logstash plugin that enables you to parse unstructured log data into something structured and searchable.

More info on Grok and its use within the Elastic Stack can be found in the Elastic documentation [here](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html).

We can use the pattern we previously created to successfully extract IPv4 addresses from our log file and process them into a custom field before they are sent to an SIEM. In an Elastic Stack scenario, we can add a filter using the `Grok` plugin within our Logstash configuration file to achieve this.

imagine a logstash.conf file:

```
input {
  ...
}

filter {
  grok {
    match => { "message" => "(?<ipv4_address>\b([0-9]{1,3}\.){3}[0-9]{1,3}\b)" }
  }
}

output {
  ...
}
```

In the example above we extract IPv4 addresses from the "message" field of incoming log events. The extracted values will be added under the custom "ipv4\_addresses" field name we defined.

[the official Grok documentation](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html)

### Regex with CyberChef

CyberChef is a powerful tool in an analyst's toolkit.

Some key features include:

* Encoding and decoding data
* Encryption and hashing algorithms
* Data analysis, such as parsing log files and extracting data
* And many more!

<figure><img src="../../../.gitbook/assets/image (332).png" alt=""><figcaption></figcaption></figure>

By selecting the "List matches" filter on the "Output format" of the operation, we can remove all of the noise from the log and output solely the IP addresses.

#### Uploading Files in CyberChef

<figure><img src="../../../.gitbook/assets/image (333).png" alt=""><figcaption></figcaption></figure>

A request was made that is encoded in base64. What is the decoded value?

<figure><img src="../../../.gitbook/assets/image (334).png" alt=""><figcaption></figcaption></figure>

1. `(POST|GET)`: This is a capturing group that matches either "POST" or "GET". The vertical bar `|` acts as an OR operator, allowing either "POST" or "GET" to be matched. This part of the expression is case-sensitive, so it will only match "POST" or "GET" in uppercase.
2. `/`: This matches a forward slash character.
3. `.*`: This part matches any sequence of characters (except for a newline). The dot `.` matches any character, and the asterisk `*` means zero or more occurrences of the preceding character. So, `.*` matches any characters (including none).
4. `=`: This matches the equal sign character.

_Using CyberChef, decode the file named "encodedflag.txt" and use regex to extract by MAC address. What is the extracted value?_

<figure><img src="../../../.gitbook/assets/image (335).png" alt=""><figcaption><p>Fake MAC adresses</p></figcaption></figure>

There are pre-defined expressions to help us find a valid MAC address

<figure><img src="../../../.gitbook/assets/image (336).png" alt=""><figcaption></figcaption></figure>

### Sigma

[Sigma](https://github.com/SigmaHQ/sigma) is a highly flexible open-source tool that describes log events in a structured format. Sigma can be used to find entries in log files using pattern matching. Sigma is used to:

1. Detect events in log files
2. Create SIEM searches
3. Identify threats

<figure><img src="../../../.gitbook/assets/image (337).png" alt=""><figcaption></figcaption></figure>

Sigma uses the YAML syntax for its rules. This task will demonstrate Sigma being used to detect failed login events in SSH.

Simple Sigma Rule:

```yaml
title: Failed SSH Logins
description: Searches sshd logs for failed SSH login attempts
status: experimental
author: CMNatic
logsource: 
    product: linux
    service: sshd

detection:
    selection:
        type: 'sshd'
        a0|contains: 'Failed'
        a1|contains: 'Illegal'
    condition: selection
falsepositives:
    - Users forgetting or mistyping their credentials
level: medium
```

In this Sigma rule:

| Key            | Value                                                       | Description                                                                                                                               |
| -------------- | ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| title          | Failed SSH Logins                                           | <p>This title outlines the purpose of the Sigma rule.<br></p>                                                                             |
| description    | <p>Searches sshd logs for failed SSH login attempts<br></p> | <p>This key provides a description that expands on the title.<br></p>                                                                     |
| status         | experimental                                                | <p>This key explains the status of the rule. For example, "experimental" means that further testing or improvements must be done.<br></p> |
| author         | CMNatic                                                     | <p>The person who wrote the rule.<br></p>                                                                                                 |
| logsource      | <p>product: linux<br>service: sshd</p>                      | <p>Where can the log files that contain the data that we're looking for be found?<br></p>                                                 |
| detection      | <p>sshd<br></p>                                             | <p>This key lists what the Sigma rule is looking to find.<br></p>                                                                         |
| a0\|contains   | a0\|contains: 'Failed'                                      | <p>In this case, look for all entries with "Failed".<br></p>                                                                              |
| a1\|contains   | a1\|contains: 'Illegal'                                     | <p>In this case, look for all entries with "Illegal".<br></p>                                                                             |
| falsepositives | <p>Users forgetting or mistyping their credentials<br></p>  | List cases where this entry may be present but doesn't necessarily indicate malicious behavior.                                           |

### Yara

Yara is a YAML-formatted tool that identifies information based on binary and textual patterns (such as hexadecimal and strings). While it is usually used in malware analysis, Yara is extremely effective in log analysis.

Let's look at this example Yara rule called "IPFinder". This YARA rule uses regex to search for any IPV4 addresses. If the log file we are analyzing contains an IP address, YARA will flag it:

```
rule IPFinder {
    meta:
        author = "CMNatic"
    strings:
        $ip = /([0-9]{1,3}\.){3}[0-9]{1,3}/ wide ascii
 
    condition:
        $ip
}
```

| Key       | Example                                           | Description                                                                                                                    |
| --------- | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| rule      | <p>IPFinder<br></p>                               | <p>This key names the rule.<br></p>                                                                                            |
| meta      | author                                            | <p>This key contains metadata. For example, in this case, it is the name of the rule's author.<br></p>                         |
| strings   | $ip = /(\[0-9]{1,3}\\.){3}\[0-9]{1,3}/ wide ascii | <p>This key contains the values that YARA should look for. In this case, it is using REGEX to look for IPV4 addresses.<br></p> |
| condition | $ip                                               | If the variable $ip is detected, then the rule should trigger.                                                                 |
