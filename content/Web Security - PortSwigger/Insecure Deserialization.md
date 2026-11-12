# What is serialization?
Serialization is the process of converting complex data structures, such as objects and their fields, into a "flatter" format that can be sent and received as a sequential stream of bytes. Deserialization is the process of restoring this byte stream to a fully functional replica of the original object, in the exact state as when it was serialized.

> Note: serialization may be referred to as marshalling (Ruby) or pickling (Python).

# What is insecure deserialization?
Insecure deserialization is when user-controllable data is deserialized by a website. This potentially enables an attacker to manipulate serialized objects in order to pass harmful data into the application code. 

It is even possible to replace a serialized object with an object of an entirely different class. Fittingly, insecure deserialization is sometimes known as an "object injection" vulnerability. An object of an unexpected class might cause an exception. However, the damage may already be done. Many deserialization-based attacks are completed **before** deserialization is finished. So, even websites using strongly typed languages can be vulnerable.

# How do insecure deserialization vulnerabilities arise?
People don't understand how dangerous deserializing user-controllable data can be. Ideally, user input should never be deserialized. Checks may not consider edge cases, and checking the data after deserialization may be too late to prevent exploitation. In short, deserializing untrusted input is almost always insecure.

# What is the impact of insecure deserialization? 
The impact of insecure deserialization can be very severe because it provides an entry point to a massively increased attack surface. It allows an attacker to reuse existing application code in harmful ways, resulting in numerous other vulnerabilities, often remote code execution. 

# Exploiting insecure deserialization vulnerabilities
## How to identify insecure deserialization
During auditing, look at data being passed into the website and try to find serialized data. Different languages use different formats, and they will be familiar with practice. Finally, test whether you can control the data. 

### PHP serialization
Look for `unserialize()` in the source code.

PHP object example: 
```php
$user->name = "carlos";
$user->isLoggedIn = true;
```

Serialized: 
```txt
O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}
```

Explanation: 
```txt
O:4:"User" - An object with the 4-character class name "User"
2 - the object has 2 attributes
s:4:"name" - The key of the first attribute is the 4-character string "name"
s:6:"carlos" - The value of the first attribute is the 6-character string "carlos"
s:10:"isLoggedIn" - The key of the second attribute is the 10-character string "isLoggedIn"
b:1 - The value of the second attribute is the boolean value true
```

### Java serialization
Any class that implements the interface `java.io.Serializable` can be serialized and deserialized. Look for `readObject()` in the code (used to deserialize from an `InputStream`). 

Serialized Java objects always begin with the same bytes. 

```ac ed``` in hexadecimal
```rO0``` in Base64

## Modifying object attributes
Tampering with the data to change attributes of a valid object can lead to various vulnerabilities. For example, if a User object has an admin boolean, try setting it to True and see if the app responds differently. 

## Modifying data types
PHP-based loose comparison (using the == operator) can compare different data types that might evaluate in ways that are unexpected. 

```
5 == "5" // true, str converted to int
5 == "5 of something" // true, str converted to beginning number
0 == "Example string" // true, str converted to 0, no number
```

Vulnerable code example: 
```
$login = unserialize($_COOKIE)
if ($login['password'] == $password) {
// log in successfully
}
```

If the attacker sets `$password` as 0, as long as the stored password does not start with a number, the condition returns `true`. This is because deserialization PRESERVES the int data type specified by the attacker in the serialized data (would not work if the 0 was converted to a str).

> Note: Don't forget to update any type labels and length indicators in the serialized data.

> Pro tip: Use the Hackvertor extension. It will make your life easier.

## Using app functionality
Certain functionality may perform dangerous operations on data from a deserialized object. You can exploit this by using deserialization to pass unexpected data. 

For example, if a website has a "Delete user" feature, and the user's profile picture is deleted by accessing the path `$user->image_location` specified in a serialized object, then `image_location` can be set to an arbitrary file path. This allows the user to delete an arbitrary file by deleting their own account.

![f931d14ca305a6d68e0fed7002134fa8.png](../../_resources/f931d14ca305a6d68e0fed7002134fa8-1.png)

## Magic methods
Magic methods are a special subset of methods that you do not have to explicitly invoke. Instead, they are invoked automatically whenever a particular event or scenario occurs. Magic methods are a common feature of object-oriented programming in various languages. They are sometimes indicated by prefixing or surrounding the method name with double-underscores. 

Example: PHP's `__construct()` (similar to Python's `__init__`) runs when an object of the class is instantiated. 

Magic methods can be dangerous when the code that they execute handles attacker-controlled data. Some languages even have magic methods that are invoked automatically during the deserialization process. 

Example: PHP's `unserialize()` looks for and invokes an object's `__wakeup()` magic method.

Example: serializable classes in Java can declare their own `readObject()` (used to read serialized data) method... This declaration acts as a magic method invoked during deserialization.

```java
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {...};
```

## Injecting arbitrary objects
If an attacker has access to the source code, they can study all of the available classes in detail. To construct a simple exploit, they would look for classes containing deserialization magic methods, then check whether any of them perform dangerous operations on controllable data. The attacker can then pass in a serialized object of this class to use its magic method for an exploit. 

Example: 
```php
function __destruct() {
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
```

If `lock_file_path` (instance variable) is a user-controlled value in the serialized data, an attacker can inject an object of the same class with a crafted instance variable to arbitrarily delete any file (using the `__destruct()` magic method in this case).

## Gadget chains
A gadget is a snippet of code that exists in the app that can help an attacker to achieve a particular goal. These gadgets can be chained, eventually resulting in input reaching a "sink gadget", where it can cause substantial damage.

> Note: As an attacker, you don't write the gadgets. The gadgets have to already exist in the code of the website.

Many insecure deserialization vulnerabilities are only exploitable through gadget chaining, and this is a key aspect of insecure deserialization exploitation. Source code access is often required.

### Working with pre-built gadget chains
Some gadget chains are known to be exploitable and tools exist to exploit them. For example, if a gadget chain exists in Java's Apache Commons Collections library, then any website implementing it may be vulnerable.

#### ysoserial
"ysoserial" is a tool for Java deserialization. It lets you choose one of the provided gadget chains for a library that you think the target application is using, then pass in a command that you want to execute. It then creates an appropriate serialized object based on the selected chain. This will take some trial and error, but nothing too substantial.

> Pro tip: Use the Java Deserialization Scanner plugin for Burp! (maybe...)

A typical usage of ysoserial looks like: 
```bash
java -jar ysoserial-master-8eb5cbfbf6-1.jar CommonsCollections4 "rm /home/carlos/morale.txt" | base64 -w 0 > payload   
```

> Note: you have to figure out what encodings the serialized data is in. It may be base64 then URL encoded, for example. 

#### PHP Generic Gadget Chains
Most languages that suffer from insecure deserialization often have tools for pre-built gadget chain attacks. PHP has PHPGGC, for example. 

A typical usage of PHPGGC looks like: 
```bash
./phpggc -l                # list gadget chains
./phpggc -l laravel        # filter gadget chains
./phpggc -i symfony/rce1   # get detailed info about a chain
./phpggc <gadget-chain> [commands]  # general usage
```

```bash
./phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64 -w 0
```

### Working with documented gadget chains
It's always worth taking a look online to see if there are any documented exploits that you can adapt to attack your target website. Even if there is no dedicated tool for automatically generating the serialized object, you can still find documented gadget chains for popular frameworks and adapt them manually. 

## Creating your own exploit
Skipped

## PHAR deserialization
Skipped

# How to prevent insecure deserialization vulnerabilities
