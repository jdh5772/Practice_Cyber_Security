# Oopsie - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA Oopsie 10.129.95.191
```
<img width="1106" height="378" alt="image" src="https://github.com/user-attachments/assets/3089b3b5-df7d-47f0-8d3d-70a404243a8f" />

- SSH(22)
- HTTP(80)
---
## HTTP
- banner grabbing
```bash
whatweb http://10.129.95.191/

curl -IL http://10.129.95.191/
```
<img width="1106" height="237" alt="image" src="https://github.com/user-attachments/assets/4f398e79-f520-4a20-94e5-c37a06619641" />

<br>
<br>

- `/etc/hosts`에 등록
<img width="1106" height="70" alt="image" src="https://github.com/user-attachments/assets/17d12d6d-0340-4299-ba3e-c1406a1fd5c2" />

<br>
<br>

- `ffuf`로 vhost 및 dns를 찾아보았으나 특별한 정보 없었음.
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://FUZZ.megacorp.com -mc all -fs 114

ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://megacorp.com -H 'Host : FUZZ.megacorp.com' -mc all -fs 10932
```

<br>

- `gobuster`로 enumeration을 하니 uploads가 발견됨.
```bash
gobuster dir -u http://megacorp.com -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,md
```
<img width="1106" height="483" alt="image" src="https://github.com/user-attachments/assets/d89c1e04-3506-4381-92d8-98eba98bba1d" />

<br>
<br>

- 접속을 해보았으나 접근이 불가능한 상태.
<img width="1200" height="656" alt="image" src="https://github.com/user-attachments/assets/58baccf5-2b77-4351-9c9f-1b55b53f833e" />

<br>
<br>

- `burpsuite`를 통해서 `site map`을확인해보니 `/cdn-cgi/login/` 경로가 확인됨.
<img width="1200" height="656" alt="image" src="https://github.com/user-attachments/assets/b14de198-2038-4261-a982-c3f894a09999" />

<br>
<br>

- `/cdn-cig/login`경로에 접속하였고, 로그인할 계정에 대한 정보가 주어지지 않았기에 먼저 `Login as Guest`를 클릭하여 guest로 로그인을 시도.
<img width="322" height="308" alt="image" src="https://github.com/user-attachments/assets/dd02cdf4-6015-4f34-a885-f48f5ae9aa2c" />

<br>
<br>

- `Uploads`탭에서 업로드를 진행할 수 있는 것은 확인되나, guest 유저로는 불가능함을 확인.
<img width="652" height="205" alt="image" src="https://github.com/user-attachments/assets/7d44e846-53ff-4e3c-9e26-7864b521f37e" />

<br>
<br>

- `Account`탭을 확인해보니 `Access ID`와 `Name`이 적혀있었고, 해당 사이트의 쿠키를 확인해보니 동일한 내용이 쿠키로 저장되어 있는 것을 확인.
<img width="1200" height="656" alt="image" src="https://github.com/user-attachments/assets/d701b413-2bfa-4eaf-9e5b-f5b819a3e823" />

---
## Upload and get shell
- `id`파라미터 부분에서 다른 유저에 대해서 확인할 수 있나 싶어 1로 변경해보니 `admin`계정에 대한 정보가 확인됨.
<img width="1544" height="357" alt="image" src="https://github.com/user-attachments/assets/dc4bb7f8-e750-4e51-b6ae-7e1382858976" />

<br>
<br>

- 해당 내용으로 쿠키를 변경하여 다시 `Uploads`탭을 확인하니 업로드가 가능한 상태로 바뀜.
<img width="1200" height="656" alt="image" src="https://github.com/user-attachments/assets/12061525-5e28-4fa1-a83a-ec32153fb15e" />

<br>
<br>

- webshell을 복사.
```bash
cp /usr/share/webshells/php/simple-backdoor.php .
```
<img width="548" height="54" alt="image" src="https://github.com/user-attachments/assets/87d7578f-64e8-49c3-b58b-b40555002374" />

<br>
<br>

- upload를 시도하여 성공.
<img width="587" height="190" alt="image" src="https://github.com/user-attachments/assets/9f9465d9-5566-439c-a68f-e82545908242" />

<br>
<br>

- `gobuster`에서 `uploads`경로가 있다는 것을 확인하여 해당 경로에 업로드가 되었을 거라 생각하여 접속을 시도.
- whoami 명령어 실행 성공.
<img width="769" height="92" alt="image" src="https://github.com/user-attachments/assets/3984b4b1-ed57-4feb-ae9c-2a7b96349bac" />

<br>
<br>

- `brupsuite`에서 request를 가로채서 reverse shell 페이로드로 변경하여 실행.
<img width="1200" height="209" alt="image" src="https://github.com/user-attachments/assets/a252e0c8-ca44-4821-9f15-6103f618c673" />

<br>
<br>

- shell 획득 성공.
<img width="1105" height="184" alt="image" src="https://github.com/user-attachments/assets/a2ca4e81-c856-4f1a-b7e1-498d3b6e2b3d" />

---
## Login robert
- `/var/www/html/cdn-cgi/login`폴더에 `db.php`파일에서 mysql에 접속할 수있는 `robert`의 계정이 발견됨.
<img width="1105" height="231" alt="image" src="https://github.com/user-attachments/assets/ff2c79de-f6d3-415a-90b7-37f702df7814" />

<br>
<br>

- 해당 계정정보를 이용하여 `robert`로 로그인할 수 있는지 시도하여 성공.
```bash
su - robert
```
<img width="1105" height="79" alt="image" src="https://github.com/user-attachments/assets/6a366728-5f8d-492f-b70a-a8afb91bb855" />

---
## Privesc
- `/usr/bin/bugtracker`라는 프로그램에 suid가 설정되어 있는 것을 확인.
```bash
find / -perm -4000 2>/dev/null
```
<img width="1105" height="175" alt="image" src="https://github.com/user-attachments/assets/79ca3c5c-c288-4efb-91d8-fd54321127e1" />

<br>
<br>

- `bugtracker`를 실행시켜서 asdf를 입력해보니 최종적으로 `cat /root/reports/asdf`로 명령이 실행되는 것을 확인함.
<img width="1105" height="205" alt="image" src="https://github.com/user-attachments/assets/5cc2ad4d-f9bc-43f2-890b-c17f09f0992e" />

---
## flag
- `/home/robert/user.txt`
<img width="1105" height="212" alt="image" src="https://github.com/user-attachments/assets/51e9d354-7c47-4792-9153-33a1a4b95314" />

<br>
<br>

- `bugtracker`에서 `../root.txt`를 입력하여 root 플래그 획득
<img width="1105" height="164" alt="image" src="https://github.com/user-attachments/assets/509d2a80-e148-4191-a491-397c6fc6ee56" />
