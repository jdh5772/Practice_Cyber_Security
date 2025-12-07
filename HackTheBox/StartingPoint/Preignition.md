# Preignition - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 80 -sC -sV -vv -oA preignition 10.129.4.69
```
<img width="494" height="142" alt="image" src="https://github.com/user-attachments/assets/f8f5e081-58bb-49ac-abc6-bac47cd30023" />

- HTTP(80)

##  Enumerate Directory
```bash
gobuster dir -u http://10.129.4.69 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,md
```
- 대부분의 web server가 php를 사용하다보니, 탐색할때 php를 기본적으로 넣어서 탐색해봄.
- admin.php 발견
<img width="1108" height="367" alt="image" src="https://github.com/user-attachments/assets/50925337-bc5f-4957-86d8-2ab89a635440" />

 ## admin.php 로그인
 - 주어진 계정이 존재하지 않으나, 기본적으로 많이 사용하는 admin/admin을 시도.
 - flag 획득
<img width="906" height="417" alt="image" src="https://github.com/user-attachments/assets/a9bdb782-4f9d-423a-982c-dfde0ca0de9c" />
