# 🦹‍♂️ SSI

Server-Side Includes (SSI) is a technology web applications use to create dynamic content on HTML pages. SSI is supported by many popular web servers such as [Apache](https://httpd.apache.org/docs/current/howto/ssi.html) and [IIS](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/serversideinclude). The use of SSI can often be inferred from the file extension. Typical file extensions include `.shtml`, `.shtm`, and `.stm`.

SSI injection occurs when an attacker can inject SSI directives into a file that is subsequently served by the web server, resulting in the execution of the injected SSI directives.

SSI utilizes `directives` to add dynamically generated content to a static HTML page.

* `name`: the directive's name
* `parameter name`: one or more parameters
* `value`: one or more parameter values

This is the syntax:

```ssi
<!--#name param1="value1" param2="value" -->
```

#### <mark style="color:red;">Common SSI directives.</mark>

**printenv**

This directive prints environment variables. It does not take any variables.

```ssi
<!--#printenv -->
```

**config**

can be used to change the configuration by specifying corresponding parameters (e.x errmsg) -> this changes error message

```ssi
<!--#config errmsg="Error!" -->
```

#### echo

prints the value of any variable given in the `var` parameter, here are some:

* `DOCUMENT_NAME`: the current file's name
* `DOCUMENT_URI`: the current file's URI
* `LAST_MODIFIED`: timestamp of the last modification of the current file
* `DATE_LOCAL`: local server time

```ssi
<!--#echo var="DOCUMENT_NAME" var="DATE_LOCAL" -->
```

**exec**

```ssi
<!--#exec cmd="whoami" -->
```

**include**

It only allows for the inclusion of files in the web root directory

```ssi
<!--#include virtual="index.html" -->
```
