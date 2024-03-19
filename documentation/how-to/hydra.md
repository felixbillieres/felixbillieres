# 🐲 Hydra

## **Hydra - Brute Force Mastery**

Hydra is a powerful and versatile tool for carrying out brute force attacks on various services. From simple login forms to complex protocols, Hydra can be your go-to choice. Let's explore its usage from basic to advanced, including HTTP form brute force.

<figure><img src="https://www.researchgate.net/publication/325961962/figure/fig5/AS:641151904260098@1529873934258/Cracking-Guessing-Weak-Password-exploitation-using-multiprotocol-brute-force-tool-Hydra.png" alt=""><figcaption></figcaption></figure>

### **1. Basic Hydra Commands**

#### **SSH Brute Force Example:**

```bash
hydra -l <username> -P <password-list> ssh://<target-ip>
```

* **`-l <username>`:** Specifies the target username.
* **`-P <password-list>`:** Specifies the path to the password list.
* **`ssh://<target-ip>`:** Specifies the target SSH server.

<figure><img src="https://www.researchgate.net/publication/325075314/figure/fig4/AS:672678016987137@1537390345318/SSH-dictionary-bruteforce-attack-with-Hydra-Comparing-start-and-end-time-Hydra-was-able.ppm" alt=""><figcaption><p>Successful bruteforce </p></figcaption></figure>

#### **HTTP Basic Auth Example:**

```bash
hydra -l <username> -P <password-list> <target-url>
```

* **`-l <username>`:** Specifies the target username.
* **`-P <password-list>`:** Specifies the path to the password list.
* **`<target-url>`:** Specifies the target URL for HTTP basic authentication.

### **2. Advanced Hydra Commands**

#### **HTTP Form Brute Force Example:**

```bash
hydra -l <username> -P <password-list> <target-url> http-post-form "<login-form>:user=^USER^&pass=^PASS^:Login failed"
```

* **`-l <username>`:** Specifies the target username.
* **`-P <password-list>`:** Specifies the path to the password list.
* **`<target-url>`:** Specifies the target URL for the login form.
* **`http-post-form`:** Specifies the HTTP POST form method.
* **`"<login-form>:user=^USER^&pass=^PASS^:Login failed"`:** Describes the login form parameters and failure message.

### **3. HTTP Form Brute Force Explained**

#### **Understanding the Command:**

* `http-post-form`: Specifies the HTTP POST method for form submission.
* `"<login-form>:user=^USER^&pass=^PASS^:Login failed"`: Describes the login form structure.
  * `<login-form>`: Replace this with the actual login form path.
  * `user=^USER^&pass=^PASS^`: Specifies the parameters for the username and password.
  * `Login failed`: Indicates the login failure message.

#### **Usage Tips:**

* Identify the login form structure by inspecting the HTML source code.
* Replace `<login-form>` with the actual form path.
* Customize the parameters based on the form fields.
* Note the failure message to recognize unsuccessful login attempts.
