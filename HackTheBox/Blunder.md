# Blunder - HackTheBox
## Recon
```bash
sudo nmap -p 80 -sC -sV -vv -oA Blunder 10.10.10.191
```
<img width="1104" height="174" alt="image" src="https://github.com/user-attachments/assets/cf07259a-58c3-4462-ba7a-bae49a638d56" />

- HTTP(80)

---
## HTTP
### banner grabbing
<img width="1104" height="294" alt="image" src="https://github.com/user-attachments/assets/2dfa3c07-a09e-4348-abaa-969e7d0e8e68" />

### gobuster
- `todo.txt`와 `README.md`를 발견.
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.191 -x php,md,txt -t 50
```
<img width="1104" height="580" alt="image" src="https://github.com/user-attachments/assets/10f61c75-1d59-4f4b-a81d-c5034b52252c" />

<br>
<br>

### CVE-2019-17240
- `README.md`에서  `bludit CMS` 발견.
<img width="1670" height="435" alt="image" src="https://github.com/user-attachments/assets/d654ba5c-165a-4852-a824-604660cbab8c" />

<br>
<br>

- `todo.txt`에서 `fergus`라는 이름을 발견.
<img width="448" height="168" alt="image" src="https://github.com/user-attachments/assets/be92e132-1613-460c-86b6-1ed07f3e3619" />

<br>
<br>

- `/admin`경로에서 `fergus`유로 기본 패스워드로 로그인 시도해보았으나 실패.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/ab524356-5710-4c82-bd0c-f7c85e39b8d5" />

<br>
<br>

- 메인페이지의 `html`소스코드에서 css 버전이 3.9.2로 나와있는데, 정확한 버전이라고 확정짓는 것은 불가.
<img width="707" height="60" alt="image" src="https://github.com/user-attachments/assets/32e5908a-41d6-4224-b4ad-64400f35b650" />

<br>
<br>

- https://github.com/bludit/bludit
- `bludit` 레포지토리를 확인해보니 `/bl-plugins/version/metadata.json`에서 버전이 확인 가능.
- `bludit 3.9.2`
<img width="696" height="222" alt="image" src="https://github.com/user-attachments/assets/87925a40-9e70-475d-bc07-4cc5c26bd6c9" />

<br>
<br>

- 브푸트포싱을 이용하여 비밀번호를 알아낼 수 있음.
<img width="1104" height="239" alt="image" src="https://github.com/user-attachments/assets/e2259ba8-c22f-464a-9f2d-43b083438cee" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2019-17240
<img width="1215" height="481" alt="image" src="https://github.com/user-attachments/assets/2b182b1f-46b2-4573-a488-2f2830fdb15b" />

<br>
<br>

- `X-Forwarded-For` 헤더를 변경해 가면서 로그인을 시도.
```bash
searchsploit -m 48942
```
<img width="1106" height="333" alt="image" src="https://github.com/user-attachments/assets/694445b8-f46c-47de-9cba-9715cc06a23e" />

<br>
<br>

- 다른 비밀번호 리스트를 시도해보았으나 실패하여 `cewl`를 사용하여 커스텀 리스트를 생성.
```bash
cewl -m 2 --with-numbers http://10.10.10.191 > passwords
```
<img width="1106" height="241" alt="image" src="https://github.com/user-attachments/assets/04549ff3-5949-44cb-8e84-2d5ed5e5ef9f" />

<br>
<br>

- 스크립트 실행하여 `fergus:RolandDeschain` 계정 발견.
```bash
python3 48942.py -l http://10.10.10.191/admin/login.php -u users -p passwords
```
<img width="1106" height="241" alt="image" src="https://github.com/user-attachments/assets/7e970fdb-536a-4478-b655-84239c38094f" />

### CVE-2019-16113
- https://nvd.nist.gov/vuln/detail/CVE-2019-16113
<img width="1216" height="398" alt="image" src="https://github.com/user-attachments/assets/dc9cc899-40b2-4cc6-8778-98dd68d58b43" />

<br>
<br>

- https://github.com/ynots0ups/CVE-2019-16113
- `proxies`를 추가해주고, 변수들을 수정한 뒤 `burpsuite`로 가로채서 확인.
<img width="1104" height="261" alt="image" src="https://github.com/user-attachments/assets/4ffde094-e367-4897-b729-9be9efb374fd" />

<br>
<br>

- 이미지 파일로 리버스 셸 코드를 저장.
<img width="1540" height="318" alt="image" src="https://github.com/user-attachments/assets/06594b46-f300-47a8-9c48-bea1a19b75e7" />

<br>
<br>

- `.htaccess`에 `png`파일을 `php`처럼 실행 가능하도록 수정.
<img width="1540" height="199" alt="image" src="https://github.com/user-attachments/assets/a95b4a6a-a809-49b6-b260-b390e76e679a" />

<br>
<br>

- 셸획득.
<img width="1106" height="251" alt="image" src="https://github.com/user-attachments/assets/4c288e11-ae95-4971-adf5-9cc66dd479ed" />

---
## Privesc
- `/var/www/bludit-3.10.0a/bl-content/databases/users.php`에서 `hugo`의 비밀번호 발견.
<img width="1106" height="422" alt="image" src="https://github.com/user-attachments/assets/2d48e779-adae-42c0-9092-543101a71c56" />

<br>
<br>

- https://crackstation.net/
- 크래킹하여 `Password120`획득.
<img width="1225" height="69" alt="image" src="https://github.com/user-attachments/assets/a44fa806-2c77-4527-a3c2-d5830f38d6b2" />

<br>
<br>

- `hugo:Password120` 로그인.
<img width="1105" height="82" alt="image" src="https://github.com/user-attachments/assets/88fa16ef-0b8c-4cb7-91f1-ce7c636cc3d7" />

<br>
<br>

- `root`유저를 제외한 나머지 유저로 `/bin/bash` 실행 가능.
<img width="1105" height="133" alt="image" src="https://github.com/user-attachments/assets/d183a3dc-d07b-4176-8b5f-3587bece2325" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/cve-2019-14287
<img width="1212" height="432" alt="image" src="https://github.com/user-attachments/assets/0da04def-015b-4cd9-9440-4cdad5a1a7a1" />

<br>
<br>

- https://github.com/M108Falcon/Sudo-CVE-2019-14287
- `-u#-1`옵션을 사용하여 `root`셸 획득.
```bash
sudo -u#-1 /bin/bash
```
<img width="1105" height="62" alt="image" src="https://github.com/user-attachments/assets/3892c606-9992-4694-80b0-820195a13b09" />

---
## FLAG
- `/home/hugo/user.txt`
<img width="1105" height="461" alt="image" src="https://github.com/user-attachments/assets/20645435-990a-46fa-8bb9-c2793add2d67" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="329" alt="image" src="https://github.com/user-attachments/assets/14b4e54d-aa7f-48dc-adb0-78dde4b4daef" />
