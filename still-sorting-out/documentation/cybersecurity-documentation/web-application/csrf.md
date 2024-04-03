# 💉 CSRF

## **Cross-Site Request Forgery (CSRF) - Beginner's Guide**

Cross-Site Request Forgery (CSRF) is an attack that tricks the victim into submitting a malicious request. It occurs when an attacker forces a user's browser to perform unwanted actions on a web application in which the user is authenticated. In this guide, we'll explore CSRF basics and analyze a sample payload.

<figure><img src="https://www.cobalt.io/hubfs/Imported_Blog_Media/cross-site_request_forgery_explainer_graphic-3.png" alt=""><figcaption></figcaption></figure>

### **1. Understanding CSRF**

#### **What is CSRF?**

CSRF is a type of attack where an attacker tricks a user's browser into performing actions on a website without the user's knowledge or consent. This can lead to unauthorized actions being taken on behalf of the user, such as changing account settings, making transactions, or performing other sensitive operations.

#### **How Does CSRF Work?**

1. The attacker embeds malicious code in a web page or email.
2. The victim, who is authenticated on a target website, loads the malicious page or clicks the malicious link.
3. The victim's browser unknowingly submits a request to the target website on which the user is authenticated.
4. The target website processes the request, thinking it came from the legitimate user.

### **2. Analyzing the Sample Payload**

```html
<form name="myForm" action="http://challenge01.root-me.org/web-client/ch22/index.php?action=profile" method="POST">
    <label for="username">Username:</label>
    <input type="text" id="username" name="username" value="test">
    <br>
    <label for="status">Status:</label>
    <input type="checkbox" id="status" name="status" checked>
    <br>
    <input type="submit" value="Submit">
</form>
<script>document.myForm.submit();</script>
```

#### **Payload Explanation:**

* **Form Details:**
  * The form targets `http://challenge01.root-me.org/web-client/ch22/index.php?action=profile`.
  * It contains a text input for the username and a checkbox for the status.
  * The form is set to automatically submit.
* **Script:**
  * The JavaScript code `document.myForm.submit();` triggers an automatic form submission.

#### **Analysis:**

* The form is designed to update the user's profile on `http://challenge01.root-me.org/web-client/ch22/index.php`.
* The attacker might trick a logged-in user into visiting a page containing this form, leading to an unintended profile update.

### **3. Mitigation Strategies**

#### **Protecting Against CSRF:**

1. **Anti-CSRF Tokens:**
   * Include unique tokens in forms to verify that the request comes from the legitimate user.
2. **SameSite Cookie Attribute:**
   * Set cookies to be sent only in first-party contexts to prevent cross-site requests.
3. **Referer Header Checking:**
   * Validate the Referer header to ensure requests originate from the same domain.
4. **Double-Submit Cookie Technique:**
   * Include a random value in both a cookie and a request parameter, validating that they match.
