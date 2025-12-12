# Igintion - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 80 -sC -sV -oA ignition -vv 10.129.1.27
```
<img width="1109" height="148" alt="image" src="https://github.com/user-attachments/assets/8a0440b7-c46f-4562-bb2d-24f034ef4699" />

- HTTP(80)

---
## HTTP(80)
- `/etc/hosts`에 `ignition.htb` 추가
<img width="1109" height="148" alt="image" src="https://github.com/user-attachments/assets/e70fc24f-cd4d-4762-9b59-7aa4b6d024fe" />

<br>
<br>

- banner grabbing
```
whatweb http://ignition.htb

curl -IL http://ignition.htb
```
<img width="1109" height="409" alt="image" src="https://github.com/user-attachments/assets/049882b3-b11b-45c7-bda1-79b54507b0e4" />

<br>
<br>

- `gobuster`로 `admin`페이지를 찾아냄.
```bash
gobuster dir -u http://ignition.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```
<img width="1109" height="488" alt="image" src="https://github.com/user-attachments/assets/7449841c-62f1-42d4-8eea-9a5bbbd728a4" />

<br>
<br>

- `admin`, `administrator`, `root`, `password`로 로그인을 시도해보았으나 실패해서 힌트를 확인하게 됨.
<img width="1476" height="67" alt="image" src="https://github.com/user-attachments/assets/3774fc0d-6453-47b1-8e82-4bf99fc577f8" />

<br>
<br>

<img width="213" height="380" alt="image" src="https://github.com/user-attachments/assets/6eb67d64-2de5-4d62-9988-067c9d67efe1" />

<br>
<br>

- 2023년에 가장 많이 사용된 비밀번호들 중에서 `qwerty123`을 사용하여 `admin`로그인
- flag 획득
<img width="447" height="504" alt="image" src="https://github.com/user-attachments/assets/e4cf6bec-9bd4-4e89-8f67-c76193ca6b01" />




