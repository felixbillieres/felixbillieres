# 3️ File Inclusion

Back in the day when HTTP parameters were used to specify what is shown on the web page, which allows for building dynamic web pages an attacker could manipulate these parameters to display the content of any local file on the hosting server, leading to a [Local File Inclusion (LFI)](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web\_Application\_Security\_Testing/07-Input\_Validation\_Testing/11.1-Testing\_for\_Local\_File\_Inclusion) vulnerability.

LFI vulnerabilities can lead to source code disclosure, sensitive data exposure, and even remote code execution

according to the language used in the web app, `some of the functions only read the content of the specified files, while others also execute the specified files`.&#x20;

This is a significant difference to note, as executing files may allow us to execute functions and eventually lead to code execution
