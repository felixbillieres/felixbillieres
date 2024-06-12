# 😙 API Testing

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We need to start by identifying API endpoints.

```
GET /api/books HTTP/1.1
Host: example.com
```

The API endpoint for this request is `/api/books`

### Lab: Exploiting an API endpoint using documentation

We start by connecting with the credentials then modify the email address and capture the request:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can't access carlos by abusing the path:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we can access a different endpoint by modifying the request:

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/pic.png" alt=""><figcaption></figcaption></figure>

And we are able to access what seems to be possible requests to the API

if we click on delete, there is a popup that queries for us:

<figure><img src="../../.gitbook/assets/deletecarlos.png" alt=""><figcaption></figcaption></figure>

Or we could've done it through the Burp interface with a DELETE /api/user/carlos req

#### Identifying API endpoints

The HTTP method specifies the action to be performed on a resource. For example:

```
GET - Retrieves data from a resource.
PATCH - Applies partial changes to a resource.
OPTIONS - Retrieves information on the types of request methods that can be used on a resource.
```

### Lab: Finding and exploiting an unused API endpoint

First we find the API endpoint:

<figure><img src="../../.gitbook/assets/image (16) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I then throw in a little sniper attack with several HHTP verbs:

{% embed url="https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods" %}

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can see the PATCH method has a different length

We send it to repeater and trigger an error:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we add a Content-Type:

```
Content-Type: application/json

{
	"":""
}
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we add the price parameter in our request:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We forward the req and refresh the page:

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And we can order our jacket:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Fuzzing to find hidden endpoints

Let's say we have the following endpoint:

`PUT /api/user/update`

we could use Burp Intruder to fuzz for other resources with the same structure. For example, we could fuzz the `/update` position of the path with a list of other common functions, such as `delete` and `add`

#### Identifying hidden parameters

Let's say we have a `PATCH /api/users/` request, which enables users to update their username and email and the JSON looks like:

```
{
    "username": "wiener",
    "email": "wiener@example.com",
}
```

And the extended request would include a user ID: `GET /api/users/123` with the following JSON

```
{
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "isAdmin": "false"
}
```

"This may indicate that the hidden `id` and `isAdmin` parameters are bound to the internal user object, alongside the updated username and email parameters."

Interesting stuff about testing vulns: [https://portswigger.net/web-security/learning-paths/api-testing/api-testing-mass-assignment-vulnerabilities/api-testing/testing-mass-assignment-vulnerabilities](https://portswigger.net/web-security/learning-paths/api-testing/api-testing-mass-assignment-vulnerabilities/api-testing/testing-mass-assignment-vulnerabilities)

### Lab: Exploiting a mass assignment vulnerability

We first find the endpoint and the JSON related:

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We copy and paste all the JSON in the request area and change some stuff

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And then forward the req and it solves the lab

#### Testing for server-side parameter pollution in the query string

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To test for server-side parameter pollution in the query string, place query syntax characters like `#`, `&`, and `=` in your input and observe how the application responds.

Let's say we have this req:

```
GET /userSearch?name=peter&back=/home
```

And the server queries the internal API with the following request:

```
GET /users/search?name=peter&publicProfile=true
```

We can use a URL-encoded `#` character to attempt to truncate the server-side request.

For example:

```
GET /userSearch?name=peter%23foo&back=/home
```

{% hint style="info" %}
It's essential that we URL-encode the `#` character. Otherwise the front-end application will interpret it as a fragment identifier and it won't be passed to the internal API.
{% endhint %}

It could lead to truncating the `publicProfile=true` part

#### Injecting invalid parameters

Using the same method, we could try to inject other parameters using the & character:

```
GET /userSearch?name=peter%26foo=xyz&back=/home
```

It would lead to the following req to the API:

```
GET /users/search?name=peter&foo=xyz&publicProfile=true
```

if the response is unchanged this may indicate that the parameter was successfully injected but ignored by the application.

#### Injecting valid parameters

Following this logic we could try to inject real parameters like an email parameter:

```
GET /userSearch?name=peter%26email=foo&back=/home    
```

which would lead to the following:

```
GET /users/search?name=peter&email=foo&publicProfile=true
```

#### Overriding existing parameters

Still following the rabbit hole, the parameters we have in our query can be overwritten to our advantage:

```
GET /userSearch?name=peter%26name=carlos&back=/home
```

req to the API:

```
GET /users/search?name=peter&name=carlos&publicProfile=true
```

It will depend on how the application processes the second parameter.

If you're able to override the original parameter, you may be able to conduct an exploit. For example, you could add `name=administrator` to the request.

### Lab: Exploiting server-side parameter pollution in a query string

the /forgot-password seemed interesting:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

i try to add the # URL encoded to test out the query, we see the "field not specified" so let's try to input with a "field" variable:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The query seems to work but the fields are not hitting any real values

Unfortunately we need to send to intruder but I do not have Pro version

So this is the remaining steps:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ok so this seems to work

We also found the following field in the /static/js/forgotPassword.js req

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we can try this req:

```
username=administrator%26field=reset_token%23
```

Will continue later

***

{% embed url="https://portswigger.net/web-security/learning-paths/api-testing/api-testing-testing-for-server-side-parameter-pollution-in-rest-paths/api-testing/server-side-parameter-pollution/testing-for-server-side-parameter-pollution-in-rest-paths" %}

{% embed url="https://portswigger.net/web-security/learning-paths/api-testing/api-testing-testing-for-server-side-parameter-pollution-in-structured-data-formats/api-testing/server-side-parameter-pollution/testing-for-server-side-parameter-pollution-in-structured-data-formats" %}
