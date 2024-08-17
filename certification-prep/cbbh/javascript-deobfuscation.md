# ☠️ JavaScript Deobfuscation

Javascript can be internally written between `<script>` elements or written into a separate `.js` file and referenced within the `HTML` code.

```html
<script src="secret.js"></script>
```

We can check out the script by clicking on `secret.js`, which should take us directly into the script.

### What is obfuscation

Obfuscation is a technique used to make a script more difficult to read by humans but allows it to function the same from a technical point of view, though performance may be slower.

code obfuscators often turn the code into a dictionary of all of the words and symbols used within the code and then attempt to rebuild the original code during execution by referring to each word and symbol from the dictionary.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Let's imagine a simple javascript code:

```javascript
console.log('JavaScript Deobfuscation Module');
```

A simple way of making our code harder to read is to do code minification (puting all the code on a single line) for this we could use [javascript-minifier](https://javascript-minifier.com/)

To go a bit deeper we could use [BeautifyTools](http://beautifytools.com/javascript-obfuscator.php) that would return our simple js code to a hard to read content that would do exactly what the simple one does:

```javascript
eval(function(p,a,c,k,e,d){e=function(c){return c};if(!''.replace(/^/,String)){while(c--){d[c]=k[c]||c}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('5.4(\'3 2 1 0\');',6,6,'Module|Deobfuscation|JavaScript|HTB|log|console'.split('|'),0,{}))
```

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

and to even add more layers of complexity we could go on [https://obfuscator.io](https://obfuscator.io)

according to what parameters we eneter we could end up with this code that is our simply echo command:

```javascript
var _0x1ec6=['Bg9N','sfrciePHDMfty3jPChqGrgvVyMz1C2nHDgLVBIbnB2r1Bgu='];(function(_0x13249d,_0x1ec6e5){var _0x14f83b=function(_0x3f720f){while(--_0x3f720f){_0x13249d['push'](_0x13249d['shift']());}};_0x14f83b(++_0x1ec6e5);}(_0x1ec6,0xb4));var _0x14f8=function(_0x13249d,_0x1ec6e5){_0x13249d=_0x13249d-0x0;var _0x14f83b=_0x1ec6[_0x13249d];if(_0x14f8['eOTqeL']===undefined){var _0x3f720f=function(_0x32fbfd){var _0x523045='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/=',_0x4f8a49=String(_0x32fbfd)['replace'](/=+$/,'');var _0x1171d4='';for(var _0x44920a=0x0,_0x2a30c5,_0x443b2f,_0xcdf142=0x0;_0x443b2f=_0x4f8a49['charAt'](_0xcdf142++);~_0x443b2f&&(_0x2a30c5=_0x44920a%0x4?_0x2a30c5*0x40+_0x443b2f:_0x443b2f,_0x44920a++%0x4)?_0x1171d4+=String['fromCharCode'](0xff&_0x2a30c5>>(-0x2*_0x44920a&0x6)):0x0){_0x443b2f=_0x523045['indexOf'](_0x443b2f);}return _0x1171d4;};_0x14f8['oZlYBE']=function(_0x8f2071){var _0x49af5e=_0x3f720f(_0x8f2071);var _0x52e65f=[];for(var _0x1ed1cf=0x0,_0x79942e=_0x49af5e['length'];_0x1ed1cf<_0x79942e;_0x1ed1cf++){_0x52e65f+='%'+('00'+_0x49af5e['charCodeAt'](_0x1ed1cf)['toString'](0x10))['slice'](-0x2);}return decodeURIComponent(_0x52e65f);},_0x14f8['qHtbNC']={},_0x14f8['eOTqeL']=!![];}var _0x20247c=_0x14f8['qHtbNC'][_0x13249d];return _0x20247c===undefined?(_0x14f83b=_0x14f8['oZlYBE'](_0x14f83b),_0x14f8['qHtbNC'][_0x13249d]=_0x14f83b):_0x14f83b=_0x20247c,_0x14f83b;};console[_0x14f8('0x0')](_0x14f8('0x1'));
```

there is some crazy stuff such as obfuscators that are not even readable:

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

## Deobfuscation

A term to now is to `Beautify` our code. if we were using Firefox, we can open the browser debugger with \[ `CTRL+SHIFT+Z` ], and then click on our script `secret.js`. This will show the script in its original formatting, but we can click on the '`{ }`' button at the bottom, which will `Pretty Print` the script into its proper JavaScript formatting:

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

and with obfuscated code:

<figure><img src="../../.gitbook/assets/image (1282).png" alt=""><figcaption></figcaption></figure>

A good tool to try and understand this code could be [UnPacker](https://matthewfl.com/unPacker.html)

later in the career i'll need to learn advanced JavaScript Deobfuscation and Reverse Engineering, i'll have to  check out the [Secure Coding 101](https://academy.hackthebox.com/module/details/38) module, which should thoroughly cover this topic.

a good thing to know how to do is decoding the basic stuff in CLI ->

base64

```
echo encodedData | base64 -d
```

hex

```
#to encode
echo hello world | xxd -p

# to decode
echo 68747470733a2f2f7777772e6861636b746865626f782e65752f0a | xxd -p -r
```
