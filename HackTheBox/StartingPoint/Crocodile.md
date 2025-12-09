# Crocodile - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 21,80 -sC -sV -vv -oA croco 10.129.1.15
```
<img width="877" height="496" alt="image" src="https://github.com/user-attachments/assets/be54c52c-98ea-4c1b-a4b5-ac889bc7bb11" />

- FTP(21)
- HTTP(80)

---
## FTP
- `anonymous` 로그인이 가능.
- 해당 폴더에 존재하는 파일들을 모두 다운로드 받아줌.
```bash
ftp 10.129.1.15
```
<img width="1096" height="560" alt="image" src="https://github.com/user-attachments/assets/724ed0af-0f6a-4d99-b868-7aec8439e4ad" />

---
## HTTP
###  banner grabbing
```bash
whatweb http://10.129.1.15/

curl -I -L http://10.129.1.15
```
<img width="1105" height="356" alt="image" src="https://github.com/user-attachments/assets/9fe56f8a-7575-4953-8ecf-92f900e9caf4" />

- `whatweb`에서 확인된 `hello@ayroui.com`의 도메인을 `/etc/hosts`에 등록해줌.
<img width="1105" height="155" alt="image" src="https://github.com/user-attachments/assets/29fa70b2-57b5-4b5d-ac72-ec9225b0b653" />

### gobuster
- `gobuster`로 `login.php`를 찾아냄.
```bash
gobuster dir -u http://ayroui.com -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,md
```
<img width="1105" height="418" alt="image" src="https://github.com/user-attachments/assets/3a9073c8-efd0-4f2a-91a8-0bc6ab39c4dc" />

- http://ayroui.com/login.php
<img width="715" height="418" alt="image" src="https://github.com/user-attachments/assets/0e8ec849-038e-4a47-9e4f-61ead7ea842a" />

### burpsuite
- 로그인시 어떻게 전달이 되는지 확인하기 위해서 `burpsuite`를 사용해서 인터셉트 해서 확인.
<img width="774" height="448" alt="image" src="https://github.com/user-attachments/assets/7100e0c5-7634-40a1-8ddd-1a33e9a7b01e" />

- FTP에서 다운로드 받은 `userlist`와 `passwd` 파일로 `password spraying`을 시도해봄.
<img width="1500" height="387" alt="image" src="https://github.com/user-attachments/assets/613f279d-f250-4a39-a9db-8f56c182b3e6" />

- `admin:rKXM59ESxesUFHAd`가 다른 시도와 달리 Content-Length가 적었음.
<img width="1496" height="185" alt="image" src="https://github.com/user-attachments/assets/ce8580a8-d5bf-40ce-a9b3-d5bdea5e8bd7" />

- `admin`계정으로 로그인이 성공했고, flag를 얻을 수 있음.
<img width="504" height="361" alt="image" src="https://github.com/user-attachments/assets/16aa9e9f-6ed5-44ad-90ea-9e4116adcef5" />




