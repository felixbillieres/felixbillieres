---
description: https://portswigger.net/web-security/learning-paths/prototype-pollution
---

# 🐊 Prototype pollution

Prototype pollution is a JavaScript vulnerability that enables an attacker to add arbitrary properties to global object prototypes, which may then be inherited by user-defined objects.

<figure><img src="../../.gitbook/assets/image (19) (1).png" alt=""><figcaption></figcaption></figure>

**Reminder on JavaScript Objects:**

A JavaScript object is essentially just a collection of `key:value` pairs known as "properties".

```
const user =  {
    username: "wiener",
    userId: 01234,
    isAdmin: false
}
//that can be accessed using the following syntax:
user.username     // "wiener"
user['userId']    // 01234
```

Some JavaScript properties can execute functions:

```
const user =  {
    username: "wiener",
    userId: 01234,
    exampleMethod: function(){
        // do something
    }
}
```
