# 😙 API Testing

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We need to start by identifying API endpoints.

```
GET /api/books HTTP/1.1
Host: example.com
```

The API endpoint for this request is `/api/books`

### Lab: Exploiting an API endpoint using documentation

We start by connecting with the credentials then modify the email address and capture the request:

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can't access carlos by abusing the path:

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we can access a different endpoint by modifying the request:

<figure><img src="../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/pic.png" alt=""><figcaption></figcaption></figure>

And we are able to access what seems to be possible requests to the API

if we click on delete, there is a popup that queries for us:

<figure><img src="../.gitbook/assets/deletecarlos.png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

I then throw in a little sniper attack with several HHTP verbs:

{% embed url="https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods" %}

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

We can see the PATCH method has a different length

We send it to repeater and trigger an error:

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

So we add a Content-Type:

```
Content-Type: application/json

{
	"":""
}
```

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

So we add the price parameter in our request:

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

We forward the req and refresh the page:

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

And we can order our jacket:

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

We copy and paste all the JSON in the request area and change some stuff

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

And then forward the req and it solves the lab
