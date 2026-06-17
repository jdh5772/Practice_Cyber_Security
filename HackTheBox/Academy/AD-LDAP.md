## LDAP Queries
```powershell
Get-ADObject -LDAPFilter '(objectClass=group)' | select name

Get-ADObject -LDAPFilter '(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))' -Properties * | select samaccountname,useraccountcontrol

(get-aduser -ldapfilter '(objectclass=user)').count

(get-adobject -ldapfilter '(objectclass=group)').count

Get-ADGroup -LDAPFilter '(member:1.2.840.113556.1.4.1941:=CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL)' | select Name

Get-ADUser -Properties * -LDAPFilter '(&(objectCategory=user)(description=*))' | select samaccountname,description

Get-ADUser -Properties * -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' | select Name,memberof, servicePrincipalName,TrustedForDelegation | fl

Get-ADComputer -Properties * -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' | select DistinguishedName,servicePrincipalName,TrustedForDelegation | fl

Get-AdUser -LDAPFilter '(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))(adminCount=1)' -Properties * | select name,memberof | fl

Get-ADGroup -LDAPFilter '(member:1.2.840.113556.1.4.1941:=CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL)' |select Name
```

## Filter
```powershell
get-ciminstance win32_product -Filter "NOT Vendor like '%Microsoft%'" | fl

Get-ADComputer  -Filter "DNSHostName -like 'SQL*'"

Get-ADGroup -Filter "adminCount -eq 1" | select Name

Get-ADUser -Filter {adminCount -eq '1' -and DoesNotRequirePreAuth -eq 'True'}

Get-ADUser -Filter "adminCount -eq '1'" -Properties * | where servicePrincipalName -ne $null | select SamAccountName,MemberOf,ServicePrincipalName | fl

get-adcomputer -filter 'name -eq "ws01"'|select sid

Get-ADGroup -Filter 'member -RecursiveMatch "CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL"' | select name

# OU
Get-ADOrganizationalUnit -Filter *
```

## Searchbase
```powershell
Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope Base -Filter *

Get-ADObject -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope Base -Filter *

Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope OneLevel -Filter *

Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope 1 -Filter *

(Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope Subtree -Filter *).count
```

## UAC
```
Get-ADUser -Filter {adminCount -gt 0} -Properties admincount,useraccountcontrol | select Name,useraccountcontrol

.\Convert-UserAccountControlValues.ps1

Please provide the userAccountControl value: : 4260384
```
```powershell
import-module .\powerview.ps1

Get-DomainUser * -AdminCount | select samaccountname,useraccountcontrol
```
```powershell
dsquery user "OU=Employees,DC=inlanefreight,DC=local" -name * -scope subtree -limit 0 | dsget user -samid -pwdneverexpires | findstr /V no
```
```powershell
Get-ADUser -Filter * -SearchBase 'OU=Admin,DC=inlanefreight,dc=local'

Get-WmiObject -Class win32_group -Filter "Domain='INLANEFREIGHT'" | Select Caption,Name

([adsisearcher]"(&(objectClass=Computer))").FindAll() | select Path
```

## Enumerating
```python3
from ldap3 import *
s = Server('10.129.1.207',get_info = ALL)
c =  Connection(s, '', '')
c.bind()
s.info
```
```bash
ldapsearch -H ldap://10.129.1.207 -x -b

ldapsearch -v -x -b 'dc=inlanefreight,dc=local' -H 'ldap://10.129.42.188' "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=524288) (memberOf:1.2.840.113556.1.4.1941:=CN=Protected Users,CN=Users,DC=inlanefreight,DC=local))"
```
```bash
python3 windapsearch.py --dc-ip 10.129.1.207 -u "" --functionality

python3 windapsearch.py --dc-ip 10.129.1.207 -u "" -U

python3 windapsearch.py --dc-ip 10.129.1.207 -u "" -C

python3 windapsearch.py --dc-ip 10.129.1.207 -u inlanefreight\\james.cross --da

python3 windapsearch.py --dc-ip 10.129.1.207 -d inlanefreight.local -u inlanefreight\\james.cross --unconstrained-users
```
```bash
python3 ldapsearch-ad.py -l 10.129.1.207 -t info

python3 ldapsearch-ad.py -l 10.129.1.207 -d inlanefreight -u james.cross -p Summer2020 -t pass-pols

python3 ldapsearch-ad.py -l 10.129.1.207 -d inlanefreight -u james.cross -p Summer2020 -t kerberoast | grep servicePrincipalName:

python3 ldapsearch-ad.py -l 10.129.1.207 -d inlanefreight -u james.cross -p Summer2020 -t asreproast
```
