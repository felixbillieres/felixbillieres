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

Right-click and copy the URL in the "show response in browser"

(problems with CA cert, will come back later)
