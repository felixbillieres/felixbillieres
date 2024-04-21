# 😙 API Testing

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

We need to start by identifying API endpoints.

```
GET /api/books HTTP/1.1
Host: example.com
```

The API endpoint for this request is `/api/books`

### Lab: Exploiting an API endpoint using documentation

We start by connecting with the credentials then modify the email address and capture the request:

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

We can't access carlos by abusing the path:

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

So we can access a different endpoint by modifying the request:

<figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

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
