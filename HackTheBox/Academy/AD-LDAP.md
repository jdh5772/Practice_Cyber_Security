## LDAP Queries
```powershell
Get-ADObject -LDAPFilter '(objectClass=group)' | select name

Get-ADObject -LDAPFilter '(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))' -Properties * | select samaccountname,useraccountcontrol

(get-aduser -ldapfilter '(objectclass=user)').count

(get-adobject -ldapfilter '(objectclass=group)').count
```

## Filter
```powershell
get-ciminstance win32_product -Filter "NOT Vendor like '%Microsoft%'" | fl

Get-ADComputer  -Filter "DNSHostName -like 'SQL*'"

Get-ADGroup -Filter "adminCount -eq 1" | select Name

Get-ADUser -Filter {adminCount -eq '1' -and DoesNotRequirePreAuth -eq 'True'}

Get-ADUser -Filter "adminCount -eq '1'" -Properties * | where servicePrincipalName -ne $null | select SamAccountName,MemberOf,ServicePrincipalName | fl

get-adcomputer -filter 'name -eq "ws01"'|select sid
```
