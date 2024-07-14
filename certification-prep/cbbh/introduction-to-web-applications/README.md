# 👶 Introduction to Web Applications

### Web Applications vs. Websites

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here are the different layouts of a web app:

| **Category**                     | **Description**                                                                                                                                                                                                                                                      |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Web Application Infrastructure` | Describes the structure of required components, such as the database, needed for the web application to function as intended. Since the web application can be set up to run on a separate server, it is essential to know which database server it needs to access. |
| `Web Application Components`     | The components that make up a web application represent all the components that the web application interacts with. These are divided into the following three areas: `UI/UX`, `Client`, and `Server` components.                                                    |
| `Web Application Architecture`   | Architecture comprises all the relationships between the various web application components.                                                                                                                                                                         |

For the infrastructure part, web apps can choose different models such as:

* `Client-Server`

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* `One Server`

<figure><img src="../../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

the riskiest design since `all eggs are in one basket`

* `Many Servers - One Database`

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

It allows different web apps access to the same data without syncing the data between them.

* `Many Servers - Many Databases`

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

if any web server or database goes offline, a backup will run in its place to reduce downtime as much as possible

## Front End vs. Back End

The front end includes everything that the user sees and interacts with, like the page's main elements such as the title and text [HTML](https://www.w3schools.com/html/html\_intro.asp), the design and animation of all elements [CSS](https://www.w3schools.com/css/css\_intro.asp), and what function each part of a page performs [JavaScript](https://www.w3schools.com/js/js\_intro.asp).

Whereas the back end drives all of the core web application functionalities, all of which is executed at the back end server, which processes everything required for the web application to run correctly.

Here are the four main back end components for web apps:

| **Component**            | **Description**                                                                                                                                                                                                              |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Back end Servers`       | The hardware and operating system that hosts all other components and are usually run on operating systems like `Linux`, `Windows`, or using `Containers`.                                                                   |
| `Web Servers`            | Web servers handle HTTP requests and connections. Some examples are `Apache`, `NGINX`, and `IIS`.                                                                                                                            |
| `Databases`              | Databases (`DBs`) store and retrieve the web application data. Some examples of relational databases are `MySQL`, `MSSQL`, `Oracle`, `PostgreSQL`, while examples of non-relational databases include `NoSQL` and `MongoDB`. |
| `Development Frameworks` | Development Frameworks are used to develop the core Web Application. Some well-known frameworks include `Laravel` (`PHP`), `ASP.NET` (`C#`), `Spring` (`Java`), `Django` (`Python`), and `Express` (`NodeJS JavaScript`).    |

Here are the 20 most comment in web apps:

<table data-header-hidden><thead><tr><th width="107"></th><th></th></tr></thead><tbody><tr><td><strong>No.</strong></td><td><strong>Mistake</strong></td></tr><tr><td><code>1.</code></td><td>Permitting Invalid Data to Enter the Database</td></tr><tr><td><code>2.</code></td><td>Focusing on the System as a Whole</td></tr><tr><td><code>3.</code></td><td>Establishing Personally Developed Security Methods</td></tr><tr><td><code>4.</code></td><td>Treating Security to be Your Last Step</td></tr><tr><td><code>5.</code></td><td>Developing Plain Text Password Storage</td></tr><tr><td><code>6.</code></td><td>Creating Weak Passwords</td></tr><tr><td><code>7.</code></td><td>Storing Unencrypted Data in the Database</td></tr><tr><td><code>8.</code></td><td>Depending Excessively on the Client Side</td></tr><tr><td><code>9.</code></td><td>Being Too Optimistic</td></tr><tr><td><code>10.</code></td><td>Permitting Variables via the URL Path Name</td></tr><tr><td><code>11.</code></td><td>Trusting third-party code</td></tr><tr><td><code>12.</code></td><td>Hard-coding backdoor accounts</td></tr><tr><td><code>13.</code></td><td>Unverified SQL injections</td></tr><tr><td><code>14.</code></td><td>Remote file inclusions</td></tr><tr><td><code>15.</code></td><td>Insecure data handling</td></tr><tr><td><code>16.</code></td><td>Failing to encrypt data properly</td></tr><tr><td><code>17.</code></td><td>Not using a secure cryptographic system</td></tr><tr><td><code>18.</code></td><td>Ignoring layer 8</td></tr><tr><td><code>19.</code></td><td>Review user actions</td></tr></tbody></table>
