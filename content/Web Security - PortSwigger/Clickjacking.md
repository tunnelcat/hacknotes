# What is clickjacking? 
A user can be tricked to click a button that is seemingly harmless. However, overlapping the decoy button is a hidden button from some external website (incorporated as a transparent iframe, for example). This differs from CSRF in that a user must perform an action (clicking) for exploitation. 

CSRF tokens do not protect from this attack as all requests are happening on an authentic on-domain basis. 

## Basic clickjacking attack 
Clickjacking attacks usually use CSS to manipulate layers. The target website is incorporated as a hidden (transparent) layer above the decoy website, and the key target elements of the target website (like buttons) are copied artificially on the decoy website. Absolute and relative position values are used to ensure that the target website accurately overlaps the decoy regardless of screen size, browser type and platform.

```html
<head>
  <style>
    #target_website {
      position:relative;
      width:128px;
      height:128px;
      opacity:0.00001;
      z-index:2;
      }
    #decoy_website {
      position:absolute;
      width:300px;
      height:400px;
      z-index:1;
      }
  </style>
</head>
...
<body>
  <div id="decoy_website">
  ...decoy web content here...
  </div>
  <iframe id="target_website" src="https://vulnerable-website.com">
  </iframe>
</body>
```

![50eac608435151d25a98e3a7160d71ee.png](../../_resources/50eac608435151d25a98e3a7160d71ee-1.png)

## Clickjacking with prefilled form input
Some websites that require form completion permit prefilled form inputs using GET parameters before submission. The target URL parameters can be modified to fill in the GET values and then the submit button can be clickjacked as usual. 

```
<iframe id="target_website" src="https://victim-website.com/my-account?email=hackerman@hacker.net">
```

## Frame busting scripts
Clickjacking is possible when websites can be framed. So framing capability for websites may be restricted via frame busting scripts. These can: 
- check and enforce that the current application window is the main/top window
- make all frames visible
- prevent clicking on invisible frames
- intercept and flag potential clickjacking attacks

To work around this, you can use the `sandbox` attribute of HTML5 iframes. When the `allow-top-navigation` value is omitted, the iframe cannot check whether it is the top window. The `allow-forms` and `allow-scripts` permit actions within the iframe as normal: 

```
<iframe id="victim_website" src="https://victim-website.com" sandbox="allow-forms"></iframe>
```

## Combining clickjacking with a DOM XSS attack
Clickjacking can take advantage of an existing DOM XSS attack, increasing the power and exploitation abilities.

## Multistep clickjacking
Some actions take multiple steps (like buying an item), so attackers will need to use multiple divisions/elements or iframes and precise positioning for successful exploitation.

# How to prevent clickjacking attacks
## X-Frame-Options
- header that controls the inclusion of a webpage within a frame

```
X-Frame-Options: deny
X-Frame-Options: sameorigin
X-Frame-Options: allow-from https://normal-website.com
```

## Content Security Policy (CSP)
- detects and prevents against XSS/clickjacking
- the `frame-ancestors` directive can be similar in behavior to `X-Frame-Options`

```
Content-Security-Policy: frame-ancestors 'none'; 
Content-Security-Policy: frame-ancestors 'self'; 
Content-Security-Policy: frame-ancestors normal-website.com; 
```
