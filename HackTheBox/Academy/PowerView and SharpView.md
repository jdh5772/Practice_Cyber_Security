## Enumerating
- Sharpview는 파라미터 전달시에 대소문자 구분을 함으로, 정확하게 써 줘야 한다.
- 기존 LDAP 쿼리 결과는 .NET 객체로 반환되어 파이프로 findstr을 사용할 수 없다.
- PowerView / SharpView는 결과를 평탄화된 문자열 속성(PSObject)으로 변환해 주기 때문에 findstr을 파이프로 사용할 수 있다.
```poweshell
.\SharpView.exe ConvertTo-SID -Name sally.jones

.\SharpView.exe Convert-ADName -ObjectName S-1-5-21-2974783224-3764228556-2640795941-1724

Get-DomainUser harry.jones  | ConvertFrom-UACValue -showall

.\SharpView.exe Get-Domain

.\SharpView.exe Get-DomainOU | findstr /b "name"

.\SharpView.exe Get-DomainUser -KerberosPreauthNotRequired

Get-DomainComputer | select dnshostname, useraccountcontrol

.\SharpView.exe Get-DomainGPO | findstr displayname

Test-AdminAccess -ComputerName SQL01

.\SharpView.exe Get-NetShare -ComputerName DC01

Find-DomainUserLocation

Get-DomainTrust
```

## AD User
```powershell
(Get-DomainUser).count

Get-DomainUser -Identity harry.jones -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,mail,useraccountcontrol

Get-DomainUser * -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,mail,useraccountcontrol | Export-Csv .\inlanefreight_users.csv -NoTypeInformation

.\SharpView.exe Get-DomainUser -KerberosPreauthNotRequired -Properties samaccountname,useraccountcontrol,memberof

.\SharpView.exe Get-DomainUser -TrustedToAuth -Properties samaccountname,useraccountcontrol,memberof

.\SharpView.exe Get-DomainUser -LDAPFilter "(userAccountControl:1.2.840.113556.1.4.803:=524288)"

Get-DomainUser -Properties samaccountname,description | Where {$_.description -ne $null}

.\SharpView.exe Get-DomainUser -SPN -Properties samaccountname,memberof,serviceprincipalname

Find-ForeignGroup

Get-DomainUser -Properties samaccountname,pwdlastset,lastlogon -Domain InlaneFreight.local | select samaccountname, pwdlastset, lastlogon | Sort-Object -Property pwdlastset

Get-DomainUser -Properties samaccountname,pwdlastset,lastlogon -Domain InlaneFreight.local | select samaccountname, pwdlastset, lastlogon | where { $_.pwdlastset -lt (Get-Date).addDays(-90) }
```
