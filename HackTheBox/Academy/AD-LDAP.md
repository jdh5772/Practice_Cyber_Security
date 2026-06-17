## LDAP Queries
```powershell
Get-ADObject -LDAPFilter '(objectClass=group)' | select name

Get-ADObject -LDAPFilter '(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))' -Properties * | select samaccountname,useraccountcontrol

(get-aduser -ldapfilter '(objectclass=user)').count

(get-adobject -ldapfilter '(objectclass=group)').count
```
