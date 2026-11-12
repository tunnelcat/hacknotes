## Excessive trust in client-side controls
- Users don't always use web browsers for client-side actions. Proxies can bypass the client-side controls pretty easily.

Example: Editing the price directly with a HTTP request to put a free item in your cart

```txt
productId=1&redir=PRODUCT&quantity=1&price=0
```

## Failure to handle unconventional input
- Inadequate validation of user input

Example: Can buy negative quantities of a product

- Exploiting the back-end functionality

Example: Can buy large amounts of product due to integer overflow

![5e95dded16638658e77556832c1a9a20.png](../../_resources/5e95dded16638658e77556832c1a9a20-1.png)

Example: After registration, the email is truncated. This may be used to abuse email subdomains. (If @dontwannacry.com users are admin, then AAAAAAAAAAAAAAAA@dontwannacry.com.attacker-controlled.com will be an admin account because it will be truncated up to dontwannacry.com, but the registration email will be sent to attacker-controlled.com

![7b1c4b1c2ab1c3538aa6dbc6da65fc6e.png](../../_resources/7b1c4b1c2ab1c3538aa6dbc6da65fc6e-1.png)

![be5c1df418cbdf91b6df3607442c1e79.png](../../_resources/be5c1df418cbdf91b6df3607442c1e79-1.png)

## Flawed assumptions about user behavior
- A user might initially be trusted, but if the trust is indefinite, then trust manipulation can occur

Example: Changing the account email to a trusted one AFTER registering with a different email

- Users don't always supply mandatory input
	- Remove one parameter at a time to see what happens to logic
	- Try deleting name and value or just the value of parameters
	- Applies to both URL and POST parameters
	- Don't forget to check cookies!

Example: Change password form, but remove the current-password parameter and it changes the password anyways... (arbitrary user password change)

- Users don't always follow intended sequences / workflows

Example: User can add product to cart, then go straight to order confirmation to buy whatever they want

Example: User can forward the POST login request, drop other packets in the workflow, and navigate manually to some wanted URL to find that they have "bypassed" the workflow

## Business domain-specific flaws
- Domain meaning: online shopping, social media, etc.
- There are a number of logical flaws that may occur, but many are domain-specific and hard to spot...

Example: Interleaving coupons to get huge reductions (app only looks at the most recent coupon to check if it is used...)

![90d1b5423b924dddb0459f1a5d37882d.png](../../_resources/90d1b5423b924dddb0459f1a5d37882d-1.png)

## Authentication bypass via encryption oracle
- Encryption oracle: a place where user can submit input and get a ciphertext
- How is this useful? If authentication uses the same encryption algo, the user can use the oracle to generate arbitrary valid ciphertext to bypass controls

Example: https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-authentication-bypass-via-encryption-oracle

- Note: this is pretty involved. You need to know about padding and deleting bytes in the ciphertext and use Burp Decoder. You also need to figure out what requests to keep in Repeater for the encryption and decryption, so be organized. 

